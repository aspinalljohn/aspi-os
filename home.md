---
title: Home
tags: [moc, hub, home]
type: home
---

# Aspi OS → Home

The front door. Everything in the vault is reachable from here in two clicks: pick a hub, the hub indexes the rest.

## Hubs

- [[venture-one hub]] → your first daily venture
- [[venture-two hub]] → your second daily venture
- [[venture-three hub]] → your third daily venture
- [[me hub]] → personal brand, health, journal, reflection
- [[reading hub]] → digested articles, threads, research, reference by topic
- [[projects hub]] → standalone, time-bound projects
- [[resources hub]] → active reference kept at the ready (specs, configs, templates)
- [[vault operations hub]] → primitives (gobble, skillify) + every running automation

## Quick entry points

- `00 inbox/` → untriaged drop zone (should stay near empty)
- `01 reading/` → digested articles, threads, research, reference
- `05 projects/` → standalone, time-bound work
- `09 resources/` → active reference kept at the ready
- `99 archive/` → completed, dormant, retired

## Across the whole vault → recently touched

```dataview
TABLE dateformat(file.mtime, "yyyy-MM-dd") AS Modified, file.folder AS Where
FROM ""
WHERE file.name != this.file.name AND !contains(file.path, ".claude") AND !contains(file.path, "memory")
SORT file.mtime DESC
LIMIT 20
```

## Brain health → notes not yet connected

These notes have **zero** inbound links → they exist but the graph can't reach them. The goal is to drive this list toward empty (the link-gate keeps new notes off it).

```dataview
TABLE file.folder AS Where, dateformat(file.mtime, "yyyy-MM-dd") AS Modified
FROM ""
WHERE length(file.inlinks) = 0
  AND file.name != this.file.name
  AND !contains(file.path, ".claude")
  AND !contains(file.path, "memory")
  AND type != "moc"
SORT file.mtime DESC
LIMIT 50
```

## Resurface → revisit today

One valuable note to re-encounter, picked fresh each day (stable within the day). Serendipity so good thinking doesn't sink.

```dataviewjs
const pool = dv.pages()
  .where(p => ["playbook","reflection","research"].includes(p.type))
  .where(p => !p.file.path.includes("99 archive") && !p.file.path.includes(".claude") && !p.file.path.includes(".tmp"))
  .array();
if (!pool.length) { dv.paragraph("_No tagged notes in the pool yet._"); }
else {
  const day = Math.floor(Date.now() / 86400000);
  const pick = pool[day % pool.length];
  dv.paragraph(`**${pick.file.link}**  ·  _${pick.type}_  ·  last touched ${pick.file.mtime.toFormat("yyyy-MM-dd")}`);
}
```

## Review queue → valuable, but gone cold

High-value notes (playbooks, reflections, research) untouched in 90+ days. Skim periodically → update, link, or archive.

```dataview
TABLE type, dateformat(file.mtime, "yyyy-MM-dd") AS "Last touched"
FROM ""
WHERE (type = "playbook" OR type = "reflection" OR type = "research")
  AND file.mtime < date(today) - dur(90 days)
  AND !contains(file.path, ".claude") AND !contains(file.path, "memory") AND !contains(file.path, "99 archive")
SORT file.mtime ASC
LIMIT 15
```
