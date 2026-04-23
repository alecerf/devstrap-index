# Install modes

The `install` section tells devstrap how to place the downloaded archive on
disk. All paths are relative to the base directory (`~/Workspace` by default).

## `directory`

Extracts the entire archive into a folder. Use this for tools that ship a
complete directory tree (runtime, standard library, bundled binaries).

```json
"install": {
  "mode": "directory",
  "dest": "node",
  "strip_components": 1
}
```

| Field              | Required | Description                                              |
|--------------------|----------|----------------------------------------------------------|
| `dest`             | yes      | Destination folder name (e.g. `"go"`, `"node"`)         |
| `strip_components` | no       | Leading path segments to strip during extraction. Default `0`. |

Most archives wrap everything in a top-level directory like
`node-v22.0.0-darwin-arm64/`. Set `strip_components` to `1` to remove it so the
contents go directly into `dest`.

## `binary`

Extracts the archive into a temporary directory and copies a single binary to
the destination. Use this for tools distributed as a standalone executable.

```json
"install": {
  "mode": "binary",
  "dest": "bin",
  "binary_name": "golangci-lint",
  "archive_path": "golangci-lint-{{.Version}}-{{.OS}}-{{.Arch}}/golangci-lint"
}
```

| Field          | Required | Description                                              |
|----------------|----------|----------------------------------------------------------|
| `dest`         | yes      | Destination directory (e.g. `"bin"`)                     |
| `binary_name`  | yes      | Filename for the installed binary                        |
| `archive_path` | yes      | Go template for the path to the binary inside the archive |

> **Tip:** download and extract the archive manually to inspect its structure
> and determine the correct `archive_path`.
