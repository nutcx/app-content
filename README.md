# Nut Cracker App Content

Versioned content packages consumed by Nut Cracker.

This repository is being prepared for the new `.mlbytes` format. The new format is a
single compressed binary bundle with explicit format, schema, and content versions.
It is inspired by MLBB's indexed `Document.unity3d` pack layout, but it adds per-entry
compression and integrity metadata. It is not encrypted and does not support the
legacy six-file encrypted layout.

The indexed payloads are the application's UTF-8 JSON datasets themselves. There is
no nested ZIP archive or extracted repository tree inside the bundle.

See [FORMAT.md](FORMAT.md) for the draft container specification.
