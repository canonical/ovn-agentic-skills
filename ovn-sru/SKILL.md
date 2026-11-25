---
name: ovn-sru
description: "Package an OVN change and ship it to Ubuntu users: turn an upstream fix into a quilt patch in the Debian salsa packaging, build/sign/upload to a PPA, and file the Ubuntu SRU / Launchpad merge request. Use when asked to backport or release an OVN fix, SRU a Launchpad bug against ovn (Ubuntu), or land a patch in the archive. Covers the salsa branch layout, quilt conventions, debuild/dput, and the git-ubuntu MP target."
argument-hint: "the Launchpad bug number or upstream commit to ship"
---

# OVN packaging / SRU

OVN reaches Ubuntu through the Debian packaging (salsa), then a sync or SRU
into the Ubuntu archive. A single OVN fix therefore travels: upstream commit →
quilt patch in the Debian tree → upload/SRU → Ubuntu series.

## Workflow

### 1. Identify the source tree

Three source trees are involved:

- **salsa Debian packaging** — `debian/sid` (current, → Ubuntu resolute),
  `debian/experimental` (the newer upstream release), `debian/hibiscus` (dev).
  Patches are quilt, kept **unapplied** (format 3.0).
- **Debian source tree** — `ovn-<version>/` with `debian/patches/` + `series`.
- **Ubuntu git-ubuntu fork** — your `+source/ovn` fork with `sru/<series>-<version>`
  branches targeting `ubuntu/<series>-devel`.

### 2. Add the fix as a quilt patch

```bash
export QUILT_PATCHES=debian/patches
quilt new <name>.patch
quilt add <files-to-change>
# edit the files to match the upstream commit
quilt refresh
quilt pop -a && quilt push -a   # verify it applies clean
```

Write patches git-style (`diff --git a/ b/`) with a DEP-3 header
(`Description:` / `Author:` / `Forwarded:` / `Origin: upstream` /
`Bug-Ubuntu:`). One commit per patch, plus a separate `Update Changelog.`
commit (exact subject, capital C + full stop). Bump `debian/changelog` with
`dch` (set `DEBEMAIL`/`DEBFULLNAME` to avoid the `@FQDN` mangle).

### 3. Build, sign, upload (PPA)

From the Debian source tree:

```bash
debuild -S -sa -us -uc -d          # source-only build
debsign -k 8DF767B54A9F173EB0B0BDA75584CBED77092D65 ovn_<v>_source.changes
dput ppa:crypticcoder/sru-ppa ovn_<v>_source.changes
```

If Launchpad rejects a same-version re-upload, bump the changelog revision.

### 4. SRU / archive landing

For the noble (24.04) SRU, push the `sru/noble-<version>` branch to the
git-ubuntu fork and register a merge request against
`~git-ubuntu-import/ubuntu/+source/ovn` → `ubuntu/noble-devel`. After review
and upload to `noble-proposed`, flip the Launchpad bug tasks to `Fix Released`.

For newer series that track Debian, a sync (or the salsa branch itself) may be
enough — coordinate with the SRU team for the exact path.

## Key rules

- Patches stay **unapplied** in the trees (source stays pristine orig).
- One patch = one commit; changelog is its own commit, last.
- `Forwarded: no` for fixes still pending upstream; `Origin: upstream` + the
  upstream SHA when backporting a merged fix.
- Sign with `8DF767B5…` ("MJ Ponsonby <mj.ponsonby@canonical.com>").
- `debian/experimental` tracks the upstream **release tag**, not the dev branch
  tip — regenerate patch context against the tag.

## Supporting files

- [references/branch-layout.md](references/branch-layout.md) — the salsa/Ubuntu
  series↔branch↔version map and the ovn-vif two-component layout.