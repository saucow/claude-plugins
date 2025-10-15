---
description: Discover relevant MCP servers for your current project - 5
argument-hint: "[project-path]"
allowed-tools: ["Task", "Read", "Glob", Bash(docker mcp:*)]
---

# Discover Relevant MCP Servers

Analyze your project and get personalized MCP server recommendations based on your actual dependencies and tech stack.

---

## Prerequisites Check

Check if mcp-find tool is available (indicates dynamic-tools feature is enabled).

If NOT available:
```
⚠️ This command requires dynamic-tools enabled.

Enable it: docker mcp feature enable dynamic-tools
Then restart Claude Code.
```

---

## Project Detection

Use Glob to check for project files:
- package.json, requirements.txt, go.mod, Cargo.toml, etc.
- README.md
- .git directory

If no project detected, ask user for project path or continue with generic recommendations.

If project detected:
```
✓ Project detected
Analyzing to find relevant MCP servers...
```

---

## Launch Agent

Launch the mcp-discover agent to analyze the project:

```
Use Task tool to launch: mcp-discover

Agent will:
1. Read project files (README, manifests, configs)
2. Extract technologies from files
3. Use mcp-find to search catalog
4. Filter and rank by project fit
5. Return recommendations with file evidence

Wait for agent to complete (15-25 seconds)...
```

---

## Receive Agent Data

Agent returns structured data (not formatted text).

Expected data structure:
- FILES_READ: [...]
- PROJECT_SUMMARY: "..."
- SEARCHES_EXECUTED: [{query, matches, servers}, ...]
- RECOMMENDED_SERVERS: [{name, found_in, description, secrets, oauth}, ...]
- SUGGESTED_SERVERS: [{name, reason, description, secrets, oauth}, ...]

---

## Format and Present

Transform agent data into user-friendly output:

```
┌─────────────────────────────────────────────────────┐
│ MCP Server Discovery Results                       │
└─────────────────────────────────────────────────────┘

Files Analyzed:
{for each file in FILES_READ}
- ✓ {file}

Searches Executed:
{for each search in SEARCHES_EXECUTED}
- {query} → {matches} matches {if matches > 0: list server names}

Project Summary:
{PROJECT_SUMMARY}

---

⭐️ Recommended

{for each server in RECOMMENDED_SERVERS}
• {name}
  - Found in: {found_in}
  - Capabilities: {description}
  - Setup: {if oauth: "OAuth - Run: docker mcp oauth authorize {name}"}
          {else if secrets: "Requires: {join(secrets, ', ')}"}
          {else: "No setup needed"}

💡 Suggested

{for each server in SUGGESTED_SERVERS}
• {name}
  - Why: {reason}
  - Capabilities: {description}
  - Setup: {same logic as above}

---

Summary:
- Files read: {count FILES_READ}
- Searches performed: {count SEARCHES_EXECUTED}
- Servers found: {count RECOMMENDED + SUGGESTED}
```

---

## Interactive Selection

Ask user:
```
What would you like to do?

1. Enable all recommended servers
2. Enable specific servers
3. Exit

Your choice:
```

Based on selection:
- Option 1: Enable all from RECOMMENDED_SERVERS
- Option 2: Show numbered list, user selects, enable selected
- Option 3: Exit

---

## Enable Servers

If user approved, tell agent which servers to enable.

Agent will use `docker mcp server enable <server-name>` for each server and report progress.

---

## Summary

Show final summary:
- How many servers enabled
- Restart notice (IMPORTANT!)
- Which secrets need configuration
- Next steps

```
✓ Enabled X servers (permanently)

⚠️ IMPORTANT: Restart Claude Code to activate these servers

Steps:
1. Exit Claude Code (Ctrl+C or /exit)
2. Restart: claude
3. Your new tools will be available!

After restart:
- Verify: /docker-mcp-toolkit:gateway-status
- Configure secrets if needed
```
