---
title: Fractional Hub
tags: [moc, hub]
type: moc
---

# Fractional Hub

Map of content for **fractional / advisory engagements** → companies where you hold a standing operating or advisory seat. Distinct from [[projects hub]]: a fractional role is an ongoing relationship at a company, not a time-bound project.

## Recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM "05 fractional"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything here

```dataview
LIST
FROM "05 fractional"
WHERE file.name != this.file.name
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[fractional hub]] · [[me hub]] · [[knowledge hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
