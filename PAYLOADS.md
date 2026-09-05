# MLBytes payload schema version 3

`Document.mlbytes` contains exactly three UTF-8 JSON documents. Each document is an
object, not a top-level array. Unknown properties are rejected. Generated documents
sort object keys lexicographically, use deterministic indentation, and end with one
line-feed byte.

There is deliberately no older-schema compatibility reader. IDs are nonnegative signed
32-bit integers, skin categories are signed 32-bit integers,
timestamps are nonnegative signed 64-bit integers, and archive values are either
empty strings or absolute HTTPS URLs whose path ends in `.zip`.

## `heroes.json`

Root property: `heroes`. Heroes are ordered by numeric `heroId` and contain:

- `heroId`: integer
- `skins`: array ordered by numeric `category`, then `skinId`

Each skin contains `skinId`, `category`, `name`, `type`, `portrait`, `landscape`,
numeric `timestamp`, and required `source`. `source` is either `null` or an object
with string `backupArchive` and an `upgrades` array. An upgrade contains
`targetSkinId`, `targetCategory`, and string `archive`; its target must exist in the
same hero catalog.

This is the only skin representation. There are no parallel `skinSources`, flattened
upgrade tables, or legacy skin payloads. An empty archive marks an unavailable file;
it does not erase the source or its other routes.

## `preparations.json`

Root property: `preparations`. Each preparation contains integer `preparationId` and
string `name`, `image`, and `archive`, plus `items`. Each item contains only string
`name`, `image`, and `archive`. Preparation IDs are globally unique; item array order
defines the replacement index.

## `skin-tags.json`

Root property: `tags`. Each item contains string `name` and string `image`.

## Editable-source projection

The two shared documents are validated from `source/`. Sorted
`source/heroes/<heroId>.json` documents are concatenated as the `heroes` array without
flattening their owned source graph. Migration reports, review records, Git metadata,
and signing secrets are never runtime entries.
