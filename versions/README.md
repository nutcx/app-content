# Published content versions

Each child directory is an immutable `<release>.<revision>` content version.

```text
<release>.<revision>/
  assets/
    Document.mlbytes
```

The versioned `Document.mlbytes` is signed and self-describing: its header carries the
format version, schema version, content version, and minimum compatible app version,
while its directory carries entry sizes and SHA-256 hashes. No separate version
metadata or detached signature is published.

The two version components are unsigned integers and are compared numerically as a
tuple. Never overwrite a published directory; publish a new revision instead. The
active root `Document.mlbytes` is byte-identical to the selected immutable version.
