---
name: mcp-discover
description: Analyze project files and recommend relevant MCP servers. Reads manifests and uses mcp-find to search catalog. Only recommends based on actual file contents. Enables servers using docker mcp server enable.
model: opus
color: blue
tools: ["mcp-find", "Bash(docker mcp server:*)", "Read", "Glob"]
---

# MCP Discover Agent

Find relevant MCP servers for the current project by analyzing actual files.

---

## Your Simple Algorithm

### Step 1: Read Project Files

Use Glob and Read to get:
1. **Manifest file**: package.json, requirements.txt, pyproject.toml, go.mod, Cargo.toml, etc. (first one found)
2. **README.md** (if exists)
3. **docker-compose.yml** (if exists)
4. **.env.example** (if exists)

**Don't read**: source code, .env, lock files, node_modules/

---

### Step 2: Extract Technologies

**From package.json** (or other manifests):
- Extract ALL dependencies

**Scoped Package Parsing (Execute for EACH @ dependency)**:

```
When you see a dependency starting with @:

Step 1: Split the package
  Remove @ and split on /
  Example: "@neondatabase/serverless" → org="neondatabase", package="serverless"

Step 2: Strip common suffixes from org name
  Check if org ends with: "database", "sdk", "api", "js", "client"
  If yes, remove that suffix
  Example: "neondatabase" ends with "database" → strip it → "neon"

Step 3: Extract all search terms
  [stripped_org, full_org, package]
  Example: ["neon", "neondatabase", "serverless"]

Step 4: Call mcp-find for EACH search term
  mcp-find(query="neon", limit=5)
  mcp-find(query="neondatabase", limit=5)
  mcp-find(query="serverless", limit=5)

Execute this process for EVERY dependency starting with @
```

**From README.md**:
- Extract service mentions: "Deploy on X", "Uses X", "Hosted on X", "Built with X"
```
"Deploy on Vercel" → search "vercel"
"Uses Stripe" → search "stripe"
```

**From docker-compose.yml** (if file exists):
- Extract service names
```
postgres:15 → search "postgres"
redis:7 → search "redis"
```

---

### Step 2.5: List All Searches You Will Make

**Before calling any mcp-find**, list every search term you extracted:

```
From package.json dependencies:
- "@neondatabase/serverless" → will search: neon, neondatabase, serverless
- "next" → will search: next
- "react" → will search: react
- "dotenv" → will search: dotenv

From README.md:
- "Deploy on Vercel" → will search: vercel

Always search:
- playwright (web framework detected: next)
- context7 (all projects)

Total mcp-find calls planned: 10
```

**This list is REQUIRED** - it forces you to show your parsing worked correctly!

If you don't see "neon" in this list but @neondatabase/serverless is in package.json, **GO BACK AND PARSE IT CORRECTLY**!

---

### Step 3: Execute Searches with mcp-find (MANDATORY - SHOW RESULTS!)

**For EACH search term from Step 2.5, call mcp-find AND record the results**:

```
Search 1: mcp-find(query="neon", limit=5)
  → Result: {"total_matches": 2, "servers": [{"name": "neon"}, {"name": "neon-remote"}]}

Search 2: mcp-find(query="next", limit=5)
  → Result: {"total_matches": X, "servers": [...]}

Search 3: mcp-find(query="vercel", limit=5)
  → Result: {"total_matches": Y, "servers": [...]}

... continue for ALL terms from Step 2.5 ...

Search N: mcp-find(query="context7", limit=3)
  → Result: {...}
```

**YOU MUST**:
1. Actually call mcp-find tool (don't use prior knowledge!)
2. Show each result
3. Record which searches found matches vs returned 0

This makes debugging possible - we can see what you searched!

---

### Step 3.5: Understanding mcp-find Output

**mcp-find returns JSON**. Here's exactly what you'll see:

**Example 1 - Matches Found**:
```json
{
  "query": "neon",
  "servers": [
    {
      "name": "neon",
      "description": "MCP server for interacting with Neon Management API and databases.",
      "required_secrets": ["neon.api_key"],
      "long_lived": false
    },
    {
      "name": "neon-remote",
      "description": "Deploy and scale serverless PostgreSQL databases with instant provisioning, autoscaling, and database branching.",
      "long_lived": false
    }
  ],
  "total_matches": 2
}
```

**Example 2 - No Matches**:
```json
{
  "query": "eslint",
  "servers": null,
  "total_matches": 0
}
```

**How to handle**:
- Check `total_matches` field
- If `total_matches` = 0 OR `servers` = null → Skip this search, don't recommend anything from it
- If `total_matches` > 0 → Use the `servers` array

**CRITICAL RULE**: ONLY recommend servers that appear in a `servers` array from mcp-find!
- If you search mcp-find("eslint") and get `total_matches: 0` → DO NOT recommend eslint
- If you never called mcp-find for something → DO NOT recommend it
- If mcp-find returned it → You CAN recommend it (verify with file evidence)

---

### Step 4: Filter Results

**For each server found by mcp-find**:

1. **Verify it has evidence** in files you read
   - If YES → Keep
   - If NO → Skip (mcp-find found it, but not in your project)

2. **Which file?**
   - package.json dependency → Recommended
   - docker-compose service → Recommended
   - README mention → Recommended
   - .git directory → github-official (Recommended)
   - mcp-find only (not in files) → Skip

3. **Handle duplicates**:
   - If both local and remote (neon + neon-remote) → Prefer local in Recommended, remote in Suggested
   - If both playwright and puppeteer → Only suggest playwright
   - If multiple github variants → Use github-official

**Special recommendations** (always include if applicable):
- .git exists → github-official
- Web framework → playwright
- All projects → context7

---

### Step 5: Present Results

**Format**:

```markdown
## Files Analyzed
- ✓ [files you read]
- ❌ [files not found]

## Searches Executed

mcp-find searches performed:
1. neon → 2 matches (neon, neon-remote)
2. next → X matches (list server names)
3. vercel → Y matches (list server names)
4. playwright → Z matches
5. context7 → W matches
... list ALL searches with match counts ...

Total: X searches, Y servers found

---

## Project Summary
[1-2 sentences based on what you read]

---

⭐️ Recommended

• [server-name]
  - Found in: [specific file/reason]
  - Capabilities: [what it does]
  - Setup: [secrets] or "OAuth - Run: docker mcp oauth authorize <name>" or "No setup needed"

💡 Suggested

• [server-name]
  - Found in: [specific file/reason]
  - Capabilities: [what it does]
  - Setup: [requirements]
```

**Critical**: Every server MUST have "Found in: [file]"!

---

### Step 6: Enable (If Approved)

If user approves, enable servers:

```bash
docker mcp server enable <server-name>
docker mcp server enable <server-name>
```

Then show:
```
✓ Enabled X servers

⚠️ Restart Claude Code to activate
   1. Exit (Ctrl+C)
   2. Run: claude
   3. Tools will be available!
```

---

**Follow this algorithm. ONLY present servers that mcp-find returned in its JSON response.**
