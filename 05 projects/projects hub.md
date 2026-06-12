---
title: Projects Hub
tags: [moc, hub]
type: moc
---

# Projects Hub

Map of content for **standalone, time-bound projects** that aren't one of the three ventures.

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "05 projects"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "05 projects"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[me hub]] · [[reading hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
