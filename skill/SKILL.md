---
name: karakeep-semantic-search
description: "Semantic bookmark search for Karakeep using vector embeddings. Searches bookmarks by meaning via REST API with configurable OpenAI or Ollama embeddings and Qdrant vector storage. Use when querying Karakeep bookmarks semantically, searching saved links by concept, syncing bookmark embeddings, or checking search service health."
---

# Karakeep Semantic Search

Search Karakeep bookmarks by meaning, not just keywords, using vector embeddings and a REST API.

## Prerequisites

Set the service URL:
```
KARAKEEP_SEMANTIC_URL=http://192.168.1.105:3001
```

## Workflow

### 1. Search bookmarks by meaning

```bash
curl -s "${KARAKEEP_SEMANTIC_URL}/search?q=<query>&limit=<n>" | jq
```

- `q` — natural language query (e.g. `how+to+build+a+startup`)
- `limit` — max results (default: 10)

Response:
```json
{
  "results": [
    {
      "bookmarkId": "abc-123",
      "score": 0.85,
      "title": "Article Title",
      "url": "https://example.com/article",
      "tags": ["tag1", "tag2"]
    }
  ],
  "query": "your search",
  "limit": 5,
  "took_ms": 150
}
```

Higher `score` (closer to 1.0) means a better semantic match.

### 2. Sync bookmarks

```bash
# Full sync — re-index all bookmarks
curl -s -X POST "${KARAKEEP_SEMANTIC_URL}/sync" | jq

# Incremental sync — index only new bookmarks since last sync
curl -s -X POST "${KARAKEEP_SEMANTIC_URL}/sync/incremental" | jq

# Sync a single bookmark by ID
curl -s -X POST "${KARAKEEP_SEMANTIC_URL}/sync/bookmark/<id>" | jq
```

### 3. Manage the index

```bash
# Delete a bookmark from the vector index
curl -s -X DELETE "${KARAKEEP_SEMANTIC_URL}/bookmark/<id>" | jq

# Clear all vectors (destructive — removes entire index)
# Check stats first to confirm vector count, then clear, then verify
curl -s "${KARAKEEP_SEMANTIC_URL}/stats" | jq
curl -s -X POST "${KARAKEEP_SEMANTIC_URL}/clear" | jq
curl -s "${KARAKEEP_SEMANTIC_URL}/stats" | jq  # vectorCount should be 0
```

### 4. Monitor the service

```bash
# Health check — returns status and vector count
curl -s "${KARAKEEP_SEMANTIC_URL}/health" | jq

# Stats — vector count and sync interval
curl -s "${KARAKEEP_SEMANTIC_URL}/stats" | jq
```

## Examples

```bash
# Find articles about building startups
curl -s "${KARAKEEP_SEMANTIC_URL}/search?q=how+to+build+a+startup&limit=5" | jq

# Find productivity content
curl -s "${KARAKEEP_SEMANTIC_URL}/search?q=getting+things+done&limit=5" | jq

# Find something vaguely remembered
curl -s "${KARAKEEP_SEMANTIC_URL}/search?q=that+article+about+AI+agents&limit=3" | jq
```

## Notes

- Background sync runs automatically at the configured interval
