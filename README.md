# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).
See [TOOLS.md](TOOLS.md) for the list of available tools.

## Adding a tool

1. Create `tools/{name}.json` with a definition like the one below.
2. Push to `trunk` — CI formats the JSON and regenerates `tools.json` and
   `TOOLS.md` automatically.

### Example: `github_release` source with binary install

```json
{
  "name": "golangci-lint",
  "description": "Fast Go linters runner",
  "homepage": "https://github.com/golangci/golangci-lint",
  "source": {
    "type": "github_release",
    "owner": "golangci",
    "repo": "golangci-lint"
  },
  "download": {
    "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}.tar.gz",
    "checksum": {
      "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/golangci-lint-{{.Version}}-checksums.txt"
    }
  },
  "install": {
    "mode": "binary",
    "dest": "bin",
    "binary_name": "golangci-lint",
    "archive_path": "golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}/golangci-lint"
  },
  "detect": {
    "binary": "bin/golangci-lint",
    "args": ["version"],
    "version_regex": "([0-9]+\\.[0-9]+\\.[0-9]+)"
  },
  "platforms": {
    "os": ["darwin", "linux"],
    "arch": { "amd64": "amd64", "arm64": "arm64" }
  }
}
```

### Key concepts

- **Source types:** `github_release` (fetches latest tag) or `json_api` (fetches
  a JSON endpoint and extracts the version via a path expression — see
  `tools/go.json` for an example).
- **Install modes:** `directory` (extract archive to a folder) or `binary`
  (extract a single binary).
- **Download URLs** are Go templates with `{{.Version}}`, `{{.OS}}`, `{{.Arch}}`,
  `{{.Tag}}`, `{{.Filename}}`, `{{.Source.Owner}}`, `{{.Source.Repo}}`.
- **Checksums:** provide a `checksum.url` pointing to a SHA-256 sums file, or
  use `source.file_match` when the API embeds checksums (see `tools/go.json`).
- **Platforms:** `arch` maps `GOARCH` values to tool-specific names (e.g.
  `"amd64": "x64"` for Node.js).

## Self-hosting

Host your own index on any static HTTP server. devstrap needs two things:

1. A `tools.json` listing paths to tool definitions.
2. The definition files at those paths, relative to the base URL.

```sh
devstrap index update --remote https://your-server.example.com/index
```
