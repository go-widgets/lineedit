# lineedit

A line editor as a pure model: key events in; the line, the cursor, candidate
listings and submitted lines out. It imports only the standard library, so a
WebAssembly guest links it without growing.

```sh
go get github.com/go-widgets/lineedit
```

## What it does

- **Editing:** insert, Backspace, Delete, Left/Right, Home/End, Ctrl-A/E/B/F,
  Ctrl-U/K/W, Ctrl-L.
- **History:** Up/Down, Ctrl-P/N, bounded, exportable for persistence; reverse
  incremental search with Ctrl-R.
- **Completion through a `Completer`:**
  - a unique match is inserted with a trailing space;
  - several matches are completed to their longest common prefix;
  - a second Tab lists them in columns, or as `value  -- description` lines,
    and asks before showing a long list.
  - It understands the line's quoting.
- **Wide characters:** columns are counted with `RuneWidth`, so CJK, emoji and
  combining characters place the cursor correctly.

`ParseKeys` turns terminal input (what xterm.js's `onData` delivers) into keys,
and `Renderer` turns successive `State`s into ANSI output. A widget can ignore
both and use `Key` and `State` directly. `Reader` drives a raw-mode terminal
stream and returns one line per `ReadLine`.

```go
e := lineedit.New(lineedit.Options{Prompt: "$ ", Complete: myCompleter})
for _, k := range lineedit.ParseKeys(input) {
    if ev := e.Key(ctx, k); ev.Submit {
        run(ev.Line)
    }
}
draw(e.State())
```

## Origin

It moved here unchanged from `github.com/go-wasmcontainers/engine/lineedit`
v0.1.0. That path becomes an alias of this module. go-widgets/shellterm builds
a line-oriented terminal on it (go-widgets/toolkit#548).

## Tests

- 100 % statement coverage, gated in CI.
- Four fuzz targets, each fuzzed for 45 s on every change: `FuzzReader` (the
  raw terminal byte stream), `FuzzEditor` (keys), `FuzzSplitPartial` and
  `FuzzSplit` (lines). The corpus is committed in `testdata/fuzz`.
- CI runs on Linux (amd64 and arm64), macOS, Windows and 32-bit Linux, and
  runs govulncheck.

## Licence

BSD 3-Clause, see [LICENSE](LICENSE).
