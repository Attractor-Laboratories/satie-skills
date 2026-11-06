# Satie skills

Give a 3D world its sound with the [Satie](https://www.satie.world) MCP server. This repository holds the
`satie-sound-for-worlds` skill (the long-form doctrine a coding agent reads: briefing, behavior discovery,
the glue per engine, listening and release, recovery) and the Claude Code plugin that installs the server,
the skill and the sound reviewer agent together.

## Any agent that reads skills

```sh
npx skills add Attractor-Laboratories/satie-skills
```

The server also serves the same skill over MCP (`skills/list`, `skills/get`).

## Claude Code plugin

```sh
claude plugin marketplace add Attractor-Laboratories/satie-skills
claude plugin install satie
```

The plugin configures the remote server at `https://www.satie.world/mcp` (OAuth sign-in on first use);
nothing runs locally and no key is stored.

## Connect without the plugin

See https://www.satie.world/developers/mcp.md for Claude Code, Codex, Cursor, VS Code and `npx add-mcp`.

These files are generated from the Satie repository (`packages/satie-mcp/skill.ts`, `scripts/export-skill.mts`,
`scripts/skills-repo.mjs`). Edit them there.
