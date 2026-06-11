# Coverity MCP Server

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that provides **Coverity Static Analysis** integration for Claude Code, GitHub Copilot, Cursor, Windsurf, Cline, OpenCode, and other MCP-compatible AI coding agents.

---

## Table of Contents

1. [Supported Platforms](#supported-platforms)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [MCP Server Tools Overview](#mcp-server-tools-overview)
5. [Agent Configuration](#agent-configuration)
   - [GitHub Copilot](#github-copilot)
   - [Claude Code](#claude-code)
   - [Cursor](#cursor)
   - [Windsurf](#windsurf)
   - [Cline](#cline)
   - [OpenCode](#opencode)
6. [Coverity Skill Guide](#coverity-skill-guide)
7. [Your First Scan](#your-first-scan)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Data Storage and Privacy](#data-storage-and-privacy)

---

## Supported Platforms

| Platform | Architecture |
|---|---|
| macOS | Apple Silicon (ARM64) |
| macOS | Intel (AMD64) |
| Windows | AMD64 |
| Linux | AMD64 |
| Linux | ARM64 |

---

## Prerequisites

### Coverity Static Analysis

- **Coverity Static Analysis 2026.3.0 or later** must be installed on the same machine where the MCP server runs.
- The Coverity command-line tools must be accessible, either via a known installation directory or via your system `PATH`.
- A valid Coverity license is required.

To verify your Coverity installation:

```bash
# Check that the coverity CLI tool is accessible
/path/to/coverity/bin/coverity --version

# Or, if coverity is on your PATH
coverity --version
```

### Node.js

Node.js 18 or later is required to run the MCP server via `npx`.

### An MCP-Compatible AI Coding Agent

You need at least one of the agents listed in the [Agent Configuration](#agent-configuration) section below, or any code agent that supports MCP servers. If your coding agent is not listed here, refer to your agent's documentation for how to configure an MCP server.

---

## Installation

### Running with npx (recommended)

No installation is required. Use `npx` in your agent configuration and the package will be downloaded and cached automatically on first use:

```bash
npx -y @black-duck/coverity-mcp-server
```

To verify the server starts correctly:

```bash
npx -y @black-duck/coverity-mcp-server -version
```

You should see output like:

```
coverity-mcp-server version 2026.3.0
```

### Global Installation

Install globally if you want the binary always available on your `PATH` or to access the bundled skill files directly:

```bash
npm install -g @black-duck/coverity-mcp-server
```

After global installation, the binary is on your `PATH`:

```bash
coverity-mcp-server -version
```

The bundled skill files are located at:

```bash
# macOS / Linux
$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md

# Windows (PowerShell)
$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md
```

---

## MCP Server Tools Overview

The Coverity MCP Server exposes five tools to your AI coding agent. The agent calls these tools automatically when you ask it to scan your code or check for issues.

### `create_project`

**Purpose:** Initializes the workspace for a project. Must be run once before any scan.

The agent will ask you for:
- The path to your Coverity installation (or confirm it is on your `PATH`)
- Clean and build commands (required for C, C++, Objective-C/C++, Kotlin, and Visual Basic; recommended but not required for Java, C#, and Go; not needed for JavaScript, Python, TypeScript, Ruby, PHP, Swift, and other interpreted languages)
- Optional Coverity Connect server URL and stream name (speeds up `HFI` incremental scans if you have a Connect server)

### `run_scan`

**Purpose:** Executes a Coverity static analysis scan on a project directory.

**Scan types:**
- `auto` (default) — the server automatically chooses full or incremental based on prior scan history
- `full` — analyzes the entire codebase; use for a comprehensive baseline
- `incremental` — analyzes only changed files; much faster for everyday development

**Scan modes (incremental only):**
- `pfi` (Perfect Fidelity Incremental) — highest accuracy, recommended before merging code
- `hfi` (High Fidelity Incremental) — fastest, ideal for pre-commit checks

**Supported languages:**

| Category | Languages |
|---|---|
| Requires build command | C, C++, Objective-C/C++, Kotlin, Visual Basic |
| Build command recommended (not required) | Java, C#, Go |
| No build command needed | JavaScript, TypeScript, Python, PHP, Ruby, Swift, Apex, Terraform/CloudFormation (Infra-as-code) |

> **Platform notes:** C# and Visual Basic are only supported on Linux AMD64 and Windows AMD64. Objective-C/C++ and Kotlin are not supported on Linux ARM64.

### `get_issues`

**Purpose:** Retrieves issues found by a previous scan with rich filtering options.

Key filters:
- `severities` — `High`, `Medium`, `Low`, `Audit`
- `kinds` — `SECURITY`, `QUALITY`
- `cwe_categories` — filter by CWE number (e.g. `"79"` for XSS, `"89"` for SQL injection)
- `only_modified` — show only issues in files changed since the last git commit
- `group_by_file` — organize results by file for targeted fixing

### `get_issue_details`

**Purpose:** Returns the full diagnostic information for a specific issue, including the complete execution path of events leading to the defect. Use this after `get_issues` to understand and fix a particular problem.

### `list_scans`

**Purpose:** Lists all previous scans for a project in reverse chronological order. Use this to review scan history or to retrieve results from an earlier scan.

---

## Agent Configuration

All agents below communicate with the MCP server over **standard input/output (stdio)**. The `npx` command handles downloading and caching the server automatically.

---

### GitHub Copilot

GitHub Copilot in Visual Studio Code supports MCP servers through VS Code's built-in MCP host (VS Code 1.99 or later required).

**Option A — Workspace configuration** (applies only to the current project)

Create or edit `.vscode/mcp.json` in your project root:

```json
{
  "servers": {
    "coverity": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

**Option B — User configuration** (applies to all projects)

Open VS Code **User Settings (JSON)** (`Ctrl+Shift+P` → *Preferences: Open User Settings (JSON)*) and add:

```json
"mcp": {
  "servers": {
    "coverity": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

After saving, open the GitHub Copilot Chat panel and you should see the Coverity tools listed as available. Enable them when prompted.

**Skill Installation:**

Install the Coverity skill for your current project or all projects according to how the MCP server was configured.

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

**Global configuration** (applies to all projects):

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.github/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.github\skills\coverity\SKILL.md`

**Workspace configuration** (applies only to the current project):

Copy the skill file to `.github/skills/coverity/SKILL.md` in your project directory.

---

### Claude Code

**Option A — CLI (recommended)**

```bash
claude mcp add --transport stdio coverity --scope user -- npx -y @black-duck/coverity-mcp-server
```

The `--scope user` flag makes the server available across all your Claude Code sessions.

**Option B — Configuration file**

Add the following to `~/.claude.json`:

```json
{
  "mcpServers": {
    "coverity": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

Restart Claude Code after making changes.

**Skill Installation:**

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.claude/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.claude\skills\coverity\SKILL.md`

---

### Cursor

**Global configuration** (applies to all projects):

- macOS / Linux: `$HOME/.cursor/mcp.json`
- Windows: `%USERPROFILE%\.cursor\mcp.json`

**Workspace configuration** (applies only to the current project):

Create `.cursor/mcp.json` in your project root.

In both cases use:

```json
{
  "mcpServers": {
    "coverity": {
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

After saving, open **Cursor Settings → MCP** to confirm the server is listed and active.

**Skill Installation:**

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

**Global configuration** (applies to all projects):

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.cursor/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.cursor\skills\coverity\SKILL.md`

**Workspace configuration** (applies only to the current project):

Copy the skill file to `.cursor/skills/coverity/SKILL.md` in your project directory.

---

### Windsurf

Note that Windsurf does not support project-specific MCP server configuration. You must configure the MCP server at the user level.

**Global configuration** (applies to all projects):

Edit the Windsurf MCP configuration file:

- macOS / Linux: `$HOME/.codeium/windsurf/mcp_config.json`
- Windows: `%USERPROFILE%\.codeium\windsurf\mcp_config.json`

Then use:

```json
{
  "mcpServers": {
    "coverity": {
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

Restart Windsurf after saving. The Coverity tools will appear in the Cascade AI panel.

**Skill Installation:**

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.codeium/windsurf/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.codeium\windsurf\skills\coverity\SKILL.md`

---

### Cline

**Global configuration** (applies to all projects):

- macOS / Linux: `$HOME/.cline/mcp.json`
- Windows: `%USERPROFILE%\.cline\mcp.json`

**Workspace configuration** (applies only to the current project):

Create `.cline/mcp.json` in your project root.

In both cases use:

```json
{
  "mcpServers": {
    "coverity": {
      "command": "npx",
      "args": ["-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

**Skill Installation:**

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

**Global configuration** (applies to all projects):

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.cline/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.cline\skills\coverity\SKILL.md`

**Workspace configuration** (applies only to the current project):

Copy the skill file to `.cline/skills/coverity/SKILL.md` in your project directory.

---

### OpenCode

**Global configuration** (applies to all projects):

OpenCode reads MCP server configuration from its config file:

- macOS / Linux: `$HOME/.config/opencode/opencode.json`
- Windows: `%USERPROFILE%\.config\opencode\opencode.json`

**Workspace configuration** (applies only to the current project):

Create `opencode.json` in your project root.

In both cases add an `mcp` section to the file:

```json
{
  "mcp": {
    "coverity": {
      "type": "local",
      "command": ["npx", "-y", "@black-duck/coverity-mcp-server"]
    }
  }
}
```

**Skill Installation:**

First, install the package globally to access the skill file:

```bash
npm install -g @black-duck/coverity-mcp-server
```

**Global configuration** (applies to all projects):

- macOS / Linux: Copy `$(npm root -g)/@black-duck/coverity-mcp-server/skills/coverity/SKILL.md` to `$HOME/.config/opencode/skills/coverity/SKILL.md`
- Windows: Copy `$(npm root -g)\@black-duck\coverity-mcp-server\skills\coverity\SKILL.md` to `%USERPROFILE%\.config\opencode\skills\coverity\SKILL.md`

**Workspace configuration** (applies only to the current project):

Copy the skill file to `.opencode/skills/coverity/SKILL.md` in your project directory.

---

## Coverity Skill Guide

The Coverity **skill** is a set of instructions that tells your AI agent exactly when and how to invoke the Coverity MCP tools — for example, automatically triggering a scan whenever you modify more than ten lines of code.

### Using the Skill Manually

With the skill installed, you can trigger a scan by simply asking:

```
Scan my code for security issues
Check my recent changes for defects
Run a Coverity analysis on this project
```

---

## Your First Scan

Once the MCP server is configured and (optionally) the skill is installed, follow these steps to run your first scan.

### 1. Open a Project

In your AI coding agent, open or navigate to the project you want to analyze.

### 2. Ask the Agent to Scan

Type a natural-language request such as:

```
Scan this project for security vulnerabilities.
My Coverity installation is at /opt/coverity/2026.3.0
```

The agent will:
1. Call `create_project` to initialize the Coverity workspace for your project (first time only).
2. Call `run_scan` to analyze your code.
3. Call `get_issues` to retrieve and summarize the findings.
4. Offer to investigate specific issues in detail using `get_issue_details`.

### 3. Review the Results

The agent will present a summary of the number of issues found organized by various characteristics such as severity, category, checker, and language. The results will also highlight issues in files you have changed (if using Git) and the most important files ranked by the severity of the issues found in them.

You can ask follow-up questions such as:

```
Show me only high-severity security issues
What issues are in the files I changed today?
Tell me more about the null pointer issue in auth.c
```

### 4. Verify Fixes

After fixing reported issues, ask the agent to verify:

```
I fixed the issues in auth.c — can you re-scan just those files?
```

The agent will run a quick (HFI) incremental scan on the modified files and confirm whether the issues are resolved. If you have configured the project with a Coverity Connect server URL and stream name, the HFI scan will be significantly faster, as it can reuse scan summary data from the server.

---

## Troubleshooting Guide

### MCP Server Does Not Start

**Symptom:** The agent reports it cannot connect to the Coverity MCP server.

**Steps:**
1. Verify Node.js 18 or later is installed: `node --version`
2. Verify `npx` is available: `npx --version`
3. Test the server manually:
   ```bash
   npx -y @black-duck/coverity-mcp-server -version
   ```
4. Check that npm can reach the registry. If behind a proxy or firewall, configure npm's proxy settings.
5. On macOS, if the underlying binary is blocked by Gatekeeper, run the server once from the terminal to trigger the security prompt, then allow it in **System Settings → Privacy & Security**.

---

### Coverity Installation Not Found

**Symptom:** The agent reports that Coverity cannot be found during project setup.

**Steps:**
1. Confirm your Coverity installation directory. Look for a `bin/coverity` (or `bin/coverity.exe`) executable inside it.
2. When prompted by the agent for the Coverity installation path, provide the full path to the installation directory (not the binary itself):
   ```
   /opt/coverity/2026.3.0
   ```
3. Alternatively, add the Coverity `bin/` directory to your system `PATH` and provide an empty string for the installation directory.

---

### Build Capture Fails (Compiled Languages)

**Symptom:** The scan fails or produces no results for C, C++, Go, or similar compiled projects.

**Steps:**
1. Test your build command in isolation before configuring it in the MCP server:
   ```bash
   cd /path/to/project
   make clean && make
   ```
2. Build commands must be a single command — do not use shell operators like `&&`. Use a Makefile target or shell script that combines steps:
   - **Wrong:** `make clean && make`
   - **Right (clean command):** `make clean`
   - **Right (build command):** `make`
3. Ensure the build command compiles all source files (not just incremental artifacts).
4. For CMake projects, run CMake's configure step before using the MCP server and provide the build command for the generated Makefiles.

---

### Scans Are Very Slow

**Symptom:** Scans take a long time.

**Recommendations:**
- Use **HFI incremental** scans (`scan_type=incremental`, `scan_mode=hfi`) for everyday development — these scan only changed files and are significantly faster.
- Use **PFI incremental** scans for pre-merge validation.
- Reserve **full scans** for establishing a baseline on a new project or after major refactoring.
- If you have a Coverity Connect server, configure the `stream_name` and `connect_url` parameters during `create_project` — this speeds up HFI scans by reusing server-side data.

---

### No Issues Found When Issues Are Expected

**Symptom:** Scan completes but reports zero issues.

**Steps:**
1. Confirm the scan type was appropriate. Incremental scans analyze only modified files; run a **full scan** for a comprehensive baseline.
2. For compiled languages, ensure the build command was run and captured all source files.
3. Check the log file for warnings or errors (see [Log Files](#log-files) below).
4. Confirm you are analyzing the correct project directory.
5. On Windows, try enabling long path support OR using the environment variable `COVERITY_COV_USER_DIR` to change the location of the directory used to store Coverity MCP server data. The default location of the intermediate directory can cause Coverity to exceed the `MAX_PATH` limit which can result in no files being captured.

---

### Skill Does Not Trigger Automatically

**Symptom:** The Coverity skill is not invoked when code changes are made.

**Steps:**
1. Confirm the skill file is in the correct location for your agent.
2. Restart your coding agent completely.
3. Confirm the Coverity MCP server is configured and connected.
4. Try manually invoking the skill: type `"scan my code"` in the chat.

---

### Permission Denied Accessing Data Directory

**Symptom:** The server cannot write to its data directory.

**Resolution:**

```bash
# macOS / Linux
mkdir -p ~/.coverity/agent
chmod 755 ~/.coverity/agent
```

On Windows, ensure your user account has write access to `%APPDATA%\Coverity\agent\`.

You can override the default data directory by setting the `COVERITY_COV_USER_DIR` environment variable to a path where you have write access.

---

### Log Files

The MCP server writes diagnostic logs to:

| Platform | Log Directory |
|---|---|
| macOS / Linux | `$HOME/.coverity/agent/logs/` |
| Windows | `%APPDATA%\Coverity\agent\logs\` |

Log files are named with a timestamp and user identity information, for example:
```
coverity-mcp-server-2026-05-14T143022-UID-501-GID-20-PID-12345.log
```

Review the most recent log file when diagnosing unexpected behavior. To enable verbose logging, pass `-debug` on the MCP server command line:

```json
"args": ["-y", "@black-duck/coverity-mcp-server", "-debug"]
```

---

### Testing the MCP Connection Manually

You can verify the MCP server responds correctly by sending it a test message from the command line:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  | npx -y @black-duck/coverity-mcp-server
```

You should receive a JSON response containing `"result"` and the server's capabilities. If you see an error or no output, check the log directory and verify prerequisites.

---

## Data Storage and Privacy

- All analysis is performed **locally on your machine**. No source code or scan results are sent to any external service.
- Scan results are stored as JSON files in:
  - macOS / Linux: `$HOME/.coverity/agent/<project-hash>/`
  - Windows: `%APPDATA%\Coverity\agent\<project-hash>\`
- The MCP server does not modify your source code or project directories.
- To remove all data collected by the MCP server, delete the agent data directory:
  ```bash
  rm -rf ~/.coverity/agent/     # macOS / Linux
  ```
  ```powershell
  Remove-Item -Recurse -Force "$env:APPDATA\Coverity\agent\"   # Windows
  ```

---

## Getting Help

- Review log files in `$HOME/.coverity/agent/logs/` (macOS/Linux) or `%APPDATA%\Coverity\agent\logs\` (Windows).
- Contact your Coverity support representative for product-specific issues. Inquiries should be directed to tech-support@blackduck.com.
- Refer to the [Model Context Protocol documentation](https://modelcontextprotocol.io/) for general MCP concepts.
