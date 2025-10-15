---
name: mcp-discover-readme
description: Analyze README.md for service mentions and find matching MCP servers using mcp-find
model: sonnet
color: blue
---

# README Service Analyzer

Analyze README for service/platform mentions and find matching MCP servers.

---

## Your Task

Extract service mentions from README and find matching MCP servers.

---

## Process

### Step 1: Check for README

Use Glob to check if README.md exists.

If not found, return:
```
{
  "readme_found": false,
  "service_mentions": [],
  "matched_servers": []
}
```

### Step 2: Read README

Read README.md content.

### Step 3: Extract Service Mentions

Look for these patterns:
- "Deploy on X" → extract X
- "Deployed to X" → extract X
- "Uses X" → extract X
- "Powered by X" → extract X
- "Built with X" → extract X
- "Hosted on X" → extract X

Examples:
- "Deploy on Vercel" → vercel
- "Uses Stripe for payments" → stripe
- "Hosted on AWS" → aws

### Step 4: Call mcp-find

For each extracted service, call mcp-find:

```
mcp-find(query="vercel", limit=5)
mcp-find(query="stripe", limit=5)
```

**mcp-find returns**:
- Success: `{"query":"vercel","total_matches":1,"servers":[...]}`
- No match: `{"query":"someservice","total_matches":0,"servers":null}`

### Step 5: Return Data

```
{
  "readme_found": true,
  "service_mentions": ["vercel", "stripe"],
  "searches_executed": [
    {"query": "vercel", "total_matches": 1, "servers": [{"name": "vercel"}]},
    {"query": "stripe", "total_matches": 2, "servers": [{"name": "stripe"}, {"name": "stripe-remote"}]}
  ],
  "matched_servers": [
    {
      "name": "vercel",
      "found_in": "README.md Deploy on Vercel",
      "description": "Deploy and scale web applications",
      "required_secrets": ["vercel.token"],
      "oauth_required": false
    },
    {
      "name": "stripe",
      "found_in": "README.md Uses Stripe for payments",
      "description": "Stripe API integration",
      "required_secrets": ["stripe.secret_key"],
      "oauth_required": false
    }
  ]
}
```

**ONLY include servers that mcp-find returned (total_matches > 0)**
