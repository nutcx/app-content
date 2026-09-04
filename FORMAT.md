# `.mlbytes` container format

Status: draft, version 1.

## Goals

- One atomic package for the complete content dataset.
- Independent container, content-schema, and dataset versions.
- Deterministic builds and safe bounded extraction.
- Publisher authenticity and payload integrity.
- No client-side Git credential and no embedded encryption secret.

## Container

An `.mlbytes` file is a standard ZIP archive. Version 1 contains these entries:

```text
META-INF/mlbytes.json
META-INF/mlbytes.sig
content/heroes/<hero-id>.json
content/skin-tags.json
content/battle-effects.json
content/preparations.json
```

`META-INF/mlbytes.json` is UTF-8 JSON and is stored without compression. It is the
versioned package header and indexes every runtime content entry. `META-INF/mlbytes.sig`
is a raw 64-byte Ed25519 signature. Runtime JSON entries use DEFLATE compression.

Migration reports, editor review files, Git metadata, and signing secrets are not part
of the runtime package.

## Header

The header contains:

```json
{
  "format": "com.nutcx.mlbytes",
  "formatVersion": 1,
  "schemaVersion": 1,
  "dataset": "mlbb-content",
  "contentVersion": 1,
  "sourceRevision": "40-character-git-commit",
  "publishedAt": "2026-09-05T00:00:00Z",
  "minAppVersionCode": 4,
  "keyId": "content-2026-01",
  "entryCount": 0,
  "totalUncompressedBytes": 0,
  "entries": []
}
```

- `formatVersion` changes when the container or signature rules change.
- `schemaVersion` changes when the JSON data model changes.
- `contentVersion` is a strictly increasing positive integer used for update and
  rollback protection.
- `sourceRevision` identifies the source commit used to build the package.
- `entries` declares each runtime path, uncompressed size, and SHA-256 digest.

The signature input is the ASCII domain separator `MLBYTES-SIGNATURE-V1`, followed by
a zero byte, followed by the exact bytes of `META-INF/mlbytes.json`. Version 1 uses
Ed25519. Clients select a pinned public key using `keyId`; private signing keys must
never be committed or embedded in an app.

## Validation

A client must reject a package when any of these checks fail:

- unsupported format or schema version;
- incompatible minimum app version;
- invalid signature or unknown key ID;
- content-version rollback;
- duplicate, undeclared, absolute, parent-relative, or case-colliding ZIP paths;
- encrypted entries, symbolic links, or unsafe extraction paths;
- entry-count, per-entry-size, or total-uncompressed-size limits;
- declared size or SHA-256 mismatch;
- content schema or relationship validation after extraction.

The client downloads to a temporary file, validates and extracts into staging, then
atomically activates the completed snapshot. A failed update leaves the previous valid
snapshot untouched. Fresh installs use a bundled package processed by the same reader.

## Distribution

Editable source data remains in Git. A release job validates that source, creates a
deterministic `.mlbytes` package, signs its header, and publishes the package as an
immutable release asset. A small signed `latest.json` pointer can expose the newest
content version, URL, size, SHA-256, schema version, and minimum app version over HTTPS.

The format intentionally does not encrypt public content. A decryption key shipped in
an Android app is extractable and would not establish publisher authenticity.
