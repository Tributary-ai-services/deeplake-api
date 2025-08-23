# 🚀 Quick Access Guide

## ✅ Current Status
- HTTP Server is running on port 8080
- Monitoring dashboard is available
- All monitoring services are up and running

## 🌐 Access URLs

### **From within WSL/Linux:**
```bash
# Monitoring Dashboard
http://localhost:8080/monitoring-dashboard.html

# Services
http://localhost:8000/docs    # API Documentation
http://localhost:3000         # Grafana (admin/admin)  
http://localhost:9091         # Prometheus
http://localhost:9093         # AlertManager
```

### **From Windows (requires port forwarding):**
Your WSL IP: `172.27.163.114`

```bash
# Monitoring Dashboard
http://172.27.163.114:8080/monitoring-dashboard.html

# Services  
http://172.27.163.114:8000/docs    # API Documentation
http://172.27.163.114:3000         # Grafana (admin/admin)
http://172.27.163.114:9091         # Prometheus  
http://172.27.163.114:9093         # AlertManager
```

## 🔧 Troubleshooting "Site Can't Be Reached"

### **Option 1: Use WSL IP directly (easiest)**
Just replace `localhost` with `172.27.163.114` in your browser:
- ❌ `http://localhost:8080/monitoring-dashboard.html`
- ✅ `http://172.27.163.114:8080/monitoring-dashboard.html`

### **Option 2: Set up Windows port forwarding**
Run in Windows PowerShell (as Administrator):
```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=172.27.163.114
```

Then you can use: `http://localhost:8080/monitoring-dashboard.html`

### **Option 3: Test from WSL first**
```bash
# From WSL terminal
curl http://localhost:8080/monitoring-dashboard.html
# Should return HTML content

# Or open in WSL browser (if available)
xdg-open http://localhost:8080/monitoring-dashboard.html
```

## 🔍 Quick Checks

### **Verify services are running:**
```bash
docker-compose ps
```

### **Check HTTP server:**
```bash
netstat -tlnp | grep :8080
# Should show: tcp 0.0.0.0:8080 LISTEN
```

### **Test individual services:**
```bash
curl http://172.27.163.114:8000/api/v1/health  # API
curl http://172.27.163.114:9091/-/healthy      # Prometheus  
curl http://172.27.163.114:3000/api/health     # Grafana
curl http://172.27.163.114:9093/-/healthy      # AlertManager
```

## 📱 Current Server Status
- ✅ HTTP Server running on 0.0.0.0:8080
- ✅ Monitoring dashboard available
- ✅ All monitoring services operational

## 🎯 Recommended Next Step
**Try this URL in your Windows browser:**
```
http://172.27.163.114:8080/monitoring-dashboard.html
```

This should work immediately without any port forwarding setup.