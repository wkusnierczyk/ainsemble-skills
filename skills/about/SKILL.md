# About Stanza

When an agent handles the `about` command, it must print the following stanza. Read the version dynamically from `~/.claude/my-plugins/ainsemble/.claude-plugin/plugin.json`.

## Format

```
{agent-description}

ainsemble: AI-native development and project management agents
├─ version:    {version from plugin.json}
├─ author:     Wacław Kuśnierczyk
├─ developer:  mailto:waclaw.kusnierczyk@gmail.com
├─ source:     https://github.com/wkusnierczyk/ainsemble-skills
└─ licence:    MIT https://opensource.org/licenses/MIT
```

## Rules

- `{agent-description}` is a one-line description specific to the agent (e.g., "TPM agent — triages issues & PRs, enforces hygiene conventions, prunes merged branches, and audits project health.")
- `{version}` must be read from the plugin manifest at runtime — never hardcode it.
- Print the stanza exactly as shown. Do not add extra text, explanation, or commands.
