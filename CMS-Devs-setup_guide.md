# CMS Devs Setup Guide

This guide walks through setting up Claude Code and the complete CMS development environment — including the CLI, VS Code extension, Figma MCP integration, and custom AI agents for module and page building. Follow the steps in order on a new machine or when onboarding a new CMS developer.

**Estimated setup time:** 20–30 minutes

---

## Prerequisites

- VS Code installed
- Access to Claude Code Team Plan
- Figma account with Dev Mode access

---

## Step 1: Install Claude Code

### 1a. Install the Claude Code CLI

The Claude Code CLI is the foundation for all agent-based workflows. Install it globally:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify the installation:

```bash
claude --version
```

You should see a version number (e.g., `v1.2.3`). If this fails, see [Troubleshooting: CLI installation fails](#troubleshooting).

### 1b. Install the VS Code Extension

1. Open VS Code
2. Go to the **Extensions** tab (`Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on Mac)
3. Search for `Claude Code` — look for the official Anthropic extension
4. Click **Install**
5. After installation completes, you'll see the Claude Code icon in the VS Code sidebar (it looks like a C)
6. Click the Claude Code icon to open the panel
7. Click **Sign in** and authenticate with your Anthropic account
   - You'll be directed to a browser to complete authentication
   - Use your Team Plan account credentials
8. Once signed in, you'll see "Team Plan" confirmation in the panel

> **Note:** Claude Code includes built-in access to Claude Sonnet 4.6 and Claude Opus 4.7 via the Team Plan — no separate API key configuration needed.

---

## Step 2: Connect Figma MCP Server

The Figma MCP (Model Context Protocol) server allows Claude Code to inspect and interact with your Figma designs. This is essential for design-aware development.

### Prerequisites

- A Figma account (free tier works)
- Dev Mode enabled on your Figma plan (required for MCP access)
- A Figma design file open in your browser

### Connection Steps

1. **Open Figma in your browser** and navigate to a design file
2. **Enter Dev Mode:**
   - In the top-right corner of Figma, click the dropdown next to your profile
   - Select **Dev Mode**
3. **Enable the MCP Server:**
   - In the right sidebar, scroll down and look for **MCP** section
   - Click **Enable desktop MCP server**
   - A confirmation message will appear with a localhost URL (e.g., `http://127.0.0.1:3845/mcp`)
4. **Copy the URL** from the confirmation message
5. **Register the MCP in Claude Code:**

   In your terminal, run:
   ```bash
   claude mcp add --transport http figma http://127.0.0.1:3845/mcp
   ```

   Or, add it to your `.claude/settings.json`:
   ```json
   {
       "mcpServers": {
           "figma": {
               "type": "http",
               "url": "http://127.0.0.1:3845/mcp"
           }
       }
   }
   ```

6. **Verify the connection:**
   - In Claude Code, run `/mcp` to see the list of active servers
   - Figma should appear in the list with status `connected`

> **Important:** The Figma MCP server runs while Figma is open in Dev Mode. If you close Dev Mode or disconnect, you'll need to re-enable it and update the URL in Claude Code.

---

## Step 3: Set Up Custom Agents

> **Prerequisites before this step:**
> - Boot the local environment
> - AI Toolset Setup Guide (`Step 1 and 2`) completed
> - Boilerplate cloned (includes `CLAUDE.md` and the agent files below)

### What Are Agents?

Agents are specialized Claude Code prompts that guide AI assistance for specific workflows. The boilerplate ships with two agents already in place:

1. **`.claude/agents/wp-module-builder.md`** — helps design, build, and test reusable CMS modules
2. **`.claude/agents/wp-page-builder.md`** — helps structure and build full CMS pages using existing modules

No manual setup is required — Claude Code automatically picks up any agent file in `.claude/agents/`.

### 3a. Verify Both Agents Are Detected

1. Open Claude Code in VS Code
2. Run `/agents` in the chat panel
3. Confirm both `wp-module-builder` and `wp-page-builder` are listed

### 3b. Invoke Agents in Claude Code

Agents are invoked with natural language — name the agent directly in your prompt:

**To use the Module Builder Agent:**
- *"Use the wp-module-builder agent to create a hero section module with image and text."*

**To use the Page Builder Agent:**
- *"Use the wp-page-builder agent to build a landing page using the hero, features, and CTA modules."*

Claude Code may also auto-delegate to the matching agent based on your prompt, even without naming it explicitly.

---

## Step 4: Model Selection and Configuration

### Recommended Models

- **Sonnet 4.6** (default) — suitable for most CMS tasks (T1–T2 complexity)
- **Opus 4.7** — for complex architectural decisions or multi-file refactoring (T3 tasks)

### Switch Models in Claude Code

**To switch to Sonnet 4.6:**
1. In the Claude Code panel, type `/model`
2. Select **Switch model**
3. Choose **Sonnet 4.6**

**To use Opus 4.7 (for complex tasks):**
1. Type `/model` and select **Opus 4.7**
2. Use intentionally — Opus 4.7 consumes tokens ~40% faster than Sonnet

After finishing complex work, switch back to Sonnet 4.6 to manage token usage.

---

## Step 5: Verify Your Setup

Run through this checklist to confirm everything is configured correctly:

| Item | How to verify |
|------|--------------|
| **Claude Code CLI installed** | `claude --version` returns a version number |
| **Claude Code VS Code extension installed** | Claude Code icon visible in VS Code sidebar |
| **Signed in to Team Plan** | Account name shown in Claude Code panel |
| **Figma MCP connected** | `claude mcp` shows Figma with status `connected` |
| **Module Builder agent detected** | `/agents` in Claude Code lists `wp-module-builder` |
| **Page Builder agent detected** | `/agents` in Claude Code lists `wp-page-builder` |
| **Agents callable from Claude Code** | Naming the agent in a prompt (e.g. "Use the wp-module-builder agent...") invokes it |

---

### Module Development Workflow

```
1. Open Claude Code
2. Ask Claude Code to use the wp-module-builder agent in your prompt
3. Describe the module you want to build (name, purpose, layout)
4. The agent will guide you through:
   - Component structure and file organization
   - Styling approach (CSS, Tailwind, BEM, etc.)
   - Accessibility requirements
   - Testing and validation
5. Review the generated code and ask for refinements as needed
6. Test the module in your local dev environment
```

### Page Development Workflow

```
1. Open Claude Code
2. Ask Claude Code to use the wp-page-builder agent in your prompt
3. Describe the page structure (sections, modules to use)
4. The agent will:
   - Suggest which modules to compose
   - Build page templates using existing modules
   - Handle layout, spacing, and responsive design
   - Ensure accessibility and performance
5. Assemble the page in your CMS
6. Test across devices and browsers
```

---

## Troubleshooting

### CLI Installation Fails

**Error:** `npm ERR! code EACCES` or permission denied

**Solution:**
```bash
sudo rm -rf /opt/homebrew/lib/node_modules/@anthropic-ai/claude-code  # macOS
npm cache clean --force
npm install -g @anthropic-ai/claude-code
claude --version
```

**Alternative (use a Node version manager like nvm):**
```bash
nvm install 16
npm install -g @anthropic-ai/claude-code
```

---

### Claude Code Extension Not Signing In

**Error:** "Sign in failed" or "Unable to authenticate"

**Solution:**
1. Confirm your Anthropic account has Team Plan access (contact your admin)
2. Sign out in VS Code (`Cmd+Shift+P` → "Claude Code: Sign Out")
3. Sign back in
4. Check your network — corporate firewalls may block Anthropic domains
5. Try opening a private/incognito browser window for authentication

---

### Figma MCP Not Connecting

**Error:** `claude mcp` shows Figma as `disconnected` or missing

**Solution:**
1. Verify Figma is open in a browser and you're in Dev Mode
2. Confirm the MCP server is enabled in the right sidebar
3. Copy the localhost URL again — it may have changed
4. Run `claude mcp remove figma` then re-add with the new URL:
   ```bash
   claude mcp add --transport http figma <new-localhost-url>
   ```
5. Check Claude Code's output panel for detailed error messages

---

### Agents Not Appearing in Claude Code

**Error:** Agents don't show up when running `/agents`, or naming the agent in a prompt doesn't invoke it

**Solution:**
1. Verify the agent files exist in the project:
   ```bash
   ls .claude/agents/
   ```
   You should see `wp-module-builder.md` and `wp-page-builder.md`
2. Confirm each file has valid YAML frontmatter (`name` and `description` fields) at the top
3. Restart the Claude Code VS Code window (`Cmd+Shift+P` → "Reload Window")
4. Try referencing the agent again

---

### `CLAUDE.md` Not Being Picked Up

**Error:** Claude Code doesn't reference project context from `CLAUDE.md`

**Solution:**
1. Verify the file exists at `/CLAUDE.md` in the project root
2. Confirm it's not listed in `.gitignore`:
   ```bash
   grep CLAUDE.md .gitignore  # should return nothing
   ```
3. Restart the Claude Code session
4. Try reloading the VS Code window

---

## Additional Resources

- [Claude Code Documentation](https://claude.ai/help)
- [MCP Documentation](https://modelcontextprotocol.io)
- [CMS AI Playbook](https://github.com/dev-offshorly/cms-ddt-ai-playbook/)
- [Basic Techniques Guide](https://github.com/maekooffshorly/ai-excellence-playbook/blob/main/docs/08-basic-techniques.md)

---

## Support

If you encounter issues not covered here:

1. Check the troubleshooting section above
2. Review Claude Code's output panel for detailed error messages
3. Run `claude --help` to see available CLI commands
4. Contact your team's Claude Code admin for Team Plan account issues
