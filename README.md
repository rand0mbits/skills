# skills

Claude Code skills by [Val Schuman](https://github.com/rand0mbits), packaged as a plugin marketplace.

## Install

```
/plugin marketplace add rand0mbits/skills
/plugin install rand0mbits-skills@rand0mbits
```

Update later with `/plugin marketplace update rand0mbits`.

## Skills

| Skill | What it does | Depends on |
| --- | --- | --- |
| `readable-review` | Runs a code review behind the scenes and presents the findings as a plain-language, numbered report grouped into Fix now / Needs your decision / Low priority, ending with a concrete proposal. Also rewrites review output that already exists. | The `code-review` skill that ships with Claude Code. It can use another review skill (such as `mattpocock-skills:code-review`) when you name it. |

Skills that depend on other skills say so in the table above and in their own `SKILL.md`.

## Adding a skill

1. Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter.
2. Add `"./skills/<name>"` to the `skills` array in `.claude-plugin/plugin.json`.
3. Add a row to the table above, including what it depends on.
4. Bump `version` in `.claude-plugin/plugin.json`, because installs are cached by version.

## License

MIT
