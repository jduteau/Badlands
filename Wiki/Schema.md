---
wiki-version: "1.0"
last-updated: "2026-04-30"
schema-version: "1.1"
maintained-by: llm-wiki
type: schema
---

# Wiki Schema

## Namespace Conventions

- Top-Level: Wiki/Locations, Wiki/NPCs, Wiki/PCs, Wiki/Factions, Wiki/Sessions, Wiki/Conspiracy, Wiki/World, Wiki/Lore, Wiki/State
- Page Naming: Title Case, hyphens for multi-word (`Wiki/Projects/My-Project`)
- Max Depth: 3 levels (e.g., `Wiki/Business/Clients/ClientName`)
- Hub Pages: Every namespace level has a hub page listing its children
- Folder Hierarchy: Namespaces map to folders (e.g., `Wiki/Tech/Docker.md`)

## Page Types and Required Properties

### PC (Player Character) — `Wiki/PCs/`

Required YAML frontmatter:

```yaml
type: pc
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active | retired | dead
player: "Player Name"
```

Required sections and fields:

**Stats** (each 1–3):
- `edge` — speed, ranged attacks, quick draws
- `heart` — courage, persuasion, social reads, endurance
- `iron` — strength, close combat, forceful action
- `shadow` — deception, stealth, manipulation, reading danger
- `wits` — observation, investigation, planning, survival

**Tracks:**
- `health` — 0–5 (starts at 5)
- `spirit` — 0–5 (starts at 5)
- `supply` — 0–5 (starts at 5)
- `momentum` — −6 to +10 (starts at +2)
- `momentum-max` — 10 minus 1 per active debility (default 10)

**Debilities** (each is true/false; each active one reduces momentum-max by 1):
- Conditions: `wounded`, `shaken`, `unprepared`
- Banes (permanent-ish): `maimed`, `corrupted`

**Experience:**
- `xp-earned` — total XP gained from fulfilled vows
- `xp-spent` — total XP spent on Advance

**Assets** (list each asset name and which ability boxes are checked):
- 3 starting assets required; one must be the **Drifter** path asset
- Upgrades cost 2 XP per ability box; new assets cost 3 XP

**Bonds:**
- Bond progress track (ticks, up to 40 ticks = 10 boxes)
- Named list of bonds with specific people or communities; note if a bond was cleared

**Vows / Progress Tracks** (one entry per active or completed vow):
- Name of vow, rank (Troublesome/Dangerous/Formidable/Extreme/Epic), filled boxes (0–10), status (active/fulfilled/forsaken)

**Background / Description:**
- Brief character concept, name, appearance, history

---

### Current_State — `Wiki/State/Current_State.md`

Required YAML frontmatter:

```yaml
type: state
updated: YYYY-MM-DD
session: <session number or title>
```

Required sections:

**Active PCs** — link to each `[[Wiki/PCs/Name]]` page with a one-line status note (health, spirit, supply, momentum, active debilities)

**Current Location** — link to `[[Wiki/Locations/Name]]`; brief note on situation there

**Active Vows** — for each PC, list active vows with rank and current progress (boxes filled)

**Active Journey** (if any) — destination, rank, progress boxes filled; note which move to use next

**Combat / Encounter in Progress** (if any) — foe name, foe progress track, who has initiative

**Recent Events** — 3–5 bullet points summarizing what happened in the last session (link to `[[Wiki/Sessions/Session-N]]`)

**Immediate Threats / Hooks** — unresolved dangers or open threads facing the party

**Key NPCs in Play** — link to `[[Wiki/NPCs/Name]]` for any NPC currently relevant; note their disposition and last known location

**World / Faction Developments** — any faction or world changes that emerged this session (link to `[[Wiki/Factions/Name]]` or `[[Wiki/World/...]]` as applicable)

---

### Entity (Person, Client, Tool, Service, Technology)

Required YAML frontmatter:

```yaml
type: entity
entity-type: person | client | tool | service | technology
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active | inactive | archived
source: memory-migration | ingest | manual
```

### Project

```yaml
type: project
status: active | completed | on-hold | cancelled
created: YYYY-MM-DD
updated: YYYY-MM-DD
started: YYYY-MM-DD
completed: YYYY-MM-DD  # if applicable
```

### Knowledge (Learning, Reference)

```yaml
type: knowledge
domain: tech | business | content | ops
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: high | medium | low | stale
```

### Feedback (Lessons Learned, Gotchas)

```yaml
type: feedback
severity: critical | important | nice-to-know
created: YYYY-MM-DD
verified: YYYY-MM-DD
applies-to: []  # page references to affected systems
```

### Hub (Namespace Index)

```yaml
type: hub
namespace: Wiki/NamespaceName
```

## Cross-Reference Rules

- Every wiki page MUST have at least one `[[Wiki/...]]` link to another wiki page
- Hub pages MUST list ALL child pages in their namespace
- When a page mentions an entity that has its own page, use `[[Wiki/Entity/Name]]` link syntax
- Tags: use `#tag` for lightweight categorization (e.g., `#docker`, `#deploy`, `#critical`)
- External links: `[Text](URL)` for URLs outside the wiki

## Content Format Rules

- Flat Markdown (no outliner `- ` prefix required on every line)
- Properties: YAML frontmatter between `---` fences at the top of the file
- Sections: standard `## Heading` syntax
- Code blocks: fenced with triple backticks
- NEVER store credentials, passwords, or API tokens in wiki pages

## L1/L2 Architecture

### L1 = Claude Memory (auto-loaded every session)

- Feedback rules and quick gotchas
- User identity (name, preferences)
- Credentials (MUST NEVER go into the wiki)
- Everything Claude needs to know at the START of every session

### L2 = Wiki (on-demand via `/wiki query`)

- Projects and their details
- Workflows and processes
- Research and learning notes
- Deep technical knowledge
- Business intelligence and strategy

### Boundary Rules

- New quick rule or gotcha discovered? --> Save to Claude Memory (L1)
- New project, workflow, or research? --> Save to Wiki (L2)
- Same info in L1 AND L2? --> Warning on `/wiki lint`

## Ingest Workflow

1. Analyze new source --> extract entities, facts, relationships
2. Identify affected wiki pages (existing + new)
3. Target: 5-15 page touches per ingest
4. Create new pages with all required properties
5. Existing pages: APPEND, never overwrite
6. Update hub pages
7. Add cross-references
8. Set `updated` property on all changed pages

## Lint Rules

- **Orphan Detection**: Pages with 0 incoming [[links]] (hub pages excluded)
- **Stale Detection**: `updated` > 90 days old AND `confidence: high`
- **Missing Properties**: Pages missing type-specific required properties
- **Broken References**: [[links]] to non-existent pages
- **Hub Completeness**: Hub pages missing children in their namespace
- **Credential Leak**: Scan for token/password/secret patterns
- **Empty Pages**: Only properties, no content
- **Cross-Ref Minimum**: Pages with fewer than 1 outgoing [[link]]
- **L1/L2 Duplicates**: Same info in Memory AND Wiki

## Conventions

- Language: English (customize per project)
- Dates: ISO 8601 (YYYY-MM-DD)
