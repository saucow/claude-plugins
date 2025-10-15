---
name: mcp-discover-defaults
description: Determine default/always-suggest MCP servers (github-official, playwright, context7)
model: haiku
color: green
---

# Default Server Recommendations

Determine standard servers that should always be suggested.

---

## Your Task

Find default servers to suggest based on project structure.

---

## Process

### Step 1: Check for Git

Use Glob to check if .git directory exists.

If yes → recommend github-official

### Step 2: Check for Web Framework

Read package.json (if exists) and check for web frameworks:
- next, react, vue, svelte, angular, remix, nuxt, astro

If web framework found → recommend playwright

### Step 3: Always Recommend

context7 - Useful for all projects (documentation)

### Step 4: Call mcp-find

For each determined server:

```
mcp-find(query="github-official", limit=1)
mcp-find(query="playwright", limit=3)
mcp-find(query="context7", limit=1)
```

### Step 5: Return Data

```
{
  "git_detected": true,
  "web_framework": "next",
  "default_servers": [
    {
      "name": "github-official",
      "reason": ".git directory detected",
      "category": "recommended",
      "description": "GitHub API integration",
      "required_secrets": ["github.personal_access_token"],
      "oauth_required": true
    },
    {
      "name": "playwright",
      "reason": "Web framework detected (next)",
      "category": "suggested",
      "description": "Browser automation and testing",
      "required_secrets": [],
      "oauth_required": false
    },
    {
      "name": "context7",
      "reason": "Useful for all projects",
      "category": "suggested",
      "description": "Documentation access",
      "required_secrets": [],
      "oauth_required": false
    }
  ]
}
```
