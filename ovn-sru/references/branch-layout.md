# OVN packaging — branch / series layout

## Debian salsa (`openstack-team` third-party `ovn`)

- `debian/sid` — current release line (e.g. OVN 26.03.x), syncs to Ubuntu
  **resolute** (26.04).
- `debian/experimental` — the *newer* upstream release (e.g. 26.09.0). Major
  bumps land here first, not in `sid`.
- `debian/hibiscus` — the dev branch (OpenStack release codename); CI fixes and
  new work happen here before reaching `sid`.

The `debian/<codename>` branches (caracal … gazpacho) are stale (all 21.06.0).

## Ubuntu source tree vs git-ubuntu

- Ubuntu ships OVN from the Debian source; the archive syncs from `debian/sid`.
- Older LTS (e.g. noble / 24.04, on OVN 24.03.x) is **EOL upstream** and is
  maintained via SRU on the git-ubuntu tree: fork
  `~<user>/ubuntu/+source/ovn`, branch `sru/<series>-<version>`, merge request
  against `~git-ubuntu-import/ubuntu/+source/ovn` → `ubuntu/<series>-devel`.

## Two-component tarballs

The source has two orig components: `ovn_<v>.orig.tar.xz` (main, tree root)
and `ovn_<v>.orig-ovn-vif.tar.xz` (unpacks into the `ovn-vif/` subdir). The
`ovn-vif` component can lag the main release version — keep them consistent in
`debian/watch` and the changelog.

## Patch application order

Quilt patches apply in `debian/patches/series` order; they must apply clean
against the **pristine** upstream tarball (unapplied source). `quilt push -a`
must succeed end-to-end before upload.