# Admin-private editable source

Editable canonical content is private application data owned by the Admin, not a
public repository tree. Its working model contains one document per hero, the shared
preparation and skin-tag documents, and the validation state needed to produce schema
3 runtime content.

The Admin can initialize or recover that private database from a trusted signed
`Document.mlbytes`:

1. Verify the bundle's Ed25519 signature, format, schema, version, bounds, hashes, and
   runtime relationships.
2. Recover the exact `heroes.json`, `preparations.json`, and `skin-tags.json` entries.
3. Reconstruct the private canonical editing view, including deterministic per-hero
   ownership.
4. Validate edits and project the private view back to exactly those three runtime
   entries.
5. Build an unsigned `Document.mlbytes` candidate for the protected signing workflow.

Of the Admin's editable outputs, only the unsigned candidate is pushed as a public
file; no JSON is published standalone. The protected workflow publishes the signed
root and immutable version bundles. Editor-only review state, migration reports, and
other provenance stay private, while the embedded runtime JSON remains recoverable
from a trusted signed bundle.
