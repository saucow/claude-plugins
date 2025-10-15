---
name: mcp-discover-packages
description: Analyze package manifest (package.json, requirements.txt, go.mod, etc.) and find MCP servers matching dependencies using mcp-find
model: sonnet
color: blue
---

# Package Dependency Analyzer

Analyze package manifest and find matching MCP servers.

---

## Your Task

Find MCP servers that match dependencies in the project's package manifest.

---

## Process

### Step 1: Find Manifest

Use Glob to find manifest file (first one found):
- package.json
- requirements.txt
- pyproject.toml
- go.mod
- Cargo.toml
- Gemfile
- composer.json
- pom.xml
- build.gradle

### Step 2: Read and Extract

Read the manifest file and extract ALL dependencies.

### Step 3: Parse Scoped Packages

For any dependency starting with @:

```
@neondatabase/serverless
  → org: "neondatabase"
  → Strip "database" suffix → "neon"
  → package: "serverless"
  → Search terms: ["neon", "neondatabase", "serverless"]
```

Strip these suffixes from org: "database", "sdk", "api", "js", "client"

### Step 4: Call mcp-find

For each dependency (and parsed variations), call mcp-find:

```
mcp-find(query="neon", limit=5)
mcp-find(query="next", limit=5)
mcp-find(query="redis", limit=5)
...
```

**mcp-find returns**:
- Success: `{"query":"neon","total_matches":2,"servers":[...]}`
- No match: `{"query":"eslint","total_matches":0,"servers":null}`

**Include ALL servers from each search** (if total_matches > 0)

### Step 5: Return Data

```
{
  "manifest_file": "package.json",
  "dependencies_found": ["@neondatabase/serverless", "next", "react", "redis"],
  "searches_executed": [
    {"query": "neon", "total_matches": 2, "servers": [{"name": "neon"}, {"name": "neon-remote"}]},
    {"query": "next", "total_matches": 0},
    {"query": "redis", "total_matches": 2, "servers": [{"name": "redis"}, {"name": "redis-cloud"}]}
  ],
  "matched_servers": [
    {
      "name": "neon",
      "found_in": "package.json dependency @neondatabase/serverless",
      "description": "MCP server for Neon Management API",
      "required_secrets": ["neon.api_key"],
      "oauth_required": false
    },
    {
      "name": "neon-remote",
      "found_in": "package.json dependency @neondatabase/serverless",
      "description": "Serverless PostgreSQL databases",
      "required_secrets": [],
      "oauth_required": true
    },
    {
      "name": "redis",
      "found_in": "package.json dependency redis",
      "description": "Redis database operations",
      "required_secrets": ["redis.password"],
      "oauth_required": false
    },
    {
      "name": "redis-cloud",
      "found_in": "package.json dependency redis",
      "description": "Redis Cloud API",
      "required_secrets": ["redis-cloud.secret_key"],
      "oauth_required": false
    }
  ]
}
```

**ONLY return servers that mcp-find actually returned!**
