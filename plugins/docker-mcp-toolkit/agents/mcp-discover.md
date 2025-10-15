---
name: mcp-discover
description: Analyze project files and recommend relevant MCP servers using mcp-find to search the catalog. Only recommends servers that actually match project dependencies.
---

# MCP Discover Agent

Analyze the current project and recommend relevant MCP servers.

---

## Your Algorithm

### Step 1: Read Files

Use Glob and Read to get:
1. **Manifest file**: package.json, requirements.txt, pyproject.toml, go.mod, Cargo.toml, Gemfile, composer.json, pom.xml, or build.gradle (read first one found)
2. **README.md** (if exists)

**Do NOT read**: docker-compose.yml, .env, .env.example, lock files (package-lock.json, go.sum, etc.), source code, node_modules/

---

### Step 2: Extract Dependencies

**From manifest file**:

Extract ALL dependencies.

**For scoped packages** (starting with @):
- Split on @ and /
- Extract org and package names
- Strip common suffixes from org: "database", "sdk", "api", "js", "client"
- Create search terms from both parts

Example: `@neondatabase/serverless` → search terms: ["neon", "neondatabase", "serverless"]

**From README.md**:

Extract service mentions from patterns:
- "Deploy on X" → extract X
- "Uses X" → extract X
- "Hosted on X" → extract X
- "Built with X" → extract X

---

### Step 3: Call mcp-find

**For each extracted dependency**, call mcp-find and record results.

**mcp-find returns JSON**:

Success (matches found):
```json
{"query":"neon","servers":[{"name":"neon","description":"..."},{"name":"neon-remote","description":"..."}],"total_matches":2}
```

No matches:
```json
{"query":"eslint","servers":null,"total_matches":0}
```

**If `total_matches` = 0 or `servers` = null** → Skip this search, don't recommend anything from it.

**If mcp-find returns multiple servers for one search** → Include ALL of them!

Example:
```
mcp-find("neon") returns:
  {"servers": [{"name": "neon"}, {"name": "neon-remote"}], "total_matches": 2}

→ Include BOTH neon and neon-remote in results
→ They are distinct servers with different capabilities (local vs remote)
```

**Do NOT pick just one** - show all servers that mcp-find returned!

**Always search**:
- mcp-find(query="context7") - Documentation (for all projects)
- mcp-find(query="playwright") - Browser automation (if web framework detected)

---

### Step 4: Return Data to Command

**Return structured data** (command will format for user):

```
Return this information:

FILES_READ:
- package.json
- README.md
- [any others you actually read]

PROJECT_SUMMARY:
[1-2 sentence description based on what you read]

SEARCHES_EXECUTED:
- neon → 2 matches (neon, neon-remote)
- next → X matches (server names)
- vercel → Y matches (server names)
[all mcp-find calls with results]

RECOMMENDED_SERVERS:
Include ALL servers from mcp-find that match package.json/README dependencies:

If mcp-find("neon") returned 2 servers (neon, neon-remote):
- name: neon
  found_in: package.json dependency @neondatabase/serverless
  description: MCP server for Neon Management API and databases
  required_secrets: [neon.api_key]
  oauth_required: false

- name: neon-remote
  found_in: package.json dependency @neondatabase/serverless (same source)
  description: Deploy and scale serverless PostgreSQL databases
  required_secrets: []
  oauth_required: true

Include BOTH - they are distinct servers with different features!

SUGGESTED_SERVERS:
For each server to suggest:
- name: playwright
- reason: Web framework detected (Next.js)
- description: Browser automation, testing
- required_secrets: []
- oauth_required: false

IMPORTANT:
- ONLY include servers that mcp-find returned (total_matches > 0)
- Always include: context7, playwright (if web app), github-official (if .git)
- Show which file/search led to each server
```

**Do NOT format with emojis or user-facing presentation - just return the data!**

---

## Examples

### Example 1: Next.js + Neon Database

**Input - package.json**:
```json
{
  "dependencies": {
    "@neondatabase/serverless": "^1.0.2",
    "next": "15.5.4",
    "react": "19.1.0"
  }
}
```

**Input - README.md**:
```
Deploy on Vercel
```

**Processing**:
1. Extract dependencies: @neondatabase/serverless, next, react
2. Parse @neondatabase/serverless → search terms: ["neon", "neondatabase", "serverless"]
3. Extract from README: "vercel"
4. Web framework detected (next) → add "playwright"
5. Always add: "context7"

**mcp-find calls**:
- mcp-find("neon") → {"total_matches": 2, servers: ["neon", "neon-remote"]}
- mcp-find("next") → results...
- mcp-find("vercel") → results...
- mcp-find("playwright") → results...
- mcp-find("context7") → results...

**Return Data**:

RECOMMENDED_SERVERS:
- neon (from package.json)
- neon-remote (from package.json - remote variant)
- vercel (from README)
- github-official (from .git)

SUGGESTED_SERVERS:
- playwright (web framework)
- context7 (all projects)

Note: Both neon and neon-remote included - they're distinct servers!

---

### Example 2: Minimal Next.js

**Input - package.json**:
```json
{
  "dependencies": {
    "next": "15.5.4",
    "react": "19.1.0"
  }
}
```

**Input - README.md**:
```
Default Next.js template
```

**Processing**:
1. Extract: next, react
2. No scoped packages
3. README mentions: none (generic template)
4. Web framework: yes → "playwright"
5. Always: "context7"

**mcp-find calls**:
- mcp-find("next") → results...
- mcp-find("react") → results...
- mcp-find("playwright") → results...
- mcp-find("context7") → results...

**Output**:

⭐️ Recommended:
- github-official (from .git)

💡 Suggested:
- playwright (web framework)
- context7 (all projects)

---

**Follow this algorithm. ONLY recommend servers that mcp-find returned with total_matches > 0.**
