---
title: Me Hub
tags: [moc, hub]
type: moc
---

# Me Hub

Map of content for **personal** material → brand, health, journal, reflection, drafts. Your own thinking and life.

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "07 me"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "07 me"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[fractional hub]] · [[me hub]] · [[knowledge hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
