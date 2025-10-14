---
description: Discover relevant MCP servers for your current project - 650
argument-hint: "[project-path]"
allowed-tools: ["Task", "Read", "Glob"]
---

# Discover Relevant MCP Servers

Analyze your project and get personalized MCP server recommendations based on your actual dependencies and tech stack.

---

## Prerequisites Check

Check if mcp-find tool is available (indicates dynamic-tools feature is enabled).

If NOT available:
```
⚠️ This command works best with dynamic-tools enabled.

Enable it: docker mcp feature enable dynamic-tools
Then restart Claude Code.

Continue with bash fallback? [y/n]
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

## Present Recommendations

Agent returns recommendations. Display them to the user in a clear format.

Expected structure:
- Files analyzed list
- Project summary
- Recommended servers (with file evidence)
- Suggested servers (with file evidence)

---

## Interactive Selection

Ask user:
```
What would you like to do?

1. Enable recommended servers
2. Select specific servers
3. Exit

Your choice:
```

Based on selection:
- Option 1: Agent enables all recommended servers using mcp-add
- Option 2: Show list, user selects, agent enables selected
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
