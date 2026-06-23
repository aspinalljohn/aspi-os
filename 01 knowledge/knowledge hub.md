---
title: Knowledge Hub
tags: [moc, hub]
type: moc
---

# Knowledge Hub

Map of content for **digested information** → articles, threads, PDFs, research, and reference material you've consumed and want to keep. Faceted by topic below.

## Recently added

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Topic
FROM "01 knowledge"
WHERE file.name != this.file.name
SORT file.mtime DESC
LIMIT 15
```

## Everything, by topic

```dataview
LIST
FROM "01 knowledge"
WHERE file.name != this.file.name
GROUP BY file.folder
```

## Related hubs

[[home]] · [[venture-one hub]] · [[venture-two hub]] · [[venture-three hub]] · [[fractional hub]] · [[me hub]] · [[projects hub]] · [[resources hub]] · [[vault operations hub]]
