# skills

A collection of skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Codex, Cursor, Windsurf, and other AI coding agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [react-performance-profiler](SKILLS/react-performance-profiler/) | Analyze React DevTools Profiler exports, identify render bottlenecks, and apply targeted optimizations |

## Structure

Each skill lives in its own directory under `SKILLS/` and contains a `SKILL.md` that defines when the skill applies, what inputs it needs, and the knowledge it provides.

```
SKILLS/
  {skill-name}/
    SKILL.md          # Skill definition (frontmatter + instructions)
    scripts/          # Optional helper scripts
```

## Usage

```bash
npx skills add https://github.com/lloydhumphreys/skills --skill react-performance-profiler
```

## Adding a Skill

1. Create a directory under `SKILLS/` using kebab-case
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`, `metadata`) and structured sections
3. Keep `SKILL.md` focused — under 500 lines — so it fits in an agent's context window
4. Add helper scripts under `scripts/` if the skill needs them

## License

MIT
