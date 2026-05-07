# Checksums

Every download is verified with a SHA-256 checksum. Two strategies are
supported — you must use exactly one.

## Separate checksum file

Provide `download.checksum.url` pointing to a standard SHA-256 sums file (one
`<hash>  <filename>` per line). devstrap downloads it and looks up the hash for
the archive filename.

```json
"download": {
  "url": "https://example.com/releases/v{{.Version}}/tool-{{.Version}}-{{.OS}}-{{.Arch}}.tar.gz",
  "checksum": {
    "url": "https://example.com/releases/v{{.Version}}/checksums.txt"
  }
}
```

This is the most common strategy — most GitHub releases publish a checksums
file.

## Embedded checksum (`file_match`)

When the version API already includes the checksum (e.g. Go's download API),
use `source.file_match` to extract it. See
[source types — file_match](source-types.md#file_match--embedded-checksums).

With this strategy, omit `download.checksum` entirely:

```json
"download": {
  "url": "https://go.dev/dl/{{.Filename}}"
}
```

## What happens if neither is provided?

The checksum verification will pass vacuously (no checksum to compare against).
However, **you should always provide one** — integrity verification is a core
safety guarantee of devstrap.

## Raw hash files

Some tools (e.g. kubectl) publish checksum files containing only the raw hex
SHA-256 hash — no filename, no extra whitespace. devstrap detects this format
automatically: if the file contains a single 64-character hex string, it is
used directly as the expected checksum.
