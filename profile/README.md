> [!IMPORTANT]
> This project is experimental, for real: it is a side project I build for fun. You are welcome to try it, but there is no alpha, beta or v1 yet, and I may change anything, or everything, at any time without notice. If you want to depend on it before an official release, fork it and pin your own copy.
>
> The default license for this project is FSL-1.1-ALv2 (Functional Source License with an Apache 2.0 future license). Check the `LICENSE` file in each repository before you use it.
>
> Measured with Claude Code on five repositories (three runs per task): on two medium repositories with questions about behavior it saved about 10% of turns and time and 12 to 16% of output tokens. On the other three it saved nothing, because the assistant found answers with `Grep` and `Read` and rarely called Onfly. Cost savings are unproven. Numbers and method: [evaluation](https://github.com/OnFlyApp/cli/blob/main/docs/evaluation.md).

**Onfly** gives an AI coding assistant a searchable, structural map of your codebase. It parses code into ASTs with isolated WebAssembly plugins, stores the pieces (functions, methods, types, doc sections) in a local index, and serves them over the **Model Context Protocol (MCP)** to Claude Code, Cursor, VS Code or any MCP client.

### What the assistant gets

* **`onfly_xray`**: inspect a file, folder or symbol cheaply. Long method bodies are collapsed to a one-line marker, with who calls it, what it calls, and the project rules for that path. Docblocks, attributes and signatures stay byte for byte.
* **`onfly_focus`**: search by meaning and by name. Returns the full code of the best matches, signatures of what they use, and the rules.
* **`get_tests` and `get_history`**: which tests mention a symbol, and which commits touched it.
* **Rules delivered with results**: short Markdown rules scoped to paths or chunk types ride along on the tool results, once per session.
* **Token telemetry**: `onfly stats` shows an estimate of the tokens the tools kept out of the model's context.

### How it works

* **AST-aware chunks.** Boundaries come from the parser, not from a sliding window, so a method is never cut in half.
* **CGO-free and sandboxed.** Language parsers run in [wazero](https://wazero.io) sandboxes: no Python, no Node, no host access.
* **Hybrid search.** Embeddings (Ollama, OpenAI or Voyage), full-text and symbol-name lists fused by rank, with optional one-sentence summaries per chunk.
* **Solo or team.** SQLite on your machine, or PostgreSQL with `pgvector` shared through CI.

---

### Repositories

* **[`cli`](https://github.com/OnFlyApp/cli)**: the `onfly` binary: indexer, MCP server, evaluation tools. Written in Go.
* **[`registry`](https://github.com/OnFlyApp/registry)**: the JSON manifest `onfly ext` reads to resolve parser versions and SHA-256 checksums.
* **Parsers**: one `ext-parser-<name>` repository per language, released as a WASM binary and installed with `onfly ext install <name>` (see the table below).

| Language | Install | Extensions | Repository |
|---|---|---|---|
| Go | `onfly ext install go` | `.go` | [`ext-parser-go`](https://github.com/OnFlyApp/ext-parser-go) |
| PHP | `onfly ext install php` | `.php` `.phtml` `.inc` | [`ext-parser-php`](https://github.com/OnFlyApp/ext-parser-php) |
| TypeScript | `onfly ext install typescript` | `.ts` `.tsx` `.mts` `.cts` | [`ext-parser-typescript`](https://github.com/OnFlyApp/ext-parser-typescript) |
| JavaScript | `onfly ext install javascript` | `.js` `.jsx` `.mjs` `.cjs` | [`ext-parser-javascript`](https://github.com/OnFlyApp/ext-parser-javascript) |
| Python | `onfly ext install python` | `.py` `.pyi` | [`ext-parser-python`](https://github.com/OnFlyApp/ext-parser-python) |
| Rust | `onfly ext install rust` | `.rs` | [`ext-parser-rust`](https://github.com/OnFlyApp/ext-parser-rust) |

More languages are possible: a parser is any program that turns a file into chunks ([parser contract](https://github.com/OnFlyApp/cli/blob/main/docs/parsers.md)). To have a parser listed as official, [open an issue in `cli`](https://github.com/OnFlyApp/cli/issues); how releases reach the registry is described in the [registry docs](https://github.com/OnFlyApp/registry/tree/main/docs).

---

### Getting started

You need Go and an embedder such as [Ollama](https://ollama.com) (`ollama pull qwen3-embedding:0.6b`).

```bash
git clone https://github.com/OnFlyApp/cli onfly && cd onfly && make build   # produces bin/onfly; put it on your PATH

cd /path/to/your/project
onfly init                  # writes .onfly/config.yaml, ignores .onfly/, installs parsers it detects
onfly index
onfly setup claude          # registers the MCP server and writes the usage guide into CLAUDE.md
```

Cursor and VS Code: `onfly mcp-config cursor` or `onfly mcp-config vscode`. Full documentation is in the [`cli` repository](https://github.com/OnFlyApp/cli#readme).

---

### License

The default license for this project is the **Functional Source License, Version 1.1, ALv2 Future License** (`FSL-1.1-ALv2`): free for any purpose other than a competing use, and each version converts to Apache 2.0 two years after its release. Check the `LICENSE` file in each repository, it is the one that applies there.
