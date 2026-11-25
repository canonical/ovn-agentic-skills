---
name: microovn-dev-loop
description: "The everyday MicroOVN developer loop — build the snap with `make`, then run the code checks (unit test, vet, fmt, lint) and the BATS functional tests. Use when working on the microovn codebase (canonical/microovn), asked to build or test MicroOVN, or onboarding to the repo. Covers the snap build, the go test/vet/lint checks and their libdqlite (cgo) prerequisite, and the BATS invocation."
argument-hint: "what to do (build / test / vet / lint / functional), or the error you hit"
---

# MicroOVN Dev Loop

MicroOVN is a **snap** — the build artifact is `microovn.snap`. Per
`docs/contributors/code.rst`, the build requirements are *Make and snapcraft*,
and you build with `make`. The Go module at `microovn/` is what gets compiled
*inside* that build; it is not the build itself.

## Build the snap

```bash
make          # snapcraft pack -o microovn.snap  (needs snapcraft + LXD)
```

`make` compiles the Go source and, as part of the snap build, builds its own
`dqlite` part — so you do **not** need to install libdqlite yourself to build
the snap.

## Code checks (what CI runs between snap builds)

These operate on the Go source directly and do need libdqlite (cgo) installed:

```bash
cd microovn

go test ./...          # full unit suite
go vet ./...           # static checks
gofmt -l .             # must print nothing
golangci-lint run      # Go linter (0 issues expected)
```

Shellcheck lint of the test suite runs from the repo root (needs `make` +
`shellcheck`):

```bash
cd ..
make check-lint
```

If a check fails, fix and rerun only that check.

## One-time setup for the Go checks (libdqlite)

Raw `go test`/`go vet`/`golangci-lint` link `go-dqlite` + `mattn/go-sqlite3`
via cgo. Ubuntu's `libdqlite0` (1.16) is too old for go-dqlite v3, so build it
from source (mirrors the snap's `dqlite` part — `canonical/dqlite`,
`--enable-build-raft`):

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

Without this, `go test`/`go vet` fail with
`build constraints exclude all Go files in .../go-dqlite/v3/.../internal/bindings`
and `undefined: sqlite3.ErrConstraintUnique`.

> **Workshop shortcut:** the repo ships a `workshop.yaml`.
> `workshop run -- deps` performs the setup above inside a Workshop sandbox,
> then `workshop run -- test-unit` / `vet` / `fmt-check` / `lint-go` / `lint-sh`
> run the checks. See `workshop.yaml` in the repo root.

## Functional tests

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

- **Build with `make`** — that is the deliverable (the snap).
- The go checks above need libdqlite installed; `make` does not.
- Commit subjects end with a full stop.
- See `docs/contributors/code.rst` for the snap build/install flow.