# Nut Cracker App Content

Public binary content and static WebP media consumed by Nut Cracker.

The client reads one signed root artifact:

```text
Document.mlbytes
```

The `.mlbytes` format is a compressed, indexed bundle with explicit format, schema,
and content versions. It is inspired by MLBB's indexed `Document.unity3d` pack shape,
but adds per-entry compression, SHA-256 integrity metadata, bounded parsing, and an
Ed25519 publisher signature. It is not encrypted. Schema 3 is the only supported
content contract; older layouts are rejected rather than translated at runtime.

The bundle contains the application's three UTF-8 JSON datasets as logical entries.
Those datasets are not published as standalone files, and the bundle contains no
nested ZIP or extracted repository tree.

## Public layout

```text
Document.mlbytes
candidates/
  <release>.<revision>/
    Document.mlbytes
versions/
  <release>.<revision>/
    assets/
      Document.mlbytes
media/
  skins/
    <costumeId>/
      portrait.webp
      head.webp
      landscape.webp
```

- The root bundle is the signed active client artifact.
- A candidate is an unsigned, validated signing input. It is never a client endpoint.
- A versioned bundle is the immutable signed record of one content version.
- Existing public WebPs remain directly addressable static assets.

Publishing uses no standalone version pointer, release manifest, media index, or
detached signature. The protected release job validates a candidate, signs it,
publishes the immutable version, then updates the root bundle with the same signed
bytes.

Editable canonical JSON belongs to the Admin's private storage. The Admin can recover
its runtime documents from a trusted signed bundle, but neither editable source nor
review/provenance records are public release artifacts.

See [FORMAT.md](FORMAT.md) for the complete container specification,
[PAYLOADS.md](PAYLOADS.md) for the three embedded JSON schemas,
[SOURCE.md](SOURCE.md) for the private editing boundary, and [MEDIA.md](MEDIA.md) for
the public static-image policy.
