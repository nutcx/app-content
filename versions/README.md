# Published content versions

Each child directory is an immutable `<release>.<revision>` content version.

```text
<release>.<revision>/
  manifest.json
  manifest.json.sig
  assets/
    Document.mlbytes
```

The two version components are unsigned integers and are compared numerically as a
tuple. Never overwrite a published directory; publish a new revision instead.
