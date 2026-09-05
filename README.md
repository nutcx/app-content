# Nut Cracker App Content

Versioned content packages consumed by Nut Cracker.

This repository publishes the `.mlbytes` format used by Nut Cracker. The format is a
single compressed binary bundle with explicit format, schema, and content versions.
It is inspired by MLBB's indexed `Document.unity3d` pack layout, but it adds per-entry
compression and integrity metadata. It is not encrypted. Schema 3 is the only
supported content contract; older layouts are rejected instead of translated at
runtime.

The indexed payloads are the application's UTF-8 JSON datasets themselves. There is
no nested ZIP archive or extracted repository tree inside the bundle.

The published client artifact is always named `Document.mlbytes` and lives under an
immutable content-version path such as:

```text
versions/1001.0/assets/Document.mlbytes
```

The root `latest.json` points to the newest immutable version manifest. Editable,
validated inputs live under `source/`; published clients consume only the generated
bundle under `versions/`.

See [FORMAT.md](FORMAT.md) for the container specification, [PAYLOADS.md](PAYLOADS.md)
for the three JSON payload schemas, and [SOURCE.md](SOURCE.md) for the source-to-release
boundary.
