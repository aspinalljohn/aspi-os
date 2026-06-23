---
title: Vault Operations Hub
tags: [moc, hub]
type: moc
---

# Vault Operations Hub

How this vault operates on itself → the primitives ([[gobble]], [[skillify]]), note standards, and every running automation. See [[automation-patterns]].

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "09 vault operations"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "09 vault operations"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[fractional hub]] · [[me hub]] · [[knowledge hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
