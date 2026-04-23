# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).

Each JSON file in `tools/` describes a tool's complete installation pipeline —
version discovery, archive download, checksum verification, and installation.

## Repository structure

```
tools/                          # one definition per tool
  go.json
  node.json
  golangci-lint.json
tools.json                      # auto-generated listing (do not edit)
.github/workflows/tools.yml    # CI that regenerates tools.json
```

`tools.json` is regenerated automatically on every push that touches
`tools/*.json`. It contains the paths to all tool definitions:

```json
["tools/go.json", "tools/golangci-lint.json", "tools/node.json"]
```

## Adding a tool

1. Create `tools/{name}.json` following the schema below.
2. Push to `trunk`. The GitHub Action formats the JSON and updates `tools.json`.

## Definition schema

Every definition has six top-level sections:

| Section       | Purpose                                                    |
|---------------|------------------------------------------------------------|
| `name`        | Tool identifier (must match the filename without `.json`)  |
| `description` | Short human-readable description                           |
| `source`      | How to discover the latest version                         |
| `download`    | Archive URL template and optional checksum file            |
| `install`     | How to extract and place the artifact                      |
| `detect`      | How to detect the currently installed version              |
| `platforms`   | Supported OS/arch combinations with optional name mappings |

### Source types

**`json_api`** — fetch a JSON endpoint and extract the version with a path expression:

```json
"source": {
  "type": "json_api",
  "url": "https://go.dev/dl/?mode=json",
  "version": {
    "path": "[0].version",
    "strip_prefix": "go"
  },
  "file_match": {
    "list_path": "[0].files",
    "filename_field": "filename",
    "filename_contains": "{{.OS}}-{{.Arch}}.tar.gz",
    "checksum_field": "sha256"
  }
}
```

`file_match` is optional — use it when the API response embeds filenames and
checksums (e.g. the Go downloads API). When present, the matched checksum is
used directly and no separate checksum file is needed.

**`github_release`** — query the GitHub releases API for the latest tag:

```json
"source": {
  "type": "github_release",
  "owner": "golangci",
  "repo": "golangci-lint"
}
```

The `v` prefix is stripped automatically.

### Download

URLs are Go templates with access to `{{.Version}}`, `{{.OS}}`, `{{.Arch}}`,
`{{.Tag}}`, `{{.Filename}}`, `{{.Source.Owner}}`, and `{{.Source.Repo}}`.

```json
"download": {
  "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}.tar.gz",
  "checksum": {
    "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/golangci-lint-{{.Version}}-checksums.txt"
  }
}
```

Omit `checksum` when using `source.file_match` (the checksum is embedded in
the API response).

### Install modes

**`directory`** — extract the archive into `<baseDir>/<dest>`:

```json
"install": {
  "mode": "directory",
  "dest": "go",
  "strip_components": 1
}
```

**`binary`** — extract a single binary from the archive into `<baseDir>/<dest>`:

```json
"install": {
  "mode": "binary",
  "dest": "bin",
  "binary_name": "golangci-lint",
  "archive_path": "golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}/golangci-lint"
}
```

### Detect

Runs a command and matches the version with a regex (first capture group):

```json
"detect": {
  "binary": "go/bin/go",
  "args": ["version"],
  "version_regex": "go([0-9]+\\.[0-9]+\\.[0-9]+)"
}
```

### Platforms

Lists supported operating systems and maps `GOARCH` values to tool-specific
architecture names:

```json
"platforms": {
  "os": ["darwin", "linux"],
  "arch": {
    "amd64": "amd64",
    "arm64": "arm64"
  }
}
```

Use the arch map to translate when a tool uses non-standard names (e.g.
`"amd64": "x64"` for Node.js).

## Self-hosting

You can host your own index on any static HTTP server. devstrap only needs:

1. A `tools.json` file listing the paths to tool definitions.
2. The tool definition files themselves, reachable at those paths relative to
   the base URL.

Point devstrap at your index with:

```sh
devstrap index update --remote https://your-server.example.com/index
```
