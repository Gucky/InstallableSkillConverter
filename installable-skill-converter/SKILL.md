---
name: installable-skill-converter
description: Convert an existing skill into an installable agent skill. Use when a user wants to make a skill work with `npx skills add`, move root-level skill contents into the proper skill subfolder, create or refine a human-facing README, and validate discovery.
---

# Installable Skill Converter

Use this skill when converting an existing skill into an installable skill package for `npx skills add`.

The `npx skills` command is provided by [vercel-labs/skills](https://github.com/vercel-labs/skills), the CLI for the open agent skills ecosystem. It supports installing from local paths and remote source formats, and can target Codex, Claude Code, Cursor, Gemini CLI, and many other agents.

Assume the current working directory is the existing skill repository unless the user gives another path. The folder usually starts with one root-level `SKILL.md` and maybe has additional files on root level as well as subfolders.

## Core Rules

- Do not commit, squash, force-push, or rewrite version-control history.
- Treat the files on disk as the source of truth. Do not reason from staged, unstaged, dirty, or clean version-control state.
- The job is structural: make the skill installable, create or refine the root README, and validate discovery.
- The root README is the only normal content file this workflow may create or update. It is the human-facing project page and should include a short skill summary plus installation instructions. If a README already exists, preserve its existing summary and description unless the user asks for broader changes; normally, only add or update the section that explains how to install the skill.
- Do not rewrite the skill's working instructions or bundled knowledge. Treat existing `SKILL.md`, `references/`, `scripts/`, and similar skill-owned content as static unless the user explicitly asks for content changes.
- Keep the root README for humans. Keep `SKILL.md` for agents.
- Move the installable skill content into a subfolder named after the skill, using the normalized `name` from `SKILL.md` frontmatter.
- Do not move files or folders that belong to version control systems, root `README.md`, license files, or repository metadata into the skill subfolder.
- If the skill has no icon asset or `agents/openai.yaml` has no icon fields, ask the user whether they want to provide or create an icon before finishing the polish pass. Add or wire icon metadata only after the user agrees.

## Step 1: Inspect

1. Find skill definitions:
   - `rg --files -g 'SKILL.md'`
   - There should normally be exactly one `SKILL.md`.
2. Read the `SKILL.md` frontmatter:
   - `name`
   - `description`
3. Read existing root `README.md` and `agents/openai.yaml` if present. Do not inspect version-control files for skill content.
4. Check whether assets include an icon, commonly under `assets/`, and whether `agents/openai.yaml` references it.

If more than one `SKILL.md` exists, stop and ask whether the repository should become a multi-skill package or whether only one skill should be converted.

## Step 2: Move Skill Content

Create a subfolder named after the skill.

Move skill-owned content into that subfolder without editing its contents:

- `SKILL.md`
- `agents/`
- `assets/`
- `scripts/`
- `references/`
- other folders or files that are directly part of the skill runtime or skill reference material

Keep repository-level content at the root:

- `README.md`
- files and folders that belong to version control systems
- `LICENSE`, `LICENSE.md`, or `COPYING`
- package or repository metadata, if present

If the skill is already in the correct subfolder, do not move it again. Continue with README and validation.

## Step 3: Check For Structural Risks

After moving files, inspect for risks but do not rewrite the skill's content automatically.

Check and report:

1. `agents/openai.yaml`
   - Icon paths should be relative to the skill directory, e.g. `./assets/icon.png`.
   - If icon paths are missing or now point nowhere, ask before changing metadata.
2. `SKILL.md`
   - If it references files that no longer exist at the expected relative path, warn the user.
   - Do not rewrite `SKILL.md` unless the user explicitly asks.
3. Scripts and bundled references
   - If they appear to assume the old repository root, warn the user.
   - Do not patch scripts or reference files unless the user explicitly asks.
4. Version-control ignore files
   - Keep them at root.
   - Adjust ignore patterns only when needed for the structural move and only if the change is clearly repository-level, not skill-content-level.

If a root README exists, warn the user that it remains at the package root for humans and is not installed as part of the skill folder. If the skill depends on information from that README, the user must decide whether to copy that information into `SKILL.md` or `references/`; do not do that migration automatically.

## Step 4: Write The README

Create or update the root `README.md` as a human-facing project page. If no README exists, create one with a short skill summary, installation instructions, usage examples, relevant safety notes, and requirements. If a README already exists, preserve its existing summary and description unless the user asks for broader changes; normally, add or update only the installation section.

Keep it short and useful. For a new README, pull the summary from `SKILL.md` frontmatter and a light read of the skill body, but do not rewrite the skill itself. Avoid internal implementation details such as complete repository trees, every bundled script, or agent-only workflow instructions.

Use this structure unless the skill clearly needs a different one:

````markdown
# <Display Name> Agent Skill

<One or two short paragraphs explaining what the skill does and when it is useful.>

## Installing <Display Name>

Install the skill with `npx`:

```bash
npx skills add <source> --skill <skill-name>
```

To install it globally for Codex:

```bash
npx skills add <source> --skill <skill-name> --agent codex --global
```

For a specific agent:

```bash
npx skills add <source> --skill <skill-name> --agent claude-code
```

For all supported agents:

```bash
npx skills add <source> --skill <skill-name> --agent '*'
```

If `npx` is not available, install Node.js first. On macOS with Homebrew:

```bash
brew install node
```

## Using <Display Name>

In Codex, you can trigger the skill directly:

```text
$<skill-name>
```

You can also ask naturally:

```text
Use the <Display Name> skill to ...
```

<A short bullet list of what the skill helps with.>

## Safety Model

<Only include safety notes that matter to a user installing the skill.>

## Requirements

- <Runtime/tool requirement>
- <Access requirement>
- An AI coding assistant that supports agent skills
````

Choose the most concrete source value available. A source can be a local path or any remote source format supported by the Skills CLI. If no final source is known yet, use a clear placeholder and state that it needs to be replaced with the source the user wants to install from.

Do not include generic template guidance like "if your repository has a different owner"; the README is for this specific skill package.

## Step 5: Validate Installability

Run a local discovery smoke test:

```bash
npx --yes skills add <local-repo-path> --list
```

Expected result:

- Source validates.
- Exactly the intended skill is found.
- The listed skill name matches `SKILL.md` frontmatter.

If `npx` cannot run because of network or sandbox restrictions, ask for the required approval and retry. If validation still cannot run, report that clearly.

## Step 6: Report

Summarize:

- new repository layout
- README created or changed
- validation result
- whether an icon is present or still needs a decision
- any warnings about root README contents not being installed with the skill
- any path or script assumptions the user may need to review

Do not commit unless the user asks.
