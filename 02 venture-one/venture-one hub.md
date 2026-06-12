---
title: Venture One Hub
tags: [moc, hub]
type: moc
---

# Venture One Hub

Map of content for your **first daily venture** → its projects, clients, frameworks, SOPs, and deliverables. Rename this hub and folder to your real venture.

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "02 venture-one"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "02 venture-one"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[me hub]] · [[reading hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
