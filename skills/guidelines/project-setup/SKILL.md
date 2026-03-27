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
9. **Install skills**: Run `npx skills add loxosceles/ai-dev --yes` and ask about additional third-party skills.

## Rules

- Execute in the current directory (must be empty or an empty git repo)
- Never skip verification steps
- If a step fails, present the error with options — don't silently retry
- Template variables use `{{double_braces}}` syntax
- Fragment files are the source of truth for configs — don't improvise alternatives

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
