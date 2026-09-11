# Verified Skin Media

`media/skins/<costumeId>/` contains the public static WebP images used by signed
content bundles.

Each costume directory contains every verified static asset available for that
costume. A missing media kind is left absent rather than filled with a guessed
substitute. The possible files are:

- `portrait.webp`: the mapped 256x512 card texture, with only its reviewed
  static background composition applied;
- `head.webp`: either the exact standalone 128x128 base head or the mapped
  100x100 skin-head sprite, using the red channel of the matching mask atlas as
  alpha; and
- `landscape.webp`: the exact mapped share-background sprite rectangle.

Heads use lossless WebP. Portraits and landscapes use WebP quality 95 with
their original dimensions and exact alpha preserved. The export process
verifies every input bundle's byte length and SHA-256 digest, resolves textures
through their exact Unity resource hierarchy or atlas metadata, and decodes
every written file again to confirm its dimensions and alpha. Source digests,
decoded pixel digests, encoding settings, dimensions, object-level selection
evidence, and bundle provenance remain in private audit records.

There is no public media index or JSON metadata. Signed payloads reference the WebPs
they use directly. Published WebP paths are immutable: correcting image bytes requires
a new public path and a new signed content version.
