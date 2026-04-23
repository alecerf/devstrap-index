# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).

Each JSON file in `tools/` describes a tool's full installation pipeline:
version discovery, download URL, checksum verification, and installation mode.

## Structure

```
index.json   # array of all tool definitions
```

## Adding a Tool

Create a JSON file in `tools/` following the schema, then add its path to
`index.json`. See existing definitions for examples of both `json_api` and
`github_release` source types.
