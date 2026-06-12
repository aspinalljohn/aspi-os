---
title: Resources Hub
tags: [moc, hub]
type: moc
---

# Resources Hub

Map of content for **active reference** kept at the ready → specs, configs, templates, cheat sheets you need on-demand for current work.

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "09 resources"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "09 resources"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[me hub]] · [[reading hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
