---
name: microovn-dev-loop
description: "The everyday MicroOVN developer loop — build, unit test, vet, and lint the Go code, and build the snap/run the BATS functional tests. Use when working on the microovn codebase (canonical/microovn), asked to build or test MicroOVN, hitting the go-dqlite cgo build error, or onboarding to the repo. Covers the libdqlite prerequisite, the exact cert/lint commands, and the snap/BATS invocation."
argument-hint: "what to do (build / test / vet / lint / snap / bats), or the error you hit"
---

# MicroOVN Dev Loop

MicroOVN is snap-deployed OVN with microcluster clustering. The Go module is at
`microovn/` (repo root holds the snap + tests), and it is a **cgo** project:
it links `canonical/go-dqlite` and `mattn/go-sqlite3`, so the Go toolchain
alone is not enough — you need a C compiler and a recent libdqlite.

## One-time setup (dqlite)

Ubuntu's `libdqlite0` (1.16) is too old for go-dqlite v3, so build dqlite from
source. This mirrors how the snap builds it (its `dqlite` part builds
`canonical/dqlite` default branch with `--enable-build-raft`).

```bash
sudo apt-get install -y make gcc pkg-config autoconf automake libtool \
  libuv1-dev libsqlite3-dev liblz4-dev git
git clone --depth 1 https://github.com/canonical/dqlite /tmp/dqlite
cd /tmp/dqlite
autoreconf -i
CFLAGS="-Wno-error=discarded-qualifiers" ./configure --enable-build-raft
make -j"$(nproc)"
sudo make install
sudo ldconfig
```

Without this, `go build` fails with
`build constraints exclude all Go files in .../go-dqlite/v3/.../internal/bindings`
and `undefined: sqlite3.ErrConstraintUnique`.

> **Workshop shortcut:** the repo ships a `workshop.yaml`. `workshop run -- deps`
> performs the setup above inside a Workshop sandbox, then
> `workshop run -- build` / `test-unit` / `vet` / `fmt-check` / `lint-go` /
> `lint-sh` run the loop. See `workshop.yaml` in the repo root.

## Workflow

Run from `microovn/` (the Go module directory):

```bash
cd microovn

go build ./...          # compile everything
go test ./...           # full unit suite
go vet ./...            # static checks
gofmt -l .              # must print nothing
golangci-lint run       # Go linter (0 issues expected)
```

Shellcheck lint of the test suite runs from the **repo root** (needs `make`
and `shellcheck`):

```bash
cd ..
make check-lint
```

If any check fails, fix and rerun only that check.

## Snap + functional tests

The `make` target builds the snap (needs `snapcraft` + LXD):

```bash
make                     # produces microovn.snap
```

BATS functional tests deploy OVN into LXD:

```bash
MICROOVN_TESTS_USE_SNAP=yes make tests/<name>.bats
```

Gotchas:

- Build/test from a **real clone**, not a `git worktree` — snapcraft's
  `git describe` fails in a worktree.
- To iterate quickly, stub the docs part in `snap/snapcraft.yaml`:
  `override-build: echo stubbed`.

## Key rules

- `go build` / `go test` fail without libdqlite installed — do the setup first.
- Commit subjects end with a full stop.
- See `docs/contributors/code.rst` for the snap build/install flow (the Go
  dev loop above is what CI exercises).