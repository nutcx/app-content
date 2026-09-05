# `.mlbytes` container format

Status: implemented, version 1.

## Goals

- One atomic package for the complete content dataset.
- Independent container, content-schema, and dataset versions.
- Deterministic builds and safe bounded parsing.
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
| `0x20` | 4 | Content release number, for example `1001` |
| `0x24` | 4 | Content revision number, for example `0` |
| `0x28` | 4 | Directory entry count |
| `0x2C` | 4 | Directory size in bytes |
| `0x30` | 8 | Absolute payload-region offset |
| `0x38` | 8 | Stored payload-region size |
| `0x40` | 8 | Total uncompressed entry size |
| `0x48` | 8 | Absolute signature-block offset |

The directory starts at `headerSize`. The payload-region offset must equal
`headerSize + directorySize`, and the signature-block offset must equal
`payloadOffset + storedPayloadSize` when the signature flag is set. When the flag is
unset for development fixtures, the signature offset is zero and the file ends at the
end of the payload region. Unknown flag bits are rejected.

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
Codec `1` is an RFC 1950 zlib stream containing DEFLATE data, with no preset
dictionary. The deterministic writer uses compression level 9 and stored mode when
compression would not strictly reduce an entry's size.

Each entry's uncompressed payload is the exact UTF-8 JSON document consumed by the
application. The bundle does not contain another archive or an extracted canonical
repository tree. Content schema 3 uses these three logical entries:

```text
heroes.json
preparations.json
skin-tags.json
```

Their stored blocks are concatenated in directory order. Reading an entry means seeking
to its block, inflating it when necessary, verifying its SHA-256, and parsing the
resulting JSON bytes directly. No filesystem extraction is required.

Migration reports, editor review files, Git metadata, and signing secrets are not
runtime entries.

Schema 3 content bundles contain exactly these three unique, case-sensitive entry
paths. Their JSON contracts are defined in [PAYLOADS.md](PAYLOADS.md).

## Version 1 limits

- Required runtime entries: exactly the three paths above; the binary parser rejects
  declared counts above 64 before allocating directory records.
- UTF-8 entry path: at most 1,024 bytes.
- Directory: at most 65,536 bytes.
- Stored or uncompressed size of one entry: at most 8 MiB.
- Sum of uncompressed entry sizes: at most 32 MiB.
- Complete bundle file: at most 40 MiB.

Each record size is `64 + pathLength`. Relative payload offsets must equal the sum of
all preceding stored sizes. Header totals must equal the corresponding directory
sums. The complete file ends exactly after the payload region for an unsigned fixture,
or exactly after the signature block for a signed production bundle; trailing bytes
are forbidden.

## Signature block

Release bundles are signed but not encrypted. Production bundles must set header flag
bit 0 and include the signature block at the declared offset. Flag-zero unsigned
bundles are permitted only as local development and test fixtures. The signature
block starts at the offset declared in the fixed header:

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

`latest.json.sig` and `manifest.json.sig` use the same complete `MLBSIG` block. Their
signed messages use `MLBYTES-LATEST-V1` and `MLBYTES-MANIFEST-V1`, respectively,
followed by one zero byte, the exact raw JSON bytes, and the signature-block bytes
through the key ID. Detached JSON is limited to 64 KiB. A detached signature file is
not a bare 64-byte Ed25519 value.

- The format version changes when the binary container or signature rules change.
- The schema version changes when the JSON data model changes.
- The content version is rendered as `<release>.<revision>`, for example `1001.0`.
  Both components are unsigned integers without leading zeroes; this is not a decimal
  or floating-point number.
- Versions are compared as numeric `(release, revision)` tuples, so `1001.9` is older
  than `1001.10`, which is older than `1002.0`.
- A normal dataset update increments the release and resets revision to zero. A
  correction increments the revision.

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
- JSON schema or relationship validation after decompression.

The client downloads to a temporary file, validates every JSON payload, then atomically
activates the completed bundle. A failed update leaves the previous valid bundle
untouched. Fresh installs use a bundled package processed by the same reader.

## Distribution

Editable source data remains in Git. A release job validates that source, creates a
deterministic `.mlbytes` bundle, signs it, and publishes it using the case-sensitive
filename `Document.mlbytes`:

```text
latest.json
latest.json.sig
versions/
  1001.0/
    manifest.json
    manifest.json.sig
    assets/
      Document.mlbytes
```

Published version directories are append-only and immutable. Changed bytes require a
new version: publish `1001.1` rather than overwriting `1001.0`. The version in the
directory name, version manifest, latest pointer, and bundle header must match.

The root `latest.json` points to the immutable version manifest using relative paths:

```json
{
  "pointerFormat": 1,
  "channel": "stable",
  "contentVersion": "1001.0",
  "release": 1001,
  "revision": 0,
  "manifest": {
    "path": "versions/1001.0/manifest.json",
    "size": 512,
    "sha256": "64-lowercase-hex-characters",
    "signaturePath": "versions/1001.0/manifest.json.sig"
  },
  "publishedAt": "2026-09-05T00:00:00Z"
}
```

The immutable version manifest identifies the bundle:

```json
{
  "manifestFormat": 1,
  "contentVersion": "1001.0",
  "release": 1001,
  "revision": 0,
  "formatVersion": "1.0",
  "schemaVersion": 3,
  "minimumAppVersionCode": 7,
  "asset": {
    "path": "versions/1001.0/assets/Document.mlbytes",
    "size": 123456,
    "sha256": "64-lowercase-hex-characters"
  }
}
```

Publish `Document.mlbytes`, the version manifest, and their signatures first. Update
`latest.json` last so clients never discover an incomplete version. Relative paths
allow the same layout to work from GitHub Raw now and a dedicated CDN later.

The client rejects a lower version than either its bundled baseline or the highest
successfully activated version. An equal version with the same bundle SHA-256 is a
no-op; an equal version with different bytes is an immutable-version collision.

The format intentionally does not encrypt public content. A decryption key shipped in
an Android app is extractable and would not establish publisher authenticity.
