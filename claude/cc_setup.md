# Claude Code Setup Documentation

This document captures all custom installed components in my Claude Code setup for reinstallation purposes.

## Marketplaces

Marketplaces are plugin repositories that must be added before installing plugins from them.

| Marketplace Name | GitHub Repository | Add Command |
|------------------|-------------------|-------------|
| anthropic-agent-skills | [anthropics/skills](https://github.com/anthropics/skills) | `/plugin marketplace add anthropics/skills` |
| claude-code-plugins | [anthropics/claude-code](https://github.com/anthropics/claude-code) | `/plugin marketplace add anthropics/claude-code` |
| every-marketplace | [EveryInc/every-marketplace](https://github.com/EveryInc/every-marketplace) | `/plugin marketplace add https://github.com/EveryInc/every-marketplace.git` |
| browser-tools | [browserbase/agent-browse](https://github.com/browserbase/agent-browse) | `/plugin marketplace add browserbase/agent-browse` |
| dev-browser-marketplace | [sawyerhood/dev-browser](https://github.com/sawyerhood/dev-browser) | `/plugin marketplace add sawyerhood/dev-browser` |
| thedotmack | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | `/plugin marketplace add thedotmack/claude-mem` |

## Enabled Plugins

Plugins provide skills, commands, agents, and MCP servers.

| Plugin Name | Marketplace | Version | Description | Install Command |
|-------------|-------------|---------|-------------|-----------------|
| example-skills | anthropic-agent-skills | 00756142ab04 | Anthropic's example skills (algorithmic-art, brand-guidelines, canvas-design, doc-coauthoring, docx, frontend-design, internal-comms, mcp-builder, pdf, pptx, skill-creator, slack-gif-creator, theme-factory, web-artifacts-builder, webapp-testing, xlsx) | `/plugin install example-skills@anthropic-agent-skills` |
| frontend-design | claude-code-plugins | 1.0.0 | Create distinctive, production-grade frontend interfaces with high design quality | `/plugin install frontend-design@claude-code-plugins` |
| browser-automation | browser-tools | 0.0.1 | Automate web browser interactions using natural language via CLI commands (Stagehand CLI) | `/plugin install browser-automation@browser-tools` |
| dev-browser | dev-browser-marketplace | 45c3e37c9bdd | Browser automation plugin with persistent pages, flexible execution, and LLM-friendly DOM snapshots | `/plugin install dev-browser@dev-browser-marketplace` |
| claude-mem | thedotmack | 7.4.0 | Persistent memory system for Claude Code - seamlessly preserve context across sessions | `/plugin install claude-mem@thedotmack` |

**Note:** compounding-engineering@every-marketplace is listed in settings.json but may need manual installation:
```
/plugin install compound-engineering@every-marketplace
```
- Version: 2.16.0
- Description: AI-powered development tools. 27 agents, 19 commands, 13 skills, 2 MCP servers for code review, research, design, and workflow automation.
- Includes MCP servers: pw (Playwright), context7

## User Skills (Custom/Local)

Skills installed directly in `~/.claude/skills/` (not from plugins).

| Skill Name | Directory | Description |
|------------|-----------|-------------|
| teach | `~/.claude/skills/teach/` | Socratic teaching method for mastering complex subjects through guided discovery and first principles reasoning |
| frontend-design | `~/.claude/skills/frontend-design/` | Create distinctive, production-grade frontend interfaces with high design quality |
| youtube-transcript | `~/.claude/skills/youtube-transcript/` | Extract transcripts from YouTube videos (requires scripts in `scripts/` subdirectory) |
| ux-designer | `~/.claude/skills/ux-design/ux-designer/` | Expert UI/UX design guidance for building unique, accessible, and user-centered interfaces |

## MCP Servers (from Plugins)

| Server Name | Plugin Source | Type | Configuration |
|-------------|---------------|------|---------------|
| pw (Playwright) | compound-engineering | stdio | `npx -y @playwright/mcp@latest` |
| context7 | compound-engineering | http | `https://mcp.context7.com/mcp` |
| mem-search | claude-mem | MCP | Memory search tools via claude-mem plugin |

**Note:** Additional MCP servers may be configured in your global settings or per-project configurations.

## Custom Configuration

### Status Line
Custom status line script at `~/.claude/scripts/statusline.sh`:
- Displays current directory name with folder icon

### Settings (settings.json)
```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/scripts/statusline.sh",
    "padding": 0
  },
  "enabledPlugins": {
    "example-skills@anthropic-agent-skills": true,
    "frontend-design@claude-code-plugins": true,
    "compounding-engineering@every-marketplace": true,
    "browser-automation@browser-tools": true,
    "dev-browser@dev-browser-marketplace": true,
    "claude-mem@thedotmack": true
  }
}
```

### Notable Permissions (settings.local.json)
- AWS MCP server tools (ccapi)
- Linear MCP server tools
- WebFetch for specific domains
- Various Bash commands for tmux, git, brew, etc.

## Reinstallation Steps

1. **Add Marketplaces:**
   ```bash
   claude /plugin marketplace add anthropics/skills
   claude /plugin marketplace add anthropics/claude-code
   claude /plugin marketplace add https://github.com/EveryInc/every-marketplace.git
   claude /plugin marketplace add browserbase/agent-browse
   claude /plugin marketplace add sawyerhood/dev-browser
   claude /plugin marketplace add thedotmack/claude-mem
   ```

2. **Install Plugins:**
   ```bash
   claude /plugin install example-skills@anthropic-agent-skills
   claude /plugin install frontend-design@claude-code-plugins
   claude /plugin install browser-automation@browser-tools
   claude /plugin install dev-browser@dev-browser-marketplace
   claude /plugin install claude-mem@thedotmack
   claude /plugin install compound-engineering@every-marketplace
   ```

3. **Copy User Skills:**
   - Copy skill directories to `~/.claude/skills/`
   - Ensure `SKILL.md` files are present in each skill directory
   - Install any skill-specific dependencies (e.g., youtube-transcript requires `youtube-transcript-api`)

4. **Restore Settings:**
   - Copy `settings.json` and `settings.local.json` to `~/.claude/`
   - Copy custom scripts to `~/.claude/scripts/`

5. **Restart Claude Code** to apply all changes.

## Dependencies

Some components require external dependencies:
- **youtube-transcript skill:** `uv add youtube-transcript-api`
- **browser-automation skill:** Node.js, npm, Chrome browser, Anthropic API key
- **dev-browser skill:** Node.js v18+, npm
- **claude-mem plugin:** Bun (auto-installed), uv (auto-installed), Node.js

## File Locations Reference

| Component | Location |
|-----------|----------|
| Global settings | `~/.claude/settings.json` |
| Local permissions | `~/.claude/settings.local.json` |
| User skills | `~/.claude/skills/` |
| Installed plugins | `~/.claude/plugins/installed_plugins.json` |
| Known marketplaces | `~/.claude/plugins/known_marketplaces.json` |
| Marketplace repos | `~/.claude/plugins/marketplaces/` |
| Plugin cache | `~/.claude/plugins/cache/` |
| Custom scripts | `~/.claude/scripts/` |
| Claude-mem database | `~/.claude-mem/claude-mem.db` |
| CLAUDE.md instructions | `~/.claude/CLAUDE.md` |
