# Hypra plugin for AI coding agents

Lets Cursor, Claude Code and other agents write [Hypra](https://hypralang.dev) pages: `.hypra` files that
compile to accessible HTML, Markdown for AI agents and WebMCP tools, and that don't build when they have
mistakes like missing alt text or unlabeled fields.

It has three parts:

| Part | What it does |
| --- | --- |
| `skills/hypra` | how Hypra is written: its syntax next to HTML, layouts, components, forms, what the compiler checks, and how to work with it. Agents load it when they work on `.hypra` files |
| `rules/hypra.mdc` | the same rules in short, for Cursor, applied to `.hypra` files |
| `.lsp.json` | starts `hypra lsp`, so the agent sees the compiler's errors and warnings as it edits |

## Install

The language server needs the `hypra` compiler on your `PATH`:

```console
$ npm install --global hypra
```

or `cargo install hypra`. The plugin works without it too, with the skill and the rules only.

**Cursor:** add the plugin from [cursor.directory](https://cursor.directory), or copy `rules/hypra.mdc` into
your project's `.cursor/rules/`.

**Claude Code:**

```console
/plugin marketplace add hypralang/hypra-plugin
/plugin install hypra@hypra
```

## Use

Ask your agent for a page or a site with Hypra, for example "build a menu page for a bakery with Hypra". It
writes `.hypra` files, runs `hypra check` and fixes what the compiler reports. The
[starter](https://github.com/hypralang/hypra/tree/main/starter) shows a whole project, and
[hypralang.dev/docs](https://hypralang.dev/docs/) has the full language.

## License

MIT OR Apache-2.0
