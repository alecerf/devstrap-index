# devstrap-index

Tool definitions for [devstrap](https://github.com/alecerf/devstrap).
See [TOOLS.md](TOOLS.md) for the list of available tools.

## Adding a tool

1. Create `tools/<name>.json` with the tool definition.
2. Push to `trunk` — CI formats the JSON and regenerates `tools.json` and
   `TOOLS.md` automatically.

## Documentation

| Page | Description |
|------|-------------|
| [Schema reference](docs/schema.md) | Every field explained |
| [Source types](docs/source-types.md) | `github_release` vs `json_api`, path expressions, `file_match` |
| [Checksums](docs/checksums.md) | Separate file vs embedded checksum strategies |
| [Install modes](docs/install-modes.md) | `directory` vs `binary` installation |
| [Examples](docs/examples.md) | Complete definitions for all supported patterns |

## Self-hosting

Host your own index on any static HTTP server. devstrap needs two things:

1. A `tools.json` listing paths to tool definitions.
2. The definition files at those paths, relative to the base URL.

```sh
devstrap index update --remote https://your-server.example.com/index
```
