# vsql-extension-actions

Reusable GitHub Actions composite actions for building and testing [VillageSQL](https://villagesql.com) extensions.

## Actions

| Action | Description |
|---|---|
| [`cpp`](cpp/action.yml) | Build and test a C++ extension |
| [`rust`](rust/action.yml) | Build and test a Rust extension |

By default, actions download build artifacts (SDK, dev server) from the latest successful run of `extension-compat.yml` in `villagesql/villagesql-server`. Pass `villagesql-version` to pin to a specific release tag instead.

Release tag naming changed at 0.0.6, from `0.0.5` to `release/0.0.6`. Pass the bare version either way: the actions look up `<version>` first and fall back to `release/<version>`.

> **Platform:** these actions currently only support `linux-x86_64` runners (e.g. `runs-on: ubuntu-latest`). Multi-platform support is tracked as a follow-up.

## C++ extensions

### Repository layout

Your extension repo must have a `CMakeLists.txt` at the root that accepts `-DVillageSQL_SDK_DIR` and produces a `.veb` file named `<extension-name>.veb` in the build directory. MTR tests go in a `mysql-test/` directory at the root.

```
my-extension/
├── CMakeLists.txt
├── src/
│   └── ...
└── mysql-test/
    ├── my_test.test
    └── my_test.result
```

### Usage

Create `.github/workflows/ci.yml` in your extension repo:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: read
    steps:
      - uses: actions/checkout@v4
      - uses: villagesql/extension-actions/cpp@main
        with:
          extension-name: my_extension
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `extension-name` | yes | — | Name of the extension. Used as the MTR suite name and the output artifact name (`<extension-name>.veb`). |
| `github-token` | yes | — | GitHub token for downloading artifacts from `villagesql/villagesql-server`. Pass `${{ secrets.GITHUB_TOKEN }}`. |
| `villagesql-version` | no | `nightly` | Dev server / SDK version to test against. Use a release tag (e.g. `0.0.4`) or `nightly` for the latest build of `extension-compat.yml`. |
| `villagesql-product` | no | `mysql-8.4` | Server product to test against when `villagesql-version` is a release. A release carries one dev server per product, so this selects which one. Values: `mysql-8.4`, `mysql-9.7`, `percona-8.4`. |

### Artifacts

On success, uploads `<extension-name>.veb` as a build artifact. On test failure, MTR logs are uploaded as `mtr-results`.

## Rust extensions

### Repository layout

Your extension repo must be a Cargo workspace or crate that `cargo vsql test` can build and test. The packaged `.veb` is created at `dist/<extension-name>.veb` relative to the extension directory.

```
my-extension/
├── Cargo.toml
├── src/
│   └── lib.rs
└── mysql-test/
    ├── my_test.test
    └── my_test.result
```

### Usage

Create `.github/workflows/ci.yml` in your extension repo:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: read
    steps:
      - uses: actions/checkout@v4
      - uses: villagesql/extension-actions/rust@main
        with:
          extension-name: my_extension
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `extension-name` | yes | — | Name of the extension. Used for the output artifact name (`<extension-name>.veb`). |
| `extension-dir` | no | `.` | Relative path to the extension directory. |
| `github-token` | yes | — | GitHub token for downloading artifacts from `villagesql/villagesql-server`. Pass `${{ secrets.GITHUB_TOKEN }}`. |
| `villagesql-version` | no | `nightly` | Dev server version to test against. Use a release tag (e.g. `0.0.4`) or `nightly` for the latest build of `extension-compat.yml`. |
| `villagesql-product` | no | `mysql-8.4` | Server product to test against when `villagesql-version` is a release. A release carries one dev server per product, so this selects which one. Values: `mysql-8.4`, `mysql-9.7`, `percona-8.4`. |

### Monorepo usage

For repos with multiple extensions, use a matrix:

```yaml
jobs:
  ci:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: read
    strategy:
      fail-fast: false
      matrix:
        include:
          - name: rot13
            dir: extensions/rot13
          - name: rational
            dir: extensions/rational
    steps:
      - uses: actions/checkout@v4
      - uses: villagesql/extension-actions/rust@main
        with:
          extension-name: ${{ matrix.name }}
          extension-dir: ${{ matrix.dir }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Artifacts

On success, uploads `<extension-name>.veb` as a build artifact.

## Pinning to a release

Replace `@main` with a tag to pin to a specific version:

```yaml
- uses: villagesql/extension-actions/cpp@v1
- uses: villagesql/extension-actions/rust@v1
```
