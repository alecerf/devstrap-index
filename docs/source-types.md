# Source types

The `source` section tells devstrap how to discover the latest version of a
tool. Two strategies are supported.

## `github_release`

Queries the GitHub Releases API and extracts the latest tag. The `v` prefix is
stripped automatically. This is the simplest option for tools hosted on GitHub.

```json
"source": {
  "type": "github_release",
  "owner": "golangci",
  "repo": "golangci-lint"
}
```

No `version` object is needed — the tag **is** the version.

> **Note:** this uses the unauthenticated GitHub API, which is rate-limited to
> 60 requests/hour per IP.

## `json_api`

Fetches any public JSON endpoint and extracts the version using a path
expression.

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

### Path expressions

Path expressions navigate into a parsed JSON value. Two operators are supported
and can be chained:

| Syntax   | Meaning              | Example                |
|----------|----------------------|------------------------|
| `[N]`    | Index into an array  | `[0]` — first element  |
| `.field` | Access an object key | `.version`             |

`[0].version` means: take the first array element, then read its `version` key.

### `file_match` — embedded checksums

Some APIs (e.g. Go's download API) include the download filename **and** its
SHA-256 checksum in the same response. Use `file_match` to extract both,
avoiding a separate checksum download.

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

| Field               | Description                                                       |
|---------------------|-------------------------------------------------------------------|
| `list_path`         | Path expression to the array of file objects                      |
| `filename_field`    | JSON key holding the filename in each object                      |
| `filename_contains` | Go template — the first file whose name contains this match wins  |
| `checksum_field`    | JSON key holding the SHA-256 checksum in the matched object       |

When `file_match` is present:

- The matched filename is available as `{{.Filename}}` in download templates.
- The checksum is used automatically — omit `download.checksum`.

See also: [checksums](checksums.md).
