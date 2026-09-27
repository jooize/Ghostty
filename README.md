# Ghostty, hardened build

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

## Reproducibility

The app is built twice on separate runners and compared file by file. For now the
comparison is reported in the run summary and does not fail the run. The bundle is signed
ad hoc (no signing key anywhere), and Sparkle has no update key, so it cannot install an
update over this build.
