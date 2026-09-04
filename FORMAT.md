# `.mlbytes` container format

Status: draft, version 1.

## Goals

- One atomic package for the complete content dataset.
- Independent container, content-schema, and dataset versions.
- Deterministic builds and safe bounded extraction.
- Publisher authenticity and payload integrity.
- No client-side Git credential and no embedded encryption secret.

## Container

An `.mlbytes` file is a purpose-built indexed binary bundle. It follows the useful
shape of MLBB's `Document.unity3d` aggregate container: a directory at the front and
contiguous entry payloads after it. It is not ZIP or UnityFS.

All integers are unsigned little-endian. Version 1 starts with this fixed 80-byte
header:

| Offset | Size | Field |
| ---: | ---: | --- |
| `0x00` | 8 | Magic: ASCII `MLBYTES` followed by `0x00` |
| `0x08` | 2 | Format major version: `1` |
| `0x0A` | 2 | Format minor version: `0` |
| `0x0C` | 4 | Flags; bit 0 means an Ed25519 signature is present |
| `0x10` | 4 | Header size: `80` |
| `0x14` | 4 | Content schema version |
| `0x18` | 4 | Minimum compatible Android app version code |
| `0x1C` | 4 | Reserved; must be zero |
| `0x20` | 8 | Monotonically increasing content version |
| `0x28` | 4 | Directory entry count |
| `0x2C` | 4 | Directory size in bytes |
| `0x30` | 8 | Absolute payload-region offset |
| `0x38` | 8 | Stored payload-region size |
| `0x40` | 8 | Total uncompressed entry size |
| `0x48` | 8 | Absolute signature-block offset |

The header is followed immediately by the directory. Each variable-size directory
record contains:

| Size | Field |
| ---: | --- |
| 2 | UTF-8 path length |
| 1 | Compression codec: `0` stored, `1` zlib/DEFLATE |
| 1 | Entry flags; must be zero in version 1 |
| 4 | Reserved; must be zero |
| 8 | Offset relative to the payload region |
| 8 | Stored/compressed size |
| 8 | Original uncompressed size |
| 32 | SHA-256 of the original uncompressed bytes |
| variable | UTF-8 path bytes; no terminator |

Directory records are sorted by their UTF-8 path bytes. Payloads appear in the same
order, are contiguous, and contain no padding or gaps. Each payload is compressed
independently so the client can read one entry without expanding the whole bundle.
The writer uses stored mode when DEFLATE would not reduce an entry's size.

Required content paths include:

```text
content/manifest.json
content/heroes/<hero-id>.json
content/skin-tags.json
content/battle-effects.json
content/preparations.json
```

Migration reports, editor review files, Git metadata, and signing secrets are not part
of the runtime bundle.

## Signature block

Release bundles are signed but not encrypted. The signature block starts at the offset
declared in the fixed header:

| Size | Field |
| ---: | --- |
| 8 | Magic: ASCII `MLBSIG` followed by two zero bytes |
| 2 | Signature-block version: `1` |
| 2 | Algorithm: `1` for Ed25519 |
| 2 | UTF-8 key ID length |
| 2 | Signature length: `64` |
| variable | UTF-8 key ID |
| 64 | Raw Ed25519 signature |

The signed message is the ASCII domain separator `MLBYTES-SIGNATURE-V1`, a zero byte,
all file bytes before the signature block, and the signature-block bytes through the
key ID. Clients select a pinned public key using the signed key ID. Private signing
keys must never be committed or embedded in an app.

- The format version changes when the binary container or signature rules change.
- The schema version changes when the JSON data model changes.
- The content version changes for every published dataset and provides rollback
  protection.

## Validation

A client must reject a package when any of these checks fail:

- unsupported format or schema version;
- incompatible minimum app version;
- invalid signature or unknown key ID;
- content-version rollback;
- duplicate, undeclared, absolute, parent-relative, or case-colliding entry paths;
- non-contiguous entries or inconsistent directory, payload, and signature offsets;
- unsafe paths or unsupported compression codecs;
- entry-count, per-entry-size, or total-uncompressed-size limits;
- declared size or SHA-256 mismatch;
- content schema or relationship validation after extraction.

The client downloads to a temporary file, validates and extracts into staging, then
atomically activates the completed snapshot. A failed update leaves the previous valid
snapshot untouched. Fresh installs use a bundled package processed by the same reader.

## Distribution

Editable source data remains in Git. A release job validates that source, creates a
deterministic `.mlbytes` bundle, signs it, and publishes the bundle as an
immutable release asset. A small signed `latest.json` pointer can expose the newest
content version, URL, size, SHA-256, schema version, and minimum app version over HTTPS.

The format intentionally does not encrypt public content. A decryption key shipped in
an Android app is extractable and would not establish publisher authenticity.
