# Monitoring Verification Guide

This guide provides step-by-step instructions to verify that Prometheus and Grafana are correctly configured and collecting metrics from the Tributary AI Service for DeepLake.

## 🔍 Prerequisites

Before verifying the monitoring stack:

1. **Ensure all services are running**:
```bash
# Docker Compose
docker-compose ps

# Kubernetes
kubectl get pods -n deeplake-prod

# Expected services:
# - deeplake-api (port 8000)
# - prometheus (port 9091)
# - grafana (port 3000)
# - alertmanager (port 9093)
```

2. **Check service health**:
```bash
# API health
curl http://localhost:8000/api/v1/health

# Prometheus health
curl http://localhost:9091/-/healthy

# Grafana health
curl http://localhost:3000/api/health
```

## 📊 Prometheus Verification

### 1. Access Prometheus UI

Open your browser and navigate to:
```
http://localhost:9091
```

### 2. Verify Targets

Navigate to **Status → Targets** to verify all endpoints are being scraped:

✅ **Expected targets**:
- `deeplake-api` (UP) - http://deeplake-api:9091/metrics
- `redis` (UP) - http://redis:6379/metrics
- `node-exporter` (UP) - http://node-exporter:9100/metrics

Each target should show:
- **State**: UP (green)
- **Last Scrape**: Recent timestamp
- **Scrape Duration**: < 1s

### 3. Verify Metrics Collection

In the Prometheus query interface, run these queries:

#### Basic HTTP Metrics
```promql
# Check if HTTP requests are being recorded
http_requests_total

# Verify request rate (should show data if API is being used)
rate(http_requests_total[5m])

# Check response time histogram
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

#### Business Metrics
```promql
# Check dataset metrics
datasets_active_total

# Verify vector operations
vector_operations_total

# Check search operations
search_operations_total
```

#### System Metrics
```promql
# Memory usage
process_resident_memory_bytes

# CPU usage
rate(process_cpu_seconds_total[5m])

# Active connections
active_connections
```

### 4. Generate Test Traffic

To ensure metrics are being collected, generate some test traffic:

```bash
# Create a test script
cat > test_metrics.sh << 'EOF'
#!/bin/bash

API_KEY="${API_KEY:-your-test-api-key}"
BASE_URL="http://localhost:8000"

echo "Generating test traffic for metrics verification..."

# Create a dataset
echo "Creating test dataset..."
curl -X POST "$BASE_URL/api/v1/datasets" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "dataset_id": "metrics-test",
    "name": "Metrics Test Dataset",
    "embedding_dimension": 384,
    "distance_metric": "cosine"
  }'

# Add some vectors
echo -e "\n\nAdding test vectors..."
for i in {1..10}; do
  curl -X POST "$BASE_URL/api/v1/datasets/metrics-test/vectors" \
    -H "Authorization: ApiKey $API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"vectors\": [[$(seq -s, 1 384 | awk '{for(i=1;i<=384;i++)printf \"%.6f,\", rand()}' | sed 's/,$//')]],
      \"metadata\": [{\"id\": $i, \"type\": \"test\"}]
    }"
done

# Perform searches
echo -e "\n\nPerforming test searches..."
for i in {1..5}; do
  curl -X POST "$BASE_URL/api/v1/datasets/metrics-test/search" \
    -H "Authorization: ApiKey $API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"query_vector\": [$(seq -s, 1 384 | awk '{for(i=1;i<=384;i++)printf \"%.6f,\", rand()}' | sed 's/,$//'])],
      \"k\": 5
    }"
done

echo -e "\n\nTest traffic generation complete!"
EOF

chmod +x test_metrics.sh
./test_metrics.sh
```

### 5. Verify Metrics After Test

After running the test script, verify metrics are updated:

```promql
# Should see increased request count
increase(http_requests_total[5m])

# Should see vector operations
increase(vector_operations_total[5m])

# Should see search operations
increase(search_operations_total[5m])
```

## 📈 Grafana Verification

### 1. Access Grafana

Open your browser and navigate to:
```
http://localhost:3000
```

Default credentials (if not configured):
- Username: `admin`
- Password: `admin` (or check GRAFANA_PASSWORD env var)

### 2. Verify Data Source

Navigate to **Configuration → Data Sources**:

✅ **Expected data source**:
- Name: `Prometheus`
- Type: `Prometheus`
- URL: `http://prometheus:9091`
- Status: ✅ (green checkmark after clicking "Test")

### 3. Import Dashboards

If dashboards aren't already imported, import them:

1. Click **+ → Import**
2. Upload the dashboard JSON files from monitoring configuration
3. Select "Prometheus" as the data source

### 4. Verify Dashboard Data

Navigate to **Dashboards → Browse** and open each dashboard:

#### Tributary AI Service Overview Dashboard

Should display:
- ✅ **Request Rate**: Real-time requests/sec
- ✅ **Error Rate**: Percentage of failed requests
- ✅ **Response Time**: 95th percentile latency
- ✅ **Active Connections**: Current connection count

#### Performance Dashboard

Should display:
- ✅ **CPU Usage**: Graph showing CPU utilization
- ✅ **Memory Usage**: Memory consumption over time
- ✅ **Cache Hit Rate**: Redis cache effectiveness
- ✅ **Database Connections**: Active connection pool

#### Business Metrics Dashboard

Should display:
- ✅ **Total Datasets**: Count of active datasets
- ✅ **Total Vectors**: Number of stored vectors
- ✅ **Search Operations**: Search rate by type
- ✅ **Vector Operations**: Insert/update/delete rates

### 5. Create Test Alert

To verify alerting is working:

1. Navigate to a panel showing request rate
2. Click panel title → Edit
3. Go to Alert tab
4. Create alert rule:
   ```
   Name: Test High Request Rate
   Condition: WHEN avg() OF query(A, 5m, now) IS ABOVE 0.1
   ```
5. Save and wait 5 minutes
6. Generate traffic to trigger alert
7. Check alert state in panel (should turn red)

## 🚨 AlertManager Verification

### 1. Access AlertManager

Open your browser and navigate to:
```
http://localhost:9093
```

### 2. Check Alert Configuration

Navigate to **Status** to see:
- ✅ Configuration file loaded
- ✅ Route configuration
- ✅ Receiver configuration

### 3. Test Alert Flow

Create a test alert to verify the full pipeline:

```bash
# Create a test alert
curl -X POST http://localhost:9093/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '[{
    "labels": {
      "alertname": "TestAlert",
      "severity": "warning",
      "service": "deeplake-api"
    },
    "annotations": {
      "summary": "Test alert for monitoring verification",
      "description": "This is a test alert to verify AlertManager is working"
    }
  }]'
```

Check:
- Alert appears in AlertManager UI
- Notification sent to configured receivers (email/Slack)

## 🔄 End-to-End Verification

### Complete Monitoring Stack Test

Run this comprehensive test to verify the entire monitoring stack:

```bash
cat > verify_monitoring.py << 'EOF'
#!/usr/bin/env python3
"""
Comprehensive monitoring verification script
"""

import requests
import time
import json
from datetime import datetime

def check_service(name, url, expected_status=200):
    """Check if a service is responding"""
    try:
        response = requests.get(url, timeout=5)
        success = response.status_code == expected_status
        return {
            "service": name,
            "url": url,
            "status": "UP" if success else "DOWN",
            "status_code": response.status_code,
            "response_time": response.elapsed.total_seconds()
        }
    except Exception as e:
        return {
            "service": name,
            "url": url,
            "status": "DOWN",
            "error": str(e)
        }

def check_prometheus_metrics(base_url="http://localhost:9091"):
    """Verify Prometheus is collecting metrics"""
    metrics_to_check = [
        "http_requests_total",
        "http_request_duration_seconds_bucket",
        "datasets_active_total",
        "vector_operations_total",
        "search_operations_total"
    ]
    
    results = []
    for metric in metrics_to_check:
        try:
            response = requests.get(f"{base_url}/api/v1/query", 
                                  params={"query": metric})
            data = response.json()
            has_data = len(data.get("data", {}).get("result", [])) > 0
            results.append({
                "metric": metric,
                "status": "FOUND" if has_data else "MISSING",
                "samples": len(data.get("data", {}).get("result", []))
            })
        except Exception as e:
            results.append({
                "metric": metric,
                "status": "ERROR",
                "error": str(e)
            })
    
    return results

def check_grafana_dashboards(base_url="http://localhost:3000", 
                           username="admin", password="admin"):
    """Check Grafana dashboards"""
    try:
        # Get dashboards
        response = requests.get(f"{base_url}/api/search?type=dash-db",
                              auth=(username, password))
        dashboards = response.json()
        
        return {
            "status": "OK",
            "dashboard_count": len(dashboards),
            "dashboards": [d["title"] for d in dashboards]
        }
    except Exception as e:
        return {
            "status": "ERROR",
            "error": str(e)
        }

def main():
    """Run all monitoring checks"""
    print("🔍 Tributary AI Service Monitoring Verification")
    print("=" * 50)
    
    # Check services
    print("\n📊 Service Health Checks:")
    services = [
        ("API", "http://localhost:8000/api/v1/health"),
        ("Prometheus", "http://localhost:9091/-/healthy"),
        ("Grafana", "http://localhost:3000/api/health"),
        ("AlertManager", "http://localhost:9093/-/healthy")
    ]
    
    for name, url in services:
        result = check_service(name, url)
        status_icon = "✅" if result["status"] == "UP" else "❌"
        print(f"{status_icon} {name}: {result['status']}")
        if "response_time" in result:
            print(f"   Response time: {result['response_time']:.3f}s")
    
    # Check Prometheus metrics
    print("\n📈 Prometheus Metrics:")
    metrics = check_prometheus_metrics()
    for metric in metrics:
        status_icon = "✅" if metric["status"] == "FOUND" else "❌"
        print(f"{status_icon} {metric['metric']}: {metric['status']}")
        if metric["status"] == "FOUND":
            print(f"   Samples: {metric['samples']}")
    
    # Check Grafana
    print("\n📊 Grafana Dashboards:")
    grafana = check_grafana_dashboards()
    if grafana["status"] == "OK":
        print(f"✅ Found {grafana['dashboard_count']} dashboards:")
        for dashboard in grafana["dashboards"]:
            print(f"   - {dashboard}")
    else:
        print(f"❌ Error: {grafana.get('error', 'Unknown error')}")
    
    # Summary
    print("\n📋 Summary:")
    all_services_up = all(check_service(name, url)["status"] == "UP" 
                         for name, url in services)
    metrics_found = sum(1 for m in metrics if m["status"] == "FOUND")
    
    if all_services_up and metrics_found > 0:
        print("✅ Monitoring stack is fully operational!")
    else:
        print("❌ Some issues detected. Please check the details above.")
    
    print(f"\nTimestamp: {datetime.now().isoformat()}")

if __name__ == "__main__":
    main()
EOF

chmod +x verify_monitoring.py
python3 verify_monitoring.py
```

## 🛠️ Troubleshooting Common Issues

### Prometheus Not Scraping Metrics

1. **Check target configuration**:
   ```bash
   # View Prometheus config
   curl http://localhost:9091/api/v1/status/config
   ```

2. **Verify metrics endpoint**:
   ```bash
   # Check if API is exposing metrics
   curl http://localhost:9091/metrics
   ```

3. **Check network connectivity**:
   ```bash
   # From Prometheus container
   docker exec prometheus curl http://deeplake-api:9091/metrics
   ```

### Grafana Shows "No Data"

1. **Verify time range**: Ensure you're looking at a time range with data
2. **Check data source**: Test connection in Data Sources settings
3. **Inspect query**: Use Explore view to debug queries
4. **Check permissions**: Ensure Grafana can reach Prometheus

### Missing Metrics

1. **Verify metric names**: Use Prometheus UI to explore available metrics
2. **Check instrumentation**: Ensure application code is registering metrics
3. **Wait for scrape interval**: Metrics appear after scrape_interval (default 15s)

## 📚 Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Tutorial](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Dashboard Best Practices](https://grafana.com/docs/grafana/latest/best-practices/)