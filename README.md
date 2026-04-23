# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).
See [TOOLS.md](TOOLS.md) for the list of available tools.

## Adding a tool

1. Create `tools/<name>.json`.
2. Push to `trunk` — CI formats the JSON and regenerates `tools.json` and
   `TOOLS.md` automatically.

Each definition has six sections: **metadata**, **source**, **download**,
**install**, **detect**, and **platforms**. The rest of this document explains
every field and all supported combinations.

---

## Metadata

| Field         | Required | Description                              |
|---------------|----------|------------------------------------------|
| `name`        | yes      | Unique tool identifier (matches filename) |
| `description` | yes      | One-line summary shown in `TOOLS.md`      |
| `homepage`    | no       | URL displayed in `TOOLS.md`               |

---

## Source — version discovery

The `source` section tells devstrap how to find the latest version.

### `github_release`

Queries the GitHub Releases API (`/repos/{owner}/{repo}/releases/latest`) and
extracts the tag name. The `v` prefix is stripped automatically.

| Field   | Required | Description                 |
|---------|----------|-----------------------------|
| `type`  | yes      | `"github_release"`          |
| `owner` | yes      | GitHub user or organisation |
| `repo`  | yes      | GitHub repository name      |

The `version` object is **not needed** — the tag is the version.

```json
"source": {
  "type": "github_release",
  "owner": "golangci",
  "repo": "golangci-lint"
}
```

### `json_api`

Fetches any JSON endpoint and navigates to the version value using a path
expression.

| Field                    | Required | Description                                             |
|--------------------------|----------|---------------------------------------------------------|
| `type`                   | yes      | `"json_api"`                                            |
| `url`                    | yes      | API endpoint returning JSON                             |
| `version.path`           | yes      | Path expression to the version string (see [Path expressions](#path-expressions)) |
| `version.strip_prefix`   | no       | Prefix to remove from the extracted value (e.g. `"go"`, `"v"`) |

```json
"source": {
  "type": "json_api",
  "url": "https://nodejs.org/dist/index.json",
  "version": {
    "path": "[0].version",
    "strip_prefix": "v"
  }
}
```

#### `file_match` — embedded checksums

Some APIs embed the download filename and checksum alongside the version (e.g.
the Go downloads API). Use `file_match` to extract both in the same request.
When present, you can omit `download.checksum` entirely.

| Field                | Required | Description                                                      |
|----------------------|----------|------------------------------------------------------------------|
| `list_path`          | yes      | Path expression to the array of file objects                     |
| `filename_field`     | yes      | JSON key holding the filename in each file object                |
| `filename_contains`  | yes      | Go template that must appear in the filename (used for matching) |
| `checksum_field`     | yes      | JSON key holding the SHA-256 checksum in the matched file object |

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

#### Path expressions

Path expressions navigate into a parsed JSON value. Two operators are supported:

| Syntax    | Meaning                            | Example        |
|-----------|------------------------------------|----------------|
| `[N]`     | Index into a JSON array            | `[0]`          |
| `.field`  | Access an object key               | `.version`     |

They can be chained: `[0].version` means "first array element, then its
`version` key". `[0].files` + `[0].files[2].sha256` would also work.

---

## Download

The `download` section describes where to fetch the archive and how to verify
its integrity. **Every download is verified with a SHA-256 checksum.**

| Field          | Required | Description                                           |
|----------------|----------|-------------------------------------------------------|
| `url`          | yes      | Go template for the archive URL                       |
| `checksum.url` | no       | Go template for a SHA-256 checksums file (see below)  |

There are two checksum strategies:

1. **Separate checksum file** — provide `checksum.url` pointing to a standard
   SHA-256 sums file (one `<hash>  <filename>` per line). devstrap downloads
   this file and looks up the archive filename.
2. **Embedded checksum** — use `source.file_match` (see above). The checksum is
   extracted from the same API response. No `checksum` object needed.

You must use **one or the other**. If neither is present, the download cannot be
verified and will fail.

```json
"download": {
  "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/tool-{{.Version}}-{{.OS}}-{{.Arch}}.tar.gz",
  "checksum": {
    "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/v{{.Version}}/checksums.txt"
  }
}
```

### Template variables

All `url`, `checksum.url`, `archive_path`, and `filename_contains` fields are
Go templates. The following variables are available:

| Variable           | Description                                                  | Example value         |
|--------------------|--------------------------------------------------------------|-----------------------|
| `{{.Version}}`     | Semver version without `v` prefix                            | `1.64.8`              |
| `{{.OS}}`          | Operating system (mapped through `platforms.os`)             | `darwin`              |
| `{{.Arch}}`        | Architecture (mapped through `platforms.arch`)               | `arm64`               |
| `{{.Tag}}`         | Original Git tag (`github_release` only)                     | `v1.64.8`             |
| `{{.Filename}}`    | Matched filename (`json_api` with `file_match` only)         | `go1.24.2.darwin-arm64.tar.gz` |
| `{{.Source.Owner}}`| `source.owner` value                                         | `golangci`            |
| `{{.Source.Repo}}` | `source.repo` value                                          | `golangci-lint`       |

> **Tip:** `{{.Filename}}` is only populated when using `file_match`. For
> `github_release` sources, build the filename manually in the URL template.

---

## Install

The `install` section tells devstrap how to place the downloaded archive on
disk. All paths are relative to the devstrap base directory (`~/Workspace` by
default).

### `directory` mode

Extracts the entire archive into a folder. Use this for tools that ship a full
directory tree (runtime, stdlib, binaries, etc.).

| Field              | Required | Description                                                  |
|--------------------|----------|--------------------------------------------------------------|
| `mode`             | yes      | `"directory"`                                                |
| `dest`             | yes      | Destination folder name (e.g. `"go"`, `"node"`)             |
| `strip_components` | no       | Number of leading path segments to strip during extraction (like `tar --strip-components`). Default `0`. |

```json
"install": {
  "mode": "directory",
  "dest": "node",
  "strip_components": 1
}
```

Most archives wrap everything in a top-level folder (e.g. `node-v22.0.0-darwin-arm64/`). Set `strip_components` to `1` to remove it.

### `binary` mode

Extracts the archive and copies a single binary to a destination directory. Use
this for tools distributed as a single executable.

| Field          | Required | Description                                                   |
|----------------|----------|---------------------------------------------------------------|
| `mode`         | yes      | `"binary"`                                                    |
| `dest`         | yes      | Destination directory (e.g. `"bin"`)                          |
| `binary_name`  | yes      | Filename for the installed binary                             |
| `archive_path` | yes      | Go template for the path to the binary inside the extracted archive |

```json
"install": {
  "mode": "binary",
  "dest": "bin",
  "binary_name": "golangci-lint",
  "archive_path": "golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}/golangci-lint"
}
```

> **Tip:** extract the archive locally and inspect the directory structure to
> figure out the correct `archive_path`.

---

## Detect — version detection

The `detect` section tells devstrap how to check the currently installed
version. It runs the binary and parses the output with a regex.

| Field           | Required | Description                                                          |
|-----------------|----------|----------------------------------------------------------------------|
| `binary`        | yes      | Path to the binary relative to base directory                        |
| `args`          | yes      | Arguments passed to the binary (e.g. `["version"]`, `["-v"]`)       |
| `version_regex` | yes      | Regex with exactly **one capture group** extracting the version string |

The captured group should match a semver-like string. Any leading `v` is
stripped automatically.

```json
"detect": {
  "binary": "go/bin/go",
  "args": ["version"],
  "version_regex": "go([0-9]+\\.[0-9]+\\.[0-9]+)"
}
```

> **Tip:** run the tool's version command locally and test your regex against
> its actual output. Remember to double-escape backslashes in JSON (`\\d` not
> `\d`).

---

## Platforms

The `platforms` section declares which OS/architecture combinations the tool
supports.

| Field  | Required | Description                                                       |
|--------|----------|-------------------------------------------------------------------|
| `os`   | yes      | List of supported operating systems using Go names (`darwin`, `linux`) |
| `arch` | yes      | Map from Go `GOARCH` values to the tool's architecture names      |

The `arch` map is the key mechanism: it translates the system architecture into
whatever the tool uses in its download URLs. For example, Node.js uses `x64`
instead of `amd64`:

```json
"platforms": {
  "os": ["darwin", "linux"],
  "arch": {
    "amd64": "x64",
    "arm64": "arm64"
  }
}
```

If the tool uses Go's standard names, the mapping is identity:

```json
"arch": { "amd64": "amd64", "arm64": "arm64" }
```

---

## Complete examples

### `github_release` + checksum file + binary install

The most common pattern for GitHub-hosted CLI tools.

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

### `json_api` + checksum file + directory install

For tools with a JSON version API and a separate checksums file.

```json
{
  "name": "node",
  "description": "Node.js JavaScript runtime",
  "homepage": "https://nodejs.org",
  "source": {
    "type": "json_api",
    "url": "https://nodejs.org/dist/index.json",
    "version": {
      "path": "[0].version",
      "strip_prefix": "v"
    }
  },
  "download": {
    "url": "https://nodejs.org/dist/v{{.Version}}/node-v{{.Version}}-{{.OS}}-{{.Arch}}.tar.gz",
    "checksum": {
      "url": "https://nodejs.org/dist/v{{.Version}}/SHASUMS256.txt"
    }
  },
  "install": {
    "mode": "directory",
    "dest": "node",
    "strip_components": 1
  },
  "detect": {
    "binary": "node/bin/node",
    "args": ["-v"],
    "version_regex": "v?([\\d.]+)"
  },
  "platforms": {
    "os": ["darwin", "linux"],
    "arch": { "amd64": "x64", "arm64": "arm64" }
  }
}
```

### `json_api` + embedded checksum (`file_match`) + directory install

For APIs that include the download filename and checksum in the version
response. No separate checksum download needed.

```json
{
  "name": "go",
  "description": "The Go programming language",
  "homepage": "https://go.dev",
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
  },
  "download": {
    "url": "https://go.dev/dl/{{.Filename}}"
  },
  "install": {
    "mode": "directory",
    "dest": "go",
    "strip_components": 1
  },
  "detect": {
    "binary": "go/bin/go",
    "args": ["version"],
    "version_regex": "go([0-9]+\\.[0-9]+\\.[0-9]+)"
  },
  "platforms": {
    "os": ["darwin", "linux"],
    "arch": { "amd64": "amd64", "arm64": "arm64" }
  }
}
```

---

## Self-hosting

Host your own index on any static HTTP server. devstrap needs two things:

1. A `tools.json` listing paths to tool definitions.
2. The definition files at those paths, relative to the base URL.

```sh
devstrap index update --remote https://your-server.example.com/index
```
