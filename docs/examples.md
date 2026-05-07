# Examples

Complete tool definitions for every supported combination.

## `github_release` + checksum file + binary install

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
    "os": { "darwin": "darwin", "linux": "linux" },
    "arch": { "amd64": "amd64", "arm64": "arm64" }
  }
}
```

## `json_api` + checksum file + directory install

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
    "os": { "darwin": "darwin", "linux": "linux" },
    "arch": { "amd64": "x64", "arm64": "arm64" }
  }
}
```

## `json_api` + embedded checksum (`file_match`) + directory install

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
    "os": { "darwin": "darwin", "linux": "linux" },
    "arch": { "amd64": "amd64", "arm64": "arm64" }
  }
}
```

## `github_release` + asset version extraction + directory install

For tools where the release tag is not the tool version (e.g. a date-based tag)
and the version must be extracted from asset filenames.

```json
{
  "name": "python",
  "description": "Python programming language",
  "homepage": "https://www.python.org",
  "source": {
    "type": "github_release",
    "owner": "astral-sh",
    "repo": "python-build-standalone",
    "version": {
      "asset_regex": "cpython-([0-9]+\\.[0-9]+\\.[0-9]+)\\+.*-{{.Arch}}-{{.OS}}-install_only\\.tar\\.gz",
      "pick": "highest"
    }
  },
  "download": {
    "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/{{.Tag}}/cpython-{{.Version}}+{{.Tag}}-{{.Arch}}-{{.OS}}-install_only.tar.gz",
    "checksum": {
      "url": "https://github.com/{{.Source.Owner}}/{{.Source.Repo}}/releases/download/{{.Tag}}/SHA256SUMS"
    }
  },
  "install": {
    "mode": "directory",
    "dest": "python",
    "strip_components": 1
  },
  "detect": {
    "binary": "python/bin/python3",
    "args": ["--version"],
    "version_regex": "Python ([0-9]+\\.[0-9]+\\.[0-9]+)"
  },
  "platforms": {
    "os": { "darwin": "apple-darwin", "linux": "unknown-linux-gnu" },
    "arch": { "amd64": "x86_64", "arm64": "aarch64" }
  }
}
```

## `github_release` + raw checksum + direct install

For tools distributed as a standalone binary (no archive). The checksum file
contains only the raw SHA-256 hash.

```json
{
  "name": "kubectl",
  "description": "Kubernetes command-line tool",
  "homepage": "https://kubernetes.io/docs/reference/kubectl/",
  "source": {
    "type": "github_release",
    "owner": "kubernetes",
    "repo": "kubernetes"
  },
  "download": {
    "url": "https://dl.k8s.io/release/v{{.Version}}/bin/{{.OS}}/{{.Arch}}/kubectl",
    "checksum": {
      "url": "https://dl.k8s.io/release/v{{.Version}}/bin/{{.OS}}/{{.Arch}}/kubectl.sha256"
    }
  },
  "install": {
    "mode": "direct",
    "binary_name": "kubectl"
  },
  "detect": {
    "binary": "bin/kubectl",
    "args": ["version", "--client"],
    "version_regex": "v([0-9]+\\.[0-9]+\\.[0-9]+)"
  },
  "platforms": {
    "os": { "darwin": "darwin", "linux": "linux" },
    "arch": { "amd64": "amd64", "arm64": "arm64" }
  }
}
```
