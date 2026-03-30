---
name: project-setup
description: How to create new projects using blueprints from the project-blueprints repo. Follow when the user asks to create, scaffold, or initialize a new project.
type: guideline
---

# Project Setup

**This is a strict guideline.** Follow these rules exactly.

New projects are created from blueprints stored at `loxosceles/project-blueprints` on GitHub.

---

## Workflow

1. **Identify the blueprint**: Ask which stack the user wants (e.g., "nextjs-sst"). Read the corresponding blueprint from `blueprints/{stack}.md` in the repo.
2. **Read the full blueprint** before starting. Understand all sections.
3. **Collect variables**: Ask for `project_name`, `git_name`, `git_email`, and any other values the blueprint requires.
4. **Execute sections in order**: Follow the blueprint step by step.
5. **Copy fragments verbatim**: Fragment files from `fragments/` are exact configs. Copy them, then replace `{{template_variables}}` with actual values.
6. **Pause on version mismatches**: If a tool (create-next-app, SST, etc.) has a new major version compared to what the blueprint specifies, stop and ask: "Should I evaluate the upgrade or use the pinned version?"
7. **Never silently modify fragments**: If a fragment doesn't work with current tool versions, report the conflict and ask.
8. **Run verification**: Execute all verification commands at the end. All must pass.
9. **Install skills and configure Kiro CLI**:
    - Run `npx skills add loxosceles/ai-dev --agent claude-code kiro-cli -y` and ask about additional third-party skills.
    - Copy agent fragments from `project-blueprints/fragments/agents/kiro/` into `~/.devcontainer-config/cache/${PROJECT}/kiro/agents/`. Custom agents require `"resources": ["skill://.kiro/skills/**/SKILL.md"]` to auto-discover skills — the fragments already include this.
    - Pre-create all host mount targets (Docker creates missing sources as root-owned, breaking permissions):
      ```bash
      PROJECT=<project-name>
      mkdir -p ~/.devcontainer-config/cache/${PROJECT}/claude
      mkdir -p ~/.devcontainer-config/cache/${PROJECT}/kiro/agents
      mkdir -p ~/.devcontainer-config/cache/${PROJECT}/kiro/settings
      mkdir -p ~/.devcontainer-config/data/${PROJECT}
      touch ~/.devcontainer-config/data/${PROJECT}/zsh_history
      touch ~/.devcontainer-config/data/${PROJECT}/zsh_history_tmux
      cp ~/.devcontainer-config/ai/agents/kiro/*.json ~/.devcontainer-config/cache/${PROJECT}/kiro/agents/
      echo '{"chat.defaultAgent":"lead-dev"}' > ~/.devcontainer-config/cache/${PROJECT}/kiro/settings/cli.json
      ```
    - Never copy agent configs into the project's `.kiro/agents/` — that causes conflicts with the host's global `~/.kiro/agents/`.

## Rules

- Execute in the current directory (must be empty or an empty git repo)
- Git remotes must use SSH, never HTTPS: `git@github.com:user/repo.git`
- Never skip verification steps
- If a step fails, present the error with options — don't silently retry
- Template variables use `{{double_braces}}` syntax
- Fragment files are the source of truth for configs — don't improvise alternatives
- **Consistency with `project-migration`**: This skill and `project-migration` must produce identical results for shared concerns (devcontainer, skills, kiro, linting, CI/CD). If you detect a discrepancy between what this skill instructs and what `project-migration` does, stop and warn the developer before proceeding.

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
