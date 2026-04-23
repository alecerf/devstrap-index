# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).

Each `.json` file at the root describes a tool's full installation pipeline:
version discovery, download URL, checksum verification, and installation mode.

## Structure

```
go.json              # one file per tool
node.json
golangci-lint.json
tools.json           # auto-generated listing of tool names
```

`tools.json` is maintained automatically by a GitHub Action — do not edit it
by hand.

## Adding a Tool

Create a `{name}.json` file at the root following the schema. On push, the
GitHub Action regenerates `tools.json`. See existing definitions for examples
of both `json_api` and `github_release` source types.
