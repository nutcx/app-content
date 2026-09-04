# Editable content source

The `source/` directory is the validated canonical input for release generation. It
contains one JSON document per hero plus the shared battle-effect, preparation, and
skin-tag documents. Its manifest records schema version 2 and the SHA-256 digest of
every declared source file.

Release tooling validates this tree first, projects it into the four runtime documents
defined by [PAYLOADS.md](PAYLOADS.md), then packs those documents into the immutable
`Document.mlbytes` artifact. Clients never download or extract the editable source
tree.

`migration-report.json` and `migration-review.json` remain source audit records. They
are validated during release preparation but are never included in the runtime
bundle.
