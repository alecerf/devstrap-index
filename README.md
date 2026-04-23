# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).

Each JSON file in `tools/` describes a tool's full installation pipeline:
version discovery, download URL, checksum verification, and installation mode.

## Structure

```
index.json          # manifest listing all tool definition files
tools/
  go.json           # json_api source, file_match checksum, directory install
  node.json         # json_api source, sha256 file checksum, directory install
  golangci-lint.json # github_release source, binary install
```

## Adding a Tool

Create a JSON file in `tools/` following the schema, then add its path to
`index.json`. See existing definitions for examples of both `json_api` and
`github_release` source types.
