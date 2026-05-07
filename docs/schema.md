# Schema reference

Every tool definition is a JSON file with six sections.

## Metadata

| Field         | Required | Description                               |
|---------------|----------|-------------------------------------------|
| `name`        | yes      | Unique tool identifier (matches filename) |
| `description` | yes      | One-line summary shown in `TOOLS.md`      |
| `homepage`    | no       | URL displayed in `TOOLS.md`               |

## Source

Tells devstrap how to discover the latest version. See
[source types](source-types.md) for full details.

| Field        | Required | Used by            | Description                         |
|--------------|----------|--------------------|-------------------------------------|
| `type`       | yes      | all                | `"github_release"` or `"json_api"`  |
| `owner`      | yes      | `github_release`   | GitHub user or organisation         |
| `repo`       | yes      | `github_release`   | GitHub repository name              |
| `url`        | yes      | `json_api`         | API endpoint returning JSON         |
| `version`    | yes      | `json_api`         | How to extract the version (see below) |
| `file_match` | no       | `json_api`         | Embedded checksum extraction (see [source types](source-types.md#file_match--embedded-checksums)) |

### `version` object (`json_api` only)

| Field          | Required | Description                                        |
|----------------|----------|----------------------------------------------------|
| `path`         | yes      | Path expression to the version string              |
| `strip_prefix` | no       | Prefix to remove (e.g. `"go"`, `"v"`)             |

Path expressions navigate JSON using `[N]` (array index) and `.field` (object
key). Example: `[0].version` → first element's `version` field.

## Download

| Field          | Required | Description                                          |
|----------------|----------|------------------------------------------------------|
| `url`          | yes      | Go template for the archive URL                      |
| `checksum.url` | no       | Go template for a SHA-256 checksums file             |

Every download is verified with SHA-256. You must provide **either**
`checksum.url` (separate checksums file) **or** `source.file_match` (embedded
checksum). See [checksums](checksums.md).

## Install

| Field              | Required       | Description                                          |
|--------------------|----------------|------------------------------------------------------|
| `mode`             | yes            | `"directory"`, `"binary"`, or `"direct"`             |
| `dest`             | `directory` only | Destination path relative to base dir                |
| `strip_components` | `directory` only | Leading path segments to strip (default `0`)       |
| `binary_name`      | `binary`, `direct` | Filename for the installed binary                    |
| `archive_path`     | `binary` only  | Go template for path to binary inside the archive    |

See [install modes](install-modes.md).

## Detect

| Field           | Required | Description                                                    |
|-----------------|----------|----------------------------------------------------------------|
| `binary`        | yes      | Path to binary relative to base dir                            |
| `args`          | yes      | Arguments to get version output (e.g. `["version"]`)           |
| `version_regex` | yes      | Regex with one capture group for the version string            |

> Double-escape backslashes in JSON: `\\d` not `\d`.

## Platforms

| Field  | Required | Description                                                        |
|--------|----------|--------------------------------------------------------------------|
| `os`   | yes      | Map from `GOOS` to tool-specific OS names                          |
| `arch` | yes      | Map from `GOARCH` to tool-specific arch names                      |

Example — tools using native OS names:

```json
"os": { "darwin": "darwin", "linux": "linux" }
```

Example — Python (python-build-standalone) uses different OS names:

```json
"os": { "darwin": "apple-darwin", "linux": "unknown-linux-gnu" }
```

Example — Node.js uses `x64` instead of `amd64`:

```json
"arch": { "amd64": "x64", "arm64": "arm64" }
```

## Template variables

All template fields (`url`, `checksum.url`, `archive_path`, `filename_contains`)
support these variables:

| Variable            | Description                              | Available             |
|---------------------|------------------------------------------|-----------------------|
| `{{.Version}}`      | Semver without `v` prefix                | always                |
| `{{.OS}}`           | Mapped operating system                  | always                |
| `{{.Arch}}`         | Mapped architecture                      | always                |
| `{{.Tag}}`          | Original Git tag                         | `github_release` only |
| `{{.Filename}}`     | Matched filename from API                | `file_match` only     |
| `{{.Source.Owner}}`  | `source.owner` value                    | when set              |
| `{{.Source.Repo}}`   | `source.repo` value                     | when set              |
