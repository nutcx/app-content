# Skin badge artwork

These 110 transparent WebP images are app display assets for the named Badge selector and skin cards. They do not change in-game badges or game resource tables.

Each filename includes a readable label and a hash suffix. The same lossless image bytes are bundled in the app catalog library and Content Admin for first-launch/offline display.

Source atlas SHA-256: `90a5df38e8f3537ddc4afe651a9b1d76a3e83c380d0951009a3d544abf333576`.
Transparency follows the atlas UI/PngMask material: alpha is read from the mask red channel. Cropped decoded RGBA pixels were verified after WebP encoding. Four future MSC-year placeholder graphics were excluded.

Publish these media files before publishing a catalog that references their raw GitHub URLs. Staging this directory locally does not make those URLs live. Replacing custom catalog artwork remains supported independently of these bundled defaults.
