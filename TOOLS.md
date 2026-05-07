# Available tools

> Auto-generated from tool definitions. Do not edit.

| Tool | Description | Install | Platforms |
|------|-------------|---------|-----------|
| [**devstrap**](https://github.com/alecerf/devstrap) | Bootstrap and update development tools | `binary` | darwin, linux |
| [**go**](https://go.dev) | The Go programming language | `directory` | darwin, linux |
| [**golangci-lint**](https://github.com/golangci/golangci-lint) | Fast Go linters runner | `binary` | darwin, linux |
| [**kubectl**](https://kubernetes.io/docs/reference/kubectl/) | Kubernetes command-line tool | `direct` | darwin, linux |
| [**node**](https://nodejs.org) | Node.js JavaScript runtime | `directory` | darwin, linux |
| [**python**](https://www.python.org) | Python programming language | `directory` | darwin, linux |

## Details

### [devstrap](https://github.com/alecerf/devstrap)

Bootstrap and update development tools

- **Homepage:** https://github.com/alecerf/devstrap
- **Source:** `github_release` (`alecerf/devstrap`)
- **Install mode:** `binary` → `bin/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `bin/devstrap version`

### [go](https://go.dev)

The Go programming language

- **Homepage:** https://go.dev
- **Source:** `json_api`
- **Install mode:** `directory` → `go/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `go/bin/go version`

### [golangci-lint](https://github.com/golangci/golangci-lint)

Fast Go linters runner

- **Homepage:** https://github.com/golangci/golangci-lint
- **Source:** `github_release` (`golangci/golangci-lint`)
- **Install mode:** `binary` → `bin/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `bin/golangci-lint version`

### [kubectl](https://kubernetes.io/docs/reference/kubectl/)

Kubernetes command-line tool

- **Homepage:** https://kubernetes.io/docs/reference/kubectl/
- **Source:** `github_release` (`kubernetes/kubernetes`)
- **Install mode:** `direct` → `/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `bin/kubectl version --client`

### [node](https://nodejs.org)

Node.js JavaScript runtime

- **Homepage:** https://nodejs.org
- **Source:** `json_api`
- **Install mode:** `directory` → `node/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `node/bin/node -v`

### [python](https://www.python.org)

Python programming language

- **Homepage:** https://www.python.org
- **Source:** `github_release` (`astral-sh/python-build-standalone`)
- **Install mode:** `directory` → `python/`
- **Platforms:** darwin, linux (amd64, arm64)
- **Detect:** `python/bin/python3 --version`

