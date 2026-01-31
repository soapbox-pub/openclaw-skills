---
name: mkstack
description: Create and build Nostr client websites using the MKStack template with OpenCode. Use when the user wants to create a new Nostr-powered web application, build a Nostr client, or start a project using MKStack.
---

# MKStack

MKStack is a template for building Nostr clients with compatible coding agents like OpenCode and Shakespeare. This skill provides the complete workflow for creating new MKStack projects.

## Workflow: Spawn a Subagent

**The entire MKStack workflow should run in a subagent.** This keeps the main chat clean and handles all steps (clone, initialize, build) in one session.

### Spawn Pattern

**Parent agent instructions:**
1. Calculate the full project path: `<workspace>/projects/<project-name>`
2. Calculate the full paths to the SKILL.md files for the following skills: `mkstack`, `opencode`
3. Spawn with explicit skill usage instructions:

```
sessions_spawn:
  task: "CRITICAL: You MUST read the mkstack skill (<skills>/mkstack/SKILL.md) first, then read the opencode skill (<skills>/opencode/SKILL.md). Do NOT proceed until you have read both skills. After reading them, create a new MKStack project for [describe the project]. Follow the mkstack workflow exactly - do NOT improvise or skip the OpenCode step. Project path: <workspace>/projects/<project-name>"
  label: "mkstack-<project-name>"
  runTimeoutSeconds: 660
```

**Important:**
- The subagent will handle cloning, git init, and running OpenCode
- Use descriptive project names (lowercase, hyphens for spaces)

## For the Subagent: Complete Workflow

**CRITICAL: Before starting this workflow, you MUST:**
1. Read this mkstack skill completely
2. Read the opencode skill completely  
3. Follow the steps below exactly — do NOT improvise, skip steps, or use alternative approaches

When you are the subagent executing this skill, follow these steps:

### Step 1: Clone and Initialize

Clone the MKStack template into a new project directory:

```bash
mkdir -p <workspace>/projects
cd <workspace>/projects
git clone https://gitlab.com/soapbox-pub/mkstack.git <project-name>
cd <project-name>
rm README.md
rm -rf .git
git init
git add .
git commit -m "Project created with MKStack: https://gitlab.com/soapbox-pub/mkstack.git"
```

**Important:** 
- Replace `<project-name>` with the actual project name (lowercase, hyphens for spaces)
- Replace `<workspace>` with your workspace directory
- **Always create projects in `<workspace>/projects/`** — not the home directory (`~`) or elsewhere

### Step 2: Run OpenCode

Use OpenCode to build the project based on the user's description:

```bash
cd <workspace>/projects/<project-name> && opencode run --model openrouter/anthropic/claude-sonnet-4.5 "[detailed project description]"
```

**exec parameters:**
- `pty: true` **(CRITICAL - REQUIRED! Without this, you will see NO OUTPUT and OpenCode will appear to hang!)**
- `workdir: <workspace>/projects/<project-name>`
- `yieldMs: 600000` (blocks until completion)
- `timeout: 600` (10 minutes)

**IMPORTANT:** You MUST have read the opencode skill first to understand the correct syntax and parameters. Do NOT skip this step!

### Step 3: Report Completion

After OpenCode completes, report back what was created and where the project is located.

## What MKStack Includes

MKStack projects come pre-configured with:
- **Nostr integration** - NostrProvider, hooks, and utilities
- **React + Vite** - Modern frontend stack
- **Tailwind CSS** - Utility-first styling
- **shadcn/ui components** - Pre-built UI components
- **TypeScript** - Type safety
- **Skills** - Pre-built Nostr functionality (comments, DMs, infinite scroll, AI chat)

## Important Notes

- **Always spawn a subagent** - Don't run the workflow in the main session
- **Read both skills** - Subagent needs mkstack AND opencode skills
- **Always use pty:true** - OpenCode requires a pseudo-terminal
- **Use high yieldMs** - Blocks until completion instead of backgrounding
- **Work in <workspace>/projects/** - Keep projects organized in the projects directory (inside your workspace, not home directory)
- **One project per directory** - Each MKStack project gets its own folder
- **Create projects directory if needed** - If `<workspace>/projects/` doesn't exist, create it first with `mkdir -p <workspace>/projects`