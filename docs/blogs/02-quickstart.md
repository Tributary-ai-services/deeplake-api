# Your First Vector Search in 10 Minutes: DeepLake API Multi-Language Quick Start

*Published: January 2025 | Reading Time: 12 minutes*

## Introduction

Ready to experience the power of semantic search in YOUR programming language? In this hands-on guide, we'll get you from zero to your first vector search in just 10 minutes - whether you're coding in Go, Java, TypeScript, Python, or any other language. By the end, you'll have DeepLake's powerful vector search working seamlessly in your preferred development environment.

## What We'll Build (Language-Agnostic)

We're going to create a simple but powerful document search engine that works with ANY programming language:
- Store documents as semantic vectors via REST/gRPC APIs
- Search by meaning using standard HTTP calls or gRPC clients
- Filter results by metadata using JSON queries
- Handle real-time updates from any language or platform

Think of it as Google search, but for your private documents – accessible from Go microservices, Java enterprise apps, TypeScript frontends, or any other technology in your stack!

## Prerequisites

Before we dive in, choose your adventure:
- **Docker** installed ([Get Docker](https://docs.docker.com/get-docker/))
- **ANY programming language** you prefer (we'll show Go, Java, TypeScript, Python examples)
- **curl** (for testing the API)
- **10 minutes of your time** ⏰

**No Python required!** DeepLake API exposes everything through standard HTTP and gRPC protocols.

## Step 1: Launch DeepLake API (2 minutes)

The fastest way to get started is with Docker Compose. We'll launch the entire stack – DeepLake API, Redis cache, and monitoring – with a single command.

### Option A: Docker Compose (Recommended)

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  deeplake-api:
    image: deeplake-api:latest
    ports:
      - "8000:8000"  # HTTP API
      - "50051:50051" # gRPC API
    environment:
      - DEEPLAKE_STORAGE_LOCATION=/data/vectors
      - REDIS_URL=redis://redis:6379/0
      - JWT_SECRET_KEY=your-super-secret-jwt-key-change-in-production
      - MONITORING_LOG_LEVEL=INFO
    volumes:
      - ./data:/data
    depends_on:
      - redis
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    restart: unless-stopped

  # Optional: Monitoring stack
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9091:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

volumes:
  redis_data:
```

Now launch it:

```bash
# Clone the repository for configuration files
git clone https://github.com/Tributary-ai-services/deeplake-api.git
cd deeplake-api

# Start all services
docker-compose up -d

# Check that services are running
docker-compose ps
```

### Option B: Manual Setup (Alternative)

If you prefer to run locally:

```bash
# Clone and setup
git clone https://github.com/Tributary-ai-services/deeplake-api.git
cd deeplake-api

# Install dependencies
pip install -r requirements.txt

# Set environment variables
export DEEPLAKE_STORAGE_LOCATION=./data/vectors
export REDIS_URL=redis://localhost:6379/0
export JWT_SECRET_KEY=$(python -c "import secrets; print(secrets.token_urlsafe(32))")

# Start Redis (separate terminal)
redis-server

# Start the API
python -m app.main
```

### Verify Installation

Check that everything is working:

```bash
# Health check
curl http://localhost:8000/api/v1/health

# Expected response:
# {
#   "status": "healthy",
#   "timestamp": "2025-01-17T10:30:00Z",
#   "services": {
#     "deeplake": "healthy",
#     "redis": "healthy"
#   }
# }
```

🎉 **Success!** Your DeepLake API is now running. Let's see what we can access:

- **API Documentation**: http://localhost:8000/docs
- **Alternative Docs**: http://localhost:8000/redoc  
- **Health Endpoint**: http://localhost:8000/api/v1/health
- **Metrics** (optional): http://localhost:9091

## Step 2: Set Up Authentication (1 minute)

DeepLake API uses secure authentication. Let's generate an API key:

```bash
# Generate an API key
python scripts/generate_api_key_quick.py

# Output will be something like:
# Generated API Key: dlk_1234567890abcdef...
# 
# Add this to your environment:
# export API_KEY="dlk_1234567890abcdef..."
```

Set the API key for easy use:

```bash
export API_KEY="dlk_1234567890abcdef..."  # Replace with your actual key
```

Test authentication:

```bash
curl -H "Authorization: ApiKey $API_KEY" \
     http://localhost:8000/api/v1/health

# Should return the same health response as before
```

## Step 3: Create Your First Dataset (1 minute)

A dataset is like a database table, but for vectors. Let's create one for storing documents:

```bash
curl -X POST "http://localhost:8000/api/v1/datasets" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-documents",
    "dimensions": 1536,
    "metric_type": "cosine",
    "description": "My first document search engine"
  }'
```

**Expected response:**
```json
{
  "success": true,
  "dataset": {
    "id": "my-documents",
    "name": "my-documents", 
    "dimensions": 1536,
    "metric_type": "cosine",
    "description": "My first document search engine",
    "created_at": "2025-01-17T10:30:00Z",
    "vector_count": 0
  }
}
```

**What just happened?**
- We created a dataset called "my-documents"
- It stores 1536-dimensional vectors (perfect for OpenAI embeddings)
- Uses cosine similarity for search (best for semantic similarity)

## Step 4: Add Your First Documents (2 minutes)

Now let's add some sample documents. In a real application, you'd convert text to vectors using an embedding model like OpenAI's text-embedding-ada-002. For this demo, we'll use mock vectors:

```bash
curl -X POST "http://localhost:8000/api/v1/datasets/my-documents/vectors/batch" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": [
      {
        "id": "doc1",
        "document_id": "machine-learning-guide", 
        "values": [0.1, 0.2, 0.3, 0.4, 0.5],
        "content": "Complete guide to machine learning algorithms and neural networks",
        "metadata": {
          "title": "Machine Learning Fundamentals",
          "category": "technology",
          "author": "Dr. Sarah Chen",
          "publish_date": "2024-01-15",
          "tags": ["ml", "ai", "algorithms"]
        }
      },
      {
        "id": "doc2", 
        "document_id": "python-tutorial",
        "values": [0.2, 0.3, 0.4, 0.5, 0.6],
        "content": "Learn Python programming from basics to advanced concepts",
        "metadata": {
          "title": "Python Programming Masterclass",
          "category": "technology", 
          "author": "John Rodriguez",
          "publish_date": "2024-02-01",
          "tags": ["python", "programming", "tutorial"]
        }
      },
      {
        "id": "doc3",
        "document_id": "cooking-recipes",
        "values": [0.8, 0.7, 0.6, 0.5, 0.4], 
        "content": "Delicious and healthy cooking recipes for every occasion",
        "metadata": {
          "title": "Healthy Cooking Made Easy",
          "category": "lifestyle",
          "author": "Chef Maria González", 
          "publish_date": "2024-01-20",
          "tags": ["cooking", "health", "recipes"]
        }
      }
    ]
  }'
```

**Expected response:**
```json
{
  "success": true,
  "inserted_count": 3,
  "inserted_ids": ["doc1", "doc2", "doc3"],
  "processing_time": "0.045s"
}
```

**Note:** In production, you'd generate real embeddings:
```python
import openai

# Convert text to vector
response = openai.Embedding.create(
    model="text-embedding-ada-002",
    input="Your document text here"
)
vector = response['data'][0]['embedding']  # This is your 1536-dimensional vector
```

## Step 5: Your First Vector Search (2 minutes)

Now for the magic moment – let's search for documents similar to a query about "artificial intelligence":

```bash
curl -X POST "http://localhost:8000/api/v1/datasets/my-documents/search" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query_vector": [0.15, 0.25, 0.35, 0.45, 0.55],
    "top_k": 5,
    "include_metadata": true,
    "include_content": true
  }'
```

**Expected response:**
```json
{
  "success": true,
  "results": [
    {
      "id": "doc1",
      "document_id": "machine-learning-guide",
      "score": 0.9876,
      "content": "Complete guide to machine learning algorithms and neural networks",
      "metadata": {
        "title": "Machine Learning Fundamentals",
        "category": "technology",
        "author": "Dr. Sarah Chen",
        "publish_date": "2024-01-15",
        "tags": ["ml", "ai", "algorithms"]
      }
    },
    {
      "id": "doc2", 
      "document_id": "python-tutorial",
      "score": 0.8932,
      "content": "Learn Python programming from basics to advanced concepts",
      "metadata": {
        "title": "Python Programming Masterclass",
        "category": "technology",
        "author": "John Rodriguez"
      }
    },
    {
      "id": "doc3",
      "score": 0.3421,
      "content": "Delicious and healthy cooking recipes for every occasion",
      "metadata": {
        "title": "Healthy Cooking Made Easy",
        "category": "lifestyle"
      }
    }
  ],
  "total_results": 3,
  "processing_time": "0.012s"
}
```

🎉 **It works!** Notice how:
- The machine learning document scored highest (0.9876) – most similar to our AI query
- Python tutorial came second (0.8932) – related to technology 
- Cooking recipes scored lowest (0.3421) – least similar

## Step 6: Advanced Search with Filters (2 minutes)

Let's make our search more intelligent by adding metadata filters:

```bash
# Search only in "technology" category
curl -X POST "http://localhost:8000/api/v1/datasets/my-documents/search" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query_vector": [0.15, 0.25, 0.35, 0.45, 0.55],
    "top_k": 5,
    "filter": {
      "category": "technology"
    },
    "include_metadata": true
  }'
```

```bash
# Search by date range and tags
curl -X POST "http://localhost:8000/api/v1/datasets/my-documents/search" \
  -H "Authorization: ApiKey $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query_vector": [0.15, 0.25, 0.35, 0.45, 0.55],
    "top_k": 5,
    "filter": {
      "publish_date": {"$gte": "2024-01-01"},
      "tags": {"$in": ["ml", "ai"]}
    },
    "include_metadata": true
  }'
```

**Powerful filtering options:**
- `{"category": "technology"}` – Exact match
- `{"publish_date": {"$gte": "2024-01-01"}}` – Date range
- `{"tags": {"$in": ["ml", "ai"]}}` – Array contains any value
- `{"score": {"$gt": 0.8, "$lt": 1.0}}` – Numeric range

## Multi-Language Integration Examples

Now that you have DeepLake API running, let's see how to integrate it into applications using different programming languages:

### 🐹 Go Example

Perfect for microservices and high-performance backends:

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

type SearchRequest struct {
    QueryVector []float64 `json:"query_vector"`
    TopK        int       `json:"top_k"`
    Filter      map[string]interface{} `json:"filter"`
}

type SearchResult struct {
    ID       string                 `json:"id"`
    Score    float64               `json:"score"`
    Metadata map[string]interface{} `json:"metadata"`
    Content  string                `json:"content"`
}

type SearchResponse struct {
    Success bool           `json:"success"`
    Results []SearchResult `json:"results"`
}

func searchDocuments(apiKey string, query []float64) (*SearchResponse, error) {
    url := "http://localhost:8000/api/v1/datasets/my-documents/search"
    
    request := SearchRequest{
        QueryVector: query,
        TopK:        5,
        Filter:      map[string]interface{}{"category": "technology"},
    }
    
    jsonData, _ := json.Marshal(request)
    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
    req.Header.Set("Authorization", "ApiKey "+apiKey)
    req.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var response SearchResponse
    json.NewDecoder(resp.Body).Decode(&response)
    
    return &response, nil
}

func main() {
    // Example usage
    query := []float64{0.15, 0.25, 0.35, 0.45, 0.55}
    results, err := searchDocuments("your-api-key", query)
    
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Printf("Found %d results:\n", len(results.Results))
    for _, result := range results.Results {
        fmt.Printf("📄 %s (Score: %.3f)\n", 
            result.Metadata["title"], result.Score)
    }
}
```

### ☕ Java/Spring Boot Example

Enterprise-ready integration:

```java
@RestController
@RequestMapping("/api/search")
public class DocumentSearchController {
    
    @Value("${deeplake.api.url}")
    private String deeplakeApiUrl;
    
    @Value("${deeplake.api.key}")
    private String apiKey;
    
    private final RestTemplate restTemplate;
    
    public DocumentSearchController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    @PostMapping("/documents")
    public ResponseEntity<SearchResponse> searchDocuments(
            @RequestBody SearchQuery query) {
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "ApiKey " + apiKey);
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        SearchRequest request = SearchRequest.builder()
            .queryVector(query.getEmbedding())
            .topK(5)
            .filter(Map.of("category", "technology"))
            .build();
        
        HttpEntity<SearchRequest> entity = new HttpEntity<>(request, headers);
        
        SearchResponse response = restTemplate.postForObject(
            deeplakeApiUrl + "/api/v1/datasets/my-documents/search",
            entity,
            SearchResponse.class
        );
        
        return ResponseEntity.ok(response);
    }
}

// Usage in service class
@Service
public class DocumentService {
    
    @Autowired
    private DocumentSearchController searchController;
    
    public List<Document> findSimilarDocuments(String userQuery) {
        // Convert query to embedding (using your preferred embedding service)
        double[] embedding = embeddingService.getEmbedding(userQuery);
        
        SearchQuery query = new SearchQuery(embedding);
        SearchResponse response = searchController.searchDocuments(query).getBody();
        
        return response.getResults().stream()
            .map(this::convertToDocument)
            .collect(Collectors.toList());
    }
}
```

### 🟦 TypeScript/Node.js Example

Perfect for modern web applications:

```typescript
interface SearchRequest {
    query_vector: number[];
    top_k: number;
    filter?: Record<string, any>;
    include_metadata?: boolean;
}

interface SearchResult {
    id: string;
    score: number;
    metadata: Record<string, any>;
    content: string;
}

interface SearchResponse {
    success: boolean;
    results: SearchResult[];
    total_count: number;
}

class DeepLakeClient {
    private baseUrl: string;
    private apiKey: string;
    
    constructor(baseUrl: string, apiKey: string) {
        this.baseUrl = baseUrl;
        this.apiKey = apiKey;
    }
    
    async searchDocuments(
        datasetId: string, 
        queryVector: number[], 
        options: Partial<SearchRequest> = {}
    ): Promise<SearchResponse> {
        const url = `${this.baseUrl}/api/v1/datasets/${datasetId}/search`;
        
        const request: SearchRequest = {
            query_vector: queryVector,
            top_k: options.top_k || 5,
            filter: options.filter,
            include_metadata: true,
            ...options
        };
        
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'Authorization': `ApiKey ${this.apiKey}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(request)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json() as SearchResponse;
    }
}

// Usage example
async function main() {
    const client = new DeepLakeClient(
        'http://localhost:8000',
        'your-api-key'
    );
    
    // Example query vector (in production, get this from an embedding service)
    const queryVector = [0.15, 0.25, 0.35, 0.45, 0.55];
    
    try {
        const results = await client.searchDocuments(
            'my-documents',
            queryVector,
            { 
                filter: { category: 'technology' },
                top_k: 10 
            }
        );
        
        console.log(`Found ${results.results.length} results:`);
        results.results.forEach(result => {
            console.log(`📄 ${result.metadata.title} (Score: ${result.score.toFixed(3)})`);
        });
        
    } catch (error) {
        console.error('Search error:', error);
    }
}

main();
```

### 🐍 Python Example (For Comparison)

Even in Python, you can use the HTTP API instead of a specialized SDK:

```python
import asyncio
import aiohttp
import json

class DeepLakeAPIClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.api_key = api_key
    
    async def search_documents(self, dataset_id: str, query_vector: list, **kwargs):
        url = f"{self.base_url}/api/v1/datasets/{dataset_id}/search"
        
        payload = {
            "query_vector": query_vector,
            "top_k": kwargs.get("top_k", 5),
            "filter": kwargs.get("filter", {}),
            "include_metadata": True
        }
        
        headers = {
            "Authorization": f"ApiKey {self.api_key}",
            "Content-Type": "application/json"
        }
        
        async with aiohttp.ClientSession() as session:
            async with session.post(url, json=payload, headers=headers) as response:
                return await response.json()

async def main():
    client = DeepLakeAPIClient("http://localhost:8000", "your-api-key")
    
    # Example search
    query_vector = [0.15, 0.25, 0.35, 0.45, 0.55]
    results = await client.search_documents(
        "my-documents",
        query_vector,
        filter={"category": "technology"}
    )
    
    print(f"Found {len(results['results'])} results:")
    for result in results['results']:
        print(f"📄 {result['metadata']['title']} (Score: {result['score']:.3f})")

asyncio.run(main())
```

## What You've Accomplished

In just 10 minutes, you've:

✅ **Launched a production-ready vector database gateway**  
✅ **Created your first dataset** with proper configuration  
✅ **Added documents** with rich metadata  
✅ **Performed semantic searches** that understand meaning  
✅ **Applied intelligent filters** for precise results  
✅ **Integrated with multiple programming languages** - Go, Java, TypeScript, Python
✅ **Learned language-agnostic patterns** that work across your entire tech stack  

## Next Steps: Take It Further

Now that you have the basics working, here are some exciting directions to explore:

### 🚀 **Immediate Next Steps**
1. **[Vector Operations Deep Dive](./03-vector-operations.md)** – Learn advanced vector management
2. **[Advanced Search Capabilities](./04-advanced-search.md)** – Hybrid search, multi-modal queries
3. **Try with real embeddings** – Use OpenAI, Cohere, or Hugging Face models

### 🎯 **Real-World Applications**
- **Document Search**: Index your PDFs, docs, and knowledge base
- **Product Recommendations**: Find similar products based on descriptions
- **Content Discovery**: Help users find relevant articles, videos, or resources
- **Question Answering**: Build a RAG system for intelligent chatbots

### 🏗️ **Production Considerations**
- **[Production Deployment](./05-production.md)** – Scale to handle real workloads
- **[Monitoring & Observability](../monitoring.md)** – Keep your system healthy
- **[Security Best Practices](../SECURITY.md)** – Protect your data

## Common Troubleshooting

**API not responding?**
```bash
# Check if services are running
docker-compose ps

# Check logs
docker-compose logs deeplake-api
```

**Authentication errors?**
```bash
# Regenerate API key
python scripts/generate_api_key_quick.py

# Test with new key
export API_KEY="your-new-key"
```

**Vector dimension mismatch?**
```bash
# Check your dataset dimensions
curl -H "Authorization: ApiKey $API_KEY" \
     http://localhost:8000/api/v1/datasets/my-documents
```

## Interactive API Documentation

Don't forget to explore the interactive API docs at http://localhost:8000/docs – you can test all endpoints directly from your browser!

## Community and Support

Join our growing community:

- 💬 **[Discord Community](https://discord.gg/deeplake)** – Get help and share ideas
- 📚 **[Full Documentation](../README.md)** – Comprehensive guides
- 🐛 **[GitHub Issues](https://github.com/Tributary-ai-services/deeplake-api/issues)** – Report bugs or request features
- 📧 **[Technical Support](mailto:tas-deeplake@scharber.com)** – Professional support

## Conclusion

Congratulations! 🎉 You've just built your first semantic search engine. What you've learned here forms the foundation for countless AI applications – from chatbots that understand context to recommendation engines that know what users really want.

The magic of vector databases isn't just in the technology – it's in how they enable applications that truly understand and connect information in meaningful ways. You're now equipped to build the next generation of intelligent applications.

**Ready to dive deeper?** Check out our [Vector Operations Deep Dive](./03-vector-operations.md) to master the advanced features that will take your application to the next level.

---

*This blog post is part of the DeepLake API Developer Blog Series. [View all posts →](./README.md)*

### Additional Resources

- 📖 **[API Reference](../api/http/README.md)** – Complete endpoint documentation
- 🔧 **[Configuration Guide](../configuration.md)** – Advanced setup options  
- 🐳 **[Docker Guide](../installation.md)** – Container deployment
- ⚡ **[Performance Tips](../observability.md)** – Optimize for speed and scale