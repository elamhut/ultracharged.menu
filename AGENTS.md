# AGENTS.md

Guidance for anyone — human or agent — editing this plugin. This file is the
single source of truth; `CLAUDE.md` only points here.

## What this is

**Ultracharged Omarchy menu**: an Omarchy shell plugin (Quickshell / QML) that
replaces the stock command menu and adds self-answering searches — calculator,
currency, units, time zones, dev utilities, process killing, links, web search,
DuckDuckGo-style bangs, `?` file search and `$` stock quotes. Fork of `dzhibas/omarchy.dzhibas.menu`.

- Plugin id: `elamhut.ultracharged.menu` (also the install directory name under
  `~/.config/omarchy/plugins/`).
- Repo: https://github.com/elamhut/ultracharged.menu

## Layout

| File | Role |
| --- | --- |
| `Menu.qml` | The menu: UI, key handling, processes it spawns, and the row builders (`calculatorRow`, `bangRow`, `urlRow`, …). |
| `MenuModel.js` | Pure logic with no QML or I/O: parsers, converters, tables (`BANGS`, `UNITS`, currencies…). Everything Menu.qml needs is listed in the export object at the bottom of the file. |
| `BarWidget.qml` | Bar button that opens the menu. |
| `manifest.json` | Plugin metadata: id, version, description (shown on the Omarchy plugins site). |
| `README.md` | User-facing feature docs. |

## How a search becomes rows

`rebuildDisplay()` in `Menu.qml` matches menu entries, then prepends the first
non-empty result of `queryRows(query)`, which calls the row builders in order.
The first builder to return a row wins. If nothing matched at all,
`webSearchRow` is added as a fallback. Bangs are first in the builder list and
also stop menu matching entirely.

To add a self-answering search:

1. Put the parsing/computation in `MenuModel.js` as a pure function and add it
   to the export object at the bottom.
2. Add a `xxxRow(query)` builder in `Menu.qml` that returns
   `root.queryRow({ kind, icon, label, detail, payload, ready })`, and add it to
   the `builders` list in `queryRows`.
3. If Enter needs new behaviour for that `kind`, handle it where `row.kind` is
   dispatched (search for `row.kind === "url"`).

## Conventions

- **Never evaluate typed text as code.** The calculator has a hand-written
  parser for this reason. Shell commands quote every interpolated value with
  `Util.shellQuote`, and spawned commands are bounded (`boundedCommand`).
- **Treat external strings as untrusted** (`.desktop` names, `ps` output,
  network data, clipboard); see the "Untrusted text" section of `MenuModel.js`.
- **No network unless the user asked for it** — currency rates are fetched only
  once a conversion is actually typed.
- **Comments explain why, not what**, in full sentences, at the density of the
  surrounding code. Match the existing tone and section headers
  (`// --- Name ---…`).
- Keep the menu responsive: no work on every keystroke that could be cached.
- The plugin must keep working on current Omarchy releases; prefer falling
  back gracefully over failing when a shell API is missing.

## When you change behaviour

- **When a feature or fix is finished, update the live Omarchy install** by
  running `omarchy restart shell`. This checkout is the installed plugin
  directory, so the restart is all it takes to load the change. Do it every
  time without being asked, then check that the shell came back up.
- Update `README.md`: each feature has a `###` section with a `Type | Get`
  table and a short paragraph.
- Bump `version` in `manifest.json` (patch for fixes, minor for features), and
  update its `description` when a feature is user-visible enough to list.
- Commit messages: imperative, one-line summary ("Add caret editing to the
  search line").

## Testing

There is no build step or test suite.

- `MenuModel.js` functions are plain JavaScript: test them with `node` by
  stripping the `.pragma` line, if any, and calling them directly.
- To try the real menu, restart the shell so the plugin reloads:

  ```bash
  omarchy restart shell
  ```
