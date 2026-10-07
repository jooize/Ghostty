# Ghostty, patched build

This branch builds [Ghostty](https://github.com/ghostty-org/ghostty)'s macOS app from a
signed upstream release tag, with a small set of patches, in GitHub Actions. It is not the
Ghostty project and not an official build.

The branch holds only the build inputs. Ghostty's own code is fetched from upstream at
build time; `main` stays an untouched copy of upstream.

| Path | Purpose |
| :-- | :-- |
| `UPSTREAM` | The pinned release: tag, commit, signer fingerprint, build number, Xcode version |
| `keys/<fingerprint>.asc` | The public key the tag must be signed with |
| `patches/series` | The patches to apply, in order, one file name per line |
| `patches/*.patch` | The patches |
| `.github/workflows/build.yml` | The build |

## What the build checks

1. The tag's PGP signature verifies against the pinned key, and the signer's fingerprint
   matches `UPSTREAM_SIGNER`.
2. The tag points at `UPSTREAM_COMMIT`.
3. Xcode's build number matches `XCODE_BUILD`.

Any mismatch fails the build.

## Build

The app is built once per run. Reproducibility is not checked yet: the last comparison
of two builds (October 2026) found 4 of 542 files differing, and a second build
comes back when that work starts. The archive is still written without build times.
The bundle is signed ad hoc (no signing key anywhere) with the hardened runtime.
`no-sparkle.patch` removes the updater, so the app links only Apple's libraries and
library validation stays on; each run checks that and starts the signed app once.

## Releases

A release tag is `v<upstream version>+jooize.<n>`, for example `v1.3.1+jooize.3`: the
upstream version, then this branch's build of it, counted from 1 for each upstream version.
The suffix names who built the release, the way a distribution's does, so it stays true
whatever the patches are for. Releases before `v1.3.1+jooize.3` were tagged `+hardening.<n>`,
from this branch's earlier name; the count continues across the rename. The tag must be
annotated, since its message becomes the release notes, and must name the version pinned
in `UPSTREAM`, or the build refuses it. Pushing the tag runs the build, attests the
archive and its manifest with GitHub's build provenance, and publishes both
with `SHA256SUMS`.

The app's bundle identifier is `bar.esko.Ghostty` (`bundle-id.patch`), so it never
passes for the official build in Launch Services or privacy permissions. The archive is
signed ad hoc and not notarized; it is meant to be installed by a package manager such as
Nix, which does not mark downloads for Gatekeeper. Check a download against the workflow,
the tag and the commit the tag names:

    gh attestation verify Ghostty.app.zip -R jooize/ghostty \
      --signer-workflow jooize/ghostty/.github/workflows/build.yml \
      --source-ref refs/tags/<tag> --source-digest <commit> \
      --deny-self-hosted-runners

The same command checks `manifest.txt` for releases after `v1.3.1+hardening.1`.
