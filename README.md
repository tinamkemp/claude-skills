# claude-skills

Personal [Claude Code Agent Skills](https://code.claude.com/docs/en/skills) — installed at `~/.claude/skills/` so they're available across all projects on this machine.

Each skill is a folder containing a `SKILL.md` (the instructions Claude reads) plus optional `references/` and `assets/`.

## Skills

- **swot** — generates a Strengths/Weaknesses/Opportunities/Threats candidate-readiness brief for a specific job opportunity, based on an existing `about-me.md` career profile.
- **swot-refresh** — regenerates the durable `about-me.md` career profile from source documents (e.g. resume, LinkedIn, Drive docs). Only runs when explicitly requested.

## Usage

These skills are consumed by downstream project repos (e.g. [professional-swot](https://github.com/tinamkemp/professional-swot)) that hold the actual career profile and generated outputs. This repo holds only the reusable skill logic.
