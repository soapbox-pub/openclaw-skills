---
name: opencode
description: Use OpenCode for AI-powered code generation and development tasks. OpenCode can create files, build projects, and write code across multiple languages. Use when you need to generate code, create web pages, build applications, or perform coding tasks that would take multiple iterations.
---

# OpenCode Skill

Use OpenCode for AI-powered code generation and development tasks.

## What is OpenCode?

OpenCode is an AI coding assistant (similar to Cursor, Windsurf, etc.) that can:
- Generate code and create files
- Build complete projects (web pages, apps, etc.)
- Edit existing code
- Work with multiple programming languages
- Execute code and iterate on results

## When to Use

Use OpenCode for:
- Creating new projects (websites, apps, scripts)
- Generating code files
- Building prototypes quickly
- Code-heavy tasks that would take multiple iterations

**Don't use for:**
- Simple file operations (use write/edit tools instead)
- Tasks that don't involve coding
- When you just need to read/analyze existing code

## Installation

Check if OpenCode is installed:
```bash
which opencode
```

If not installed, install it:
```bash
# Install OpenCode
curl -fsSL https://opencode.ai/install | bash

# Create symlink to make it accessible in PATH
ln -sf ~/.opencode/bin/opencode ~/.local/bin/opencode

# Verify installation
opencode --version
```

## Configuration

### 1. Provider Setup

Configure OpenRouter (or another provider):

```bash
# Create config file
mkdir -p ~/.config/opencode
cat > ~/.config/opencode/opencode.json << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "options": {
        "apiKey": "sk-or-v1-YOUR_API_KEY_HERE"
      }
    }
  }
}
EOF
```

**For the skill:** When a user wants to configure a provider:
1. Ask for the API key
2. Create or update `~/.config/opencode/opencode.json`
3. Verify with `opencode models <provider>`

Alternative: Use environment variables instead of config file:
```bash
export OPENROUTER_API_KEY="sk-or-v1-YOUR_KEY"
export ANTHROPIC_API_KEY="sk-ant-YOUR_KEY"
```

### 2. Model Selection

**Before running OpenCode**, ask which model to use unless the user has a known preference.

Check available models:
```bash
opencode models openrouter
```

**Suggested models by task complexity:**
- **Complex coding tasks**: `openrouter/anthropic/claude-sonnet-4.5`
- **Simple tasks**: `openrouter/anthropic/claude-haiku-4.5`
- **OpenAI models**: `openrouter/openai/gpt-4o`

Always specify the model with `--model` flag.

## Usage Pattern

### Recommended Approach: Spawn Subagent

For most OpenCode tasks, **spawn a subagent** to keep the main chat clean:

```
sessions_spawn:
  task: "Use OpenCode to create a landing page with contact form at /tmp/my-project"
  label: "opencode-landing-page"
  runTimeoutSeconds: 660
```

The subagent should:
1. Create project directory
2. Run OpenCode with appropriate settings
3. Report results back to main session

### Direct Execution Pattern

When running OpenCode in a subagent (or directly):

```bash
exec:
  command: >
    cd /path/to/project &&
    opencode run --model openrouter/anthropic/claude-sonnet-4.5 "Task description"
  pty: true
  timeout: 600
  yieldMs: 600000
```

**Key parameters:**
- `pty: true` - **Required** to see streaming output
- `timeout: 600` - 10 minutes (adjust based on complexity)
- `yieldMs: 600000` - Block until completion (matches timeout in ms)
- `--model` - Always specify the model

**Important**: Without `pty: true`, you won't see OpenCode's output. The process will still work and create files, but you'll have no visibility into progress.

### Checking Results

After OpenCode completes, check what was created:

```bash
# List files
ls -la /path/to/project/

# Read key files
cat /path/to/project/index.html

# Report results
# Mention what files were created and their purpose
```

## Common Workflows

### Create a New Project

```bash
# Create directory
mkdir -p /tmp/my-project
cd /tmp/my-project

# Run OpenCode
opencode run --model openrouter/anthropic/claude-sonnet-4.5 \
  "Create a landing page for a coffee shop with menu, about section, and contact form. Use modern CSS, make it mobile responsive."
```

### Add to Existing Project

```bash
# Navigate to project
cd /existing/project

# Let OpenCode analyze and extend
opencode run --model openrouter/anthropic/claude-sonnet-4.5 \
  "Add a dark mode toggle to this website"
```

### File Attachments

Attach specific files for context:

```bash
opencode run --model openrouter/anthropic/claude-sonnet-4.5 \
  --file design.png \
  "Create a landing page matching this design"
```

## Output Handling

### What to Expect

OpenCode streams output character-by-character (with `pty: true`):
- Progress indicators (e.g., "| Write path/to/file")
- Status messages
- Questions (rare, usually just proceeds)
- Completion confirmation

### Parsing Results

After execution, OpenCode will have:
- Created/modified files in the project directory
- Provided a summary of what was done

Report to the user:
- What files were created/modified
- Brief description of the result
- Location of the project
- Any next steps (e.g., "Open index.html in a browser")

## Configuration Options

See [OpenCode Config Docs](https://opencode.ai/docs/config/) for full reference.

Common config options in `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openrouter/anthropic/claude-sonnet-4.5",
  "provider": {
    "openrouter": {
      "options": {
        "apiKey": "{env:OPENROUTER_API_KEY}"
      }
    }
  },
  "tools": {
    "write": true,
    "bash": true
  },
  "permission": {
    "edit": "auto",
    "bash": "ask"
  }
}
```

**Variable substitution:**
- `{env:VAR_NAME}` - Load from environment variable
- `{file:path}` - Load from file

## Troubleshooting

### No Output Visible

**Problem**: Process runs but you see no output  
**Solution**: Add `pty: true` to exec parameters

### Process Hangs

**Problem**: OpenCode runs for 10+ minutes with no files created  
**Solution**: 
- Check API key is configured correctly
- Verify model is accessible: `opencode models <provider>`
- Try a simpler task first to test setup

### Authentication Issues

**Problem**: "No provider configured" or authentication errors  
**Solution**:
- Check `~/.config/opencode/opencode.json` exists
- Run `opencode auth list` to see configured providers
- Verify API key is valid

### Files Not Created

**Problem**: OpenCode completes but no files in directory  
**Solution**:
- Check you're in the right directory
- Look for error messages in output
- Try with `--format json` to see structured output

## Tips

1. **Be specific in prompts** - Include details about tech stack, styling, features
2. **Start simple** - Test setup with simple tasks before complex projects
3. **Use subagents** - Keep main chat clean, let subagents handle the work
4. **Check models** - More capable models cost more but produce better results
5. **Project context** - OpenCode works better when run from project directory
6. **Timeouts** - Complex tasks may need 10-15 minutes, adjust timeout accordingly

## Reference

- **Docs**: https://opencode.ai/docs
- **CLI Reference**: https://opencode.ai/docs/cli/
- **Config Schema**: https://opencode.ai/config.json
- **Models**: Run `opencode models` to see available models