# Nut Cracker App Content

Public binary content and static WebP media consumed by Nut Cracker.

The client reads three independently signed root artifacts:

```text
Document.mlbytes       # versioned application content
ClientPolicy.mlbytes   # notices, availability, and update policy
ClientConfig.mlbytes   # operational restore/CDN configuration
```

The `.mlbytes` format is a compressed, indexed bundle with explicit format, schema,
and content versions. It is inspired by MLBB's indexed `Document.unity3d` pack shape,
but adds per-entry compression, SHA-256 integrity metadata, bounded parsing, and an
Ed25519 publisher signature. It is not encrypted. Schema 3 is the only supported
content contract; older layouts are rejected rather than translated at runtime.

`Document.mlbytes` contains the application's three UTF-8 JSON datasets as logical
entries. Each operational artifact contains one internal JSON entry. Those payloads
are not published as standalone files, and no bundle contains a nested ZIP or
extracted repository tree.

## Public layout

```text
Document.mlbytes
ClientPolicy.mlbytes
ClientConfig.mlbytes
candidates/
  <release>.<revision>/
    Document.mlbytes
  client-policy/
    <revision>-<policyId>/
      ClientPolicy.mlbytes
  client-config/
    <revision>-<configId>/
      ClientConfig.mlbytes
versions/
  <release>.<revision>/
    assets/
      Document.mlbytes
policies/
  <revision>-<policyId>/
    ClientPolicy.mlbytes
configs/
  <revision>-<configId>/
    ClientConfig.mlbytes
media/
  skins/
    <costumeId>/
      portrait.webp
      head.webp
      landscape.webp
```

- The three root bundles are the signed active client endpoints.
- A candidate is an unsigned, validated signing input. It is never a client endpoint.
- `versions/`, `policies/`, and `configs/` hold immutable signed release records.
- Existing public WebPs remain directly addressable static assets.

Publishing uses no standalone version pointer, release manifest, media index, or
detached signature. A protected release job validates a candidate, signs it,
publishes the matching immutable record, then updates its root bundle with the same
signed bytes.

Editable canonical JSON belongs to the Admin's private storage. The Admin can recover
its runtime documents from a trusted signed bundle, but neither editable source nor
review/provenance records are public release artifacts.

See [FORMAT.md](FORMAT.md) for the complete container specification,
[PAYLOADS.md](PAYLOADS.md) for the three embedded JSON schemas,
[SOURCE.md](SOURCE.md) for the private editing boundary, and [MEDIA.md](MEDIA.md) for
the public static-image policy.
