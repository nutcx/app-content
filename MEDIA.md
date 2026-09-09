# Verified Skin Media

`media/skins/<costumeId>/` contains static images exported from the exact MLBB
Unity bundles recorded in `media/skins/manifest.json`.

Each costume directory contains:

- `portrait.webp`: the mapped 256x512 card texture, with only its reviewed
  static background composition applied;
- `head.webp`: the mapped 100x100 head sprite, using the red channel of the
  matching mask atlas as alpha; and
- `landscape.webp`: the exact mapped share-background sprite rectangle.

The files are lossless WebP. The export process verifies every input bundle's
byte length and SHA-256 digest, selects textures by mapped Unity path ID, and
decodes every written file again to confirm a pixel-exact round trip. The
manifest preserves the source mapping, dimensions, pixel digests, and bundle
provenance required to reproduce or audit an export.

Published content should use commit-pinned raw GitHub URLs for these files so a
released content snapshot cannot change when the repository's default branch
moves.
