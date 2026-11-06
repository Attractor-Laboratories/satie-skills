# Satie plugin for Claude Code

One install gives Claude Code the Satie MCP server (remote, OAuth sign-in on first use), the
`satie-sound-for-worlds` skill and the `satie-sound-reviewer` agent.

```sh
claude plugin marketplace add Attractor-Laboratories/satie-skills
claude plugin install satie
```

The plugin only configures the remote server at `https://www.satie.world/mcp`; nothing runs
locally and no key is stored. The skill files here are generated from `packages/satie-mcp/skill.ts`
in the Satie repository (`npx tsx scripts/export-skill.mts`); edit them there.
