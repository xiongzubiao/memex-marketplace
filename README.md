# memex-marketplace

A Claude Code plugin marketplace for [memex](https://www.npmjs.com/package/@xiongzubiao/memex) — a personal-wiki storage and search engine for AI agents.

## Install

**One command:**

```bash
npm install -g @xiongzubiao/memex
```

`npm install -g` drops a `memex` binary on PATH, downloads the embedding
model + tokenizer + ONNX Runtime into `~/.memex/`, and (because it's a
global install) runs `memex install` automatically — registering hooks
+ skills with every agent CLI it detects (Claude Code, Codex, Gemini
CLI, OpenClaw, Hermes). Re-running is idempotent.

Then verify:

```bash
memex doctor
```

Should report `PASS` on the embedding model, tokenizer, ONNX Runtime,
and at least one agent CLI on PATH.

**Alternative — Claude Code marketplace path:**

```bash
claude plugin marketplace add https://github.com/xiongzubiao/memex-marketplace
claude plugin install memex
```

Use the full `https://` URL (not the `github.com/owner/repo` shorthand) —
Claude Code tries SSH for the shorthand form, which fails for users without an
SSH key configured for GitHub. The marketplace path registers the plugin with
Claude Code only; for the other four agents use `memex install` from the
`npm install -g` path above.

## What this repo contains

Only the marketplace pointer (`.claude-plugin/marketplace.json`). The plugin itself — Rust CLI binary, postinstall scripts, hooks, skills — lives on the public npm registry:

| Package | What's in it |
|---------|--------------|
| [`@xiongzubiao/memex`](https://www.npmjs.com/package/@xiongzubiao/memex) | Wrapper + hooks + skills + postinstall (downloads the embedding model, tokenizer, ONNX Runtime) |
| [`@xiongzubiao/memex-darwin-arm64`](https://www.npmjs.com/package/@xiongzubiao/memex-darwin-arm64) | Apple Silicon binary |
| [`@xiongzubiao/memex-darwin-x64`](https://www.npmjs.com/package/@xiongzubiao/memex-darwin-x64) | Intel Mac binary |
| [`@xiongzubiao/memex-linux-x64`](https://www.npmjs.com/package/@xiongzubiao/memex-linux-x64) | Linux x86_64 binary |
| [`@xiongzubiao/memex-linux-arm64`](https://www.npmjs.com/package/@xiongzubiao/memex-linux-arm64) | Linux ARM64 binary |

`npm install` picks the right platform package automatically via `os`/`cpu` constraints.

## What memex does

- Auto-ingests agent session transcripts (Claude Code, Codex, Gemini CLI, OpenClaw, Hermes) into a local Markdown wiki — via session-end / `/new` / `/reset` hooks, depending on each agent's lifecycle.
- Provides slash commands inside each agent:
  - `/memex-query` — search your wiki and synthesize an answer using your agent.
  - `/memex-ingest` — read external source material and create wiki pages.
  - `/memex-brainstorm` — multi-LLM brainstorming via external CLIs.
- Hybrid retrieval: BM25 (sqlite-fts) + dense vector (sqlite-vec) with reciprocal-rank fusion.
- Long-lived background daemon that keeps the embedding model warm across queries.

After install, each agent's session lifecycle hook spawns the daemon automatically and ingests transcripts on session end. You can also use the CLI directly: `memex query`, `memex write`, `memex backfill claude-code`, etc.

## Platforms

| Platform | Status |
|----------|--------|
| macOS (Apple Silicon) | Supported |
| macOS (Intel) | Supported |
| Linux x86_64 | Supported |
| Linux ARM64 | Supported |
| Windows | Not yet supported — daemon requires Unix sockets + flock. Run inside WSL2 as a workaround. |

## Uninstall

**From the `npm install -g` path:**

```bash
memex uninstall                          # stop daemon, strip hooks + skills
npm uninstall -g @xiongzubiao/memex      # remove the binary
rm -rf ~/.memex/                         # optional: remove model + ORT + wiki
```

Order matters: `memex uninstall` needs `memex` on PATH, so run it before the
`npm uninstall -g`. Add `--purge` to `memex uninstall` to also delete
`~/.memex/` (your wiki + DB + models) in one step.

**From the Claude Code marketplace path:**

```bash
claude plugin uninstall memex@xiongzubiao
claude plugin marketplace remove xiongzubiao
rm -rf ~/.memex/                         # optional
```

`claude plugin marketplace remove` may need to run twice — the first
invocation sometimes leaves the entry in `~/.claude/settings.json`'s
`extraKnownMarketplaces` and a second invocation cleans it up.

## Source code

The memex source repository is private. If you want access, reach out to [@xiongzubiao](https://github.com/xiongzubiao).

## License

MIT — see [LICENSE](./LICENSE).
