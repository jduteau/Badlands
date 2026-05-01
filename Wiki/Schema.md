---
wiki-version: "1.0"
last-updated: 2026-04-30
maintained-by: llm-wiki
type: schema
---

# Wiki Schema
Ironsworn: Badlands solo campaign world knowledge base. Setting: the western
frontier, post-Civil War.
Serves two consumers: the GM (browsing Obsidian) and the world arbiter
agent (querying during play). Write every page to serve both.

---

## Namespace Conventions

- Top-Level: `Wiki/Locations`, `Wiki/NPCs`, `Wiki/PCs`, `Wiki/Factions`, `Wiki/Sessions`, `Wiki/Threats`, `Wiki/World`, `Wiki/Lore`, `Wiki/State`
- Page Naming: Title Case, hyphens for multi-word (`Wiki/NPCs/Person/Jake-Powell`)
- Max Depth: 3 levels (e.g., `Wiki/NPCs/Person/NPCName`)
- Hub Pages: Every namespace level has a hub page listing its children
- Folder Hierarchy: Namespaces map to folders (e.g., `Wiki/Factions/The-Calvert-Gang.md`)

### Namespace Definitions

| Namespace | Contains | Page Type |
|-----------|----------|-----------|
| `Wiki/Locations` | Places: towns, ranches, trails, landmarks, wilderness regions | entity |
| `Wiki/NPCs` | Named characters the PC has met or heard of | entity |
| `Wiki/PCs` | The player character (the drifter) | entity |
| `Wiki/Factions` | Gangs, outfits, institutions, communities with collective agenda | entity |
| `Wiki/Sessions` | Individual play session records | project |
| `Wiki/Threats` | Synthesized analysis of active dangers and escalating situations | knowledge |
| `Wiki/World` | Frontier canon: territory history, frontier context, public knowledge | knowledge |
| `Wiki/Lore` | In-world documents, wanted posters, rumors, legends | knowledge |
| `Wiki/State` | Current session state block (overwritten each session) | knowledge |

---

## Page Types and Required Properties

### When to Use Each Type

**entity** — Use for anything with identity and persistence in the world: a
named person, a named place, a named organization. If it has a name and the PC
could encounter it, it is an entity. Subtype is determined by the entity-type
field.

**project** — Use for sessions only. A session is a discrete play event with a
date, a location, and outcomes. Not used for anything else in this wiki.

**knowledge** — Use for synthesized analysis, world reference material,
in-world documents, and the current state block. If it is a fact about the
world rather than a named thing in the world, it is knowledge. Subtype is
determined by the domain field.

**feedback** — Not used in this wiki.

**hub** — One per namespace. Structural index page only.

---

### Entity

An entity is a named, persistent thing in the world: a person, place, or
organization. Every named NPC, location, faction, and the player character
gets its own entity page.

**How to classify during ingest:**

- The player character → entity-type: pc
- Named person or character → entity-type: npc
- Named place (town, ranch, landmark, region, wilderness area) → entity-type: location
- Named group, gang, outfit, community, institution → entity-type: faction

**Required YAML frontmatter (all entities):**

```yaml
type: entity
entity-type: pc | npc | location | faction
name: [canonical name]
status: active | inactive | destroyed | dead | unknown
source: emergent | world-canon | player-created
confidence: canon | established | inferred | emergent
created: YYYY-MM-DD
updated: YYYY-MM-DD
```

---

#### entity-type: pc

The player character — the drifter. One page only. This page is the mechanical
and narrative record of the PC. Because this is a solo campaign, there is no
player/GM split; all content on this page is visible to both roles. The
`gm_only` flag is never used on the PC page.

**Required YAML frontmatter (additional fields):**

```yaml
player: [player name]

# Stats (each 1–3; set at creation, updated only on Advance)
stat_edge: [integer 1–3]
stat_heart: [integer 1–3]
stat_iron: [integer 1–3]
stat_shadow: [integer 1–3]
stat_wits: [integer 1–3]

# Tracks (live — overwrite each session)
track_health: [integer 0–5]
track_spirit: [integer 0–5]
track_supply: [integer 0–5]
track_momentum: [integer -6 to +10]
track_momentum_max: [integer; default 10; -1 per active debility]
track_momentum_reset: [integer; default +2; -1 per active bane]

# Debilities — Conditions (live — overwrite each session; true = marked)
debility_wounded: true | false
debility_shaken: true | false
debility_unprepared: true | false

# Debilities — Banes (permanent-ish; true = marked)
bane_maimed: true | false
bane_corrupted: true | false

# Experience
xp_earned: [integer; total XP gained from fulfilled vows]
xp_spent: [integer; total XP spent on Advance]

# Status
pc_status: active | retired | dead
last_location: "[[Wiki/Locations/...]]"
first_session: 1
```

**Notes on track_momentum_max and track_momentum_reset:**

- `track_momentum_max` starts at 10; subtract 1 for each active debility
  (wounded, shaken, unprepared, maimed, corrupted).
- `track_momentum_reset` starts at +2; subtract 1 for each active bane
  (maimed, corrupted). This is the value momentum resets to after burning.
- Recalculate both fields whenever a debility or bane is marked or cleared.

**Required sections:**

- **Description** — appearance, manner, notable traits; what makes this drifter
  memorable on the frontier
- **Background** — origin, what set them on the road, what drives them
- **Assets** — all assets the character currently holds. For each asset, list
  the asset name, its category (path, companion, talent, ritual), and which
  ability boxes are checked (e.g. `[x] [ ] [ ]`). Update on Advance; tag
  [Session N].
  - One asset must be the **Drifter** path asset (required at creation).
  - Starting assets: 3 (one must be Drifter).
  - Advance: 2 XP to check a new ability box; 3 XP to add a new asset.
- **Companion** — if the PC has a companion asset (e.g. Horse), record it here
  as a subsection. Include: companion name, asset name and checked abilities,
  current health track (0–5), and any narrative notes. Update health and
  status each session; tag significant changes [Session N].
  - Companion health is tracked here on the PC page, not on a separate page.
  - If the companion is gravely wounded or lost, note it here immediately.
- **Vows** — all active and completed vows. For each vow, record:
  - Name of vow
  - Rank: Troublesome | Dangerous | Formidable | Extreme | Epic
  - Progress: boxes filled (0–10); ticks within current box (0–3)
  - Status: active | fulfilled | forsaken
  - XP awarded: [integer, if fulfilled]
  - Brief narrative note on origin
  - Update progress and status each session; tag [Session N].
  - Active vows are also summarized as live YAML (see below).
- **Bonds** — narrative account of bonds formed, tested, and cleared. For each
  bond, note: the person or community bonded with, which session it was forged,
  whether it has been tested or cleared. Update each session; tag [Session N].
  - Bond progress track: ticks accumulated toward the next bond (40 ticks = 10
    boxes = full bond progress track, used when Write Your Epilogue).
  - Bonds grant +1 on relevant moves (see rules); record active bonds clearly.
- **Relationships** — how the PC relates to significant NPCs; tag changes [Session N]
- **History** — significant events in session order; tag [Session N]
- **Advancement Log** — XP earned, XP spent, assets added or upgraded; tag [Session N]
- **Cross-References**

**Live vow YAML block** (embed in the Vows section for quick arbiter access):

```yaml
active_vows:
  - name: [vow name]
    rank: Troublesome | Dangerous | Formidable | Extreme | Epic
    progress_boxes: [integer 0–10]
    progress_ticks: [integer 0–3]
  # repeat for each active vow
```

---

#### entity-type: npc

Named characters in the world. Includes allies, antagonists, townsfolk,
lawmen, outlaws, and anyone else with a recurring or significant presence.

**Required YAML frontmatter (additional fields):**

```yaml
npc_status: active | dead | missing | imprisoned | unknown
npc_scope: ally | antagonist | neutral | lawman | outlaw | civilian | threat
last_location: "[[Wiki/Locations/...]]"
disposition: friendly | neutral | suspicious | hostile | unknown
disposition_known: [what the PC believes]
disposition_true: [GM truth — may differ]
faction: "[[Wiki/Factions/...]]"   # omit if none
first_session: [integer]
bond_held: true | false   # true if the PC holds a bond with this NPC
```

**npc_scope definitions:**

- `ally` — someone who has actively helped the PC; may share a bond.
- `antagonist` — an active threat or enemy with personal or factional enmity.
- `neutral` — not yet aligned; could go either way.
- `lawman` — a sheriff, marshal, Pinkerton, or other law presence; alignment
  varies by campaign.
- `outlaw` — a known criminal or outlaw; not necessarily an antagonist.
- `civilian` — a non-combatant with a recurring presence: bartender, rancher,
  homesteader, merchant.
- `threat` — a named dangerous figure without significant social presence;
  primarily a combat or encounter entity. Elevate to antagonist if they develop
  narrative weight.

**Required sections:**

- **Description** — appearance, manner, memorable traits; how the PC first
  encountered them
- **Role** — function in the frontier world and in the campaign
- **PC Relationship** — current standing with the PC; update each change;
  tag changes [Session N]
- **What the PC Knows** — information established in play; facts and impressions
- **GM Truth** — actual allegiance, hidden motives, connections
- **Bond Status** — whether a bond exists, how it was forged, whether it has
  been tested or cleared; tag [Session N]
- **History** — significant events in session order; tag [Session N]
- **Cross-References**

---

#### entity-type: location

Named places on the frontier.

**Required YAML frontmatter (additional fields):**

```yaml
region: [territory or area name, e.g. "Red Creek Valley"]
location_status: intact | damaged | abandoned | contested | unknown
scale: territory | town | site
bond_held: true | false   # true if the PC holds a bond with this community
```

**scale definitions:**

- `territory` — large geographic region: a valley, a stretch of badlands, a
  river corridor.
- `town` — a settlement: a cattle town, a mining camp, a rail stop, a
  homestead cluster.
- `site` — a specific place: a ranch, a canyon, a cave, a waystation, a ruin.

**Required sections:**

- **Overview** — what this place is, why it matters to the campaign
- **Description** — physical appearance, atmosphere, who lives here
- **Current Status** — what has changed as a result of PC action or world
  events; update each visit; tag [Session N]
- **Known Inhabitants** — NPCs and factions present, with links
- **Bond Status** — whether the PC holds a bond with this community; how it was
  forged; whether tested or cleared; tag [Session N]
- **PC History** — chronological visits and events; tag [Session N]
- **GM Notes** — secrets, hooks, what could happen here
- **Cross-References**

---

#### entity-type: faction

Named groups with collective agenda and presence.

**Required YAML frontmatter (additional fields):**

```yaml
faction_type: gang | outfit | lawmen | institution | community | railroad | other
scale: local | regional | territorial
known_to_pc: true | partial | false
allegiance: hostile | neutral | friendly | unknown
```

**Required sections:**

- **Overview** — purpose, reach, history in the territory
- **Leadership** — known leaders; note what the PC knows vs. GM truth
- **Goals** — stated goals and true goals
- **PC Relationship** — current standing with the PC; tag changes [Session N]
- **Threat Connections** — links to `Wiki/Threats` pages and other Factions
- **GM Notes** — future plans, hooks
- **Cross-References**

---

### Project

Used for sessions only.

**Required YAML frontmatter:**

```yaml
type: project
status: completed | active | abandoned
session_number: [integer]
title: [short descriptive title]
date_played: YYYY-MM-DD
primary_location: "[[Wiki/Locations/...]]"
created: YYYY-MM-DD
updated: YYYY-MM-DD
```

**Required sections:**

- **Summary** — 2–3 sentences: what happened, what changed
- **Narrative** — full session blog post from session scribe; paste directly;
  immutable once filed; never edit
- **Key Events** — bullet list of significant moments; cross-reference all
  NPCs, Locations, Factions affected
- **Moves Triggered** — significant move outcomes (strong hits, weak hits,
  misses, momentum burns) that shaped events; reference move name and result
- **Foe Progress Tracks** — for any named foe fought this session, record the
  foe name, rank, harm dealt (progress marked), and outcome (ongoing / ended).
  This is the canonical record for foe combat state; it is not tracked elsewhere.
- **World State Changes** — what changed as a result of this session; feeds the
  update pass on other pages after ingest
- **Open Threads** — leads, questions, unresolved situations; link to Threats
  or NPC pages where they live
- **PC Status at End** — location, all track values (health, spirit, supply,
  momentum), debilities marked, active vow progress snapshot, immediate situation
- **Cross-References**

Sessions are immutable once filed. Never overwrite session content.

---

### Knowledge

Used for threat analysis, world canon, and in-world lore.

**How to classify during ingest:**

- Analysis of how factions/NPCs connect, escalating danger threads → domain: threat
- Facts from frontier history, territorial context, public institutions → domain: world
- In-world documents, wanted posters, rumors, legends, hearsay → domain: lore
- Current session state block → domain: state

**Required YAML frontmatter (all knowledge pages):**

```yaml
type: knowledge
domain: threat | world | lore | state
confidence: canon | established | inferred | emergent
created: YYYY-MM-DD
updated: YYYY-MM-DD
```

---

#### domain: threat

Synthesized analysis of an active danger, escalating situation, or persistent
enemy force. One page per distinct threat thread.

**Required YAML frontmatter (additional fields):**

```yaml
threat_tier: emerging | active | escalating | resolved
pc_awareness: unaware | partial | significant | full
```

**Required sections:**

- **The Threat** — who or what is behind it, what they want, what they are doing
- **Known Evidence** — facts the PC has discovered; session refs
- **Hidden Evidence** — GM-only facts not yet discovered
- **Key Actors** — NPCs and Factions with links
- **Connections** — links to other Threat pages
- **Status** — what the PC has done to this thread; tag [Session N]
- **Cross-References**

---

#### domain: world

Reference material about the frontier territories, their history, and the
broader western context of the campaign.

**Required YAML frontmatter (additional fields):**

```yaml
world_category: territory-history | frontier-context | institution | geography | culture | public-record
canon_source: [e.g. "Ironsworn Badlands rules" | "emergent" | "campaign-established"]
```

**Required sections:**

- **Overview** — what this world entry covers
- **Relevance to Campaign** — how it connects to active content; grows as the
  campaign develops
- **Cross-References**

---

#### domain: lore

In-world documents, wanted posters, rumor-mill accounts, and other textual
artifacts discovered or referenced during play.

**Required YAML frontmatter (additional fields):**

```yaml
lore_type: wanted-poster | rumor | document | legend | letter | newspaper
reliability: reliable | unreliable | planted | unknown
discovered_session: [integer]
source_location: "[[Wiki/Locations/...]]"   # omit if not location-specific
```

**Required sections:**

- **Content** — the actual text or account, written in in-world voice
- **GM Analysis** — what is true, false, or misleading
- **Connections** — what this lore points toward
- **Cross-References**

---

#### domain: state

The current session state block. Lives at `Wiki/State/Current-Session.md`.
This page is OVERWRITTEN (not appended) after each session. It is the only
page in the wiki that is not append-only.

```yaml
domain: state
confidence: established
```

**Content format:**

```
PC: [[Wiki/PCs/Name]] — last location: [[Wiki/Locations/...]]

Tracks:
  Health: [n]/5 | Spirit: [n]/5 | Supply: [n]/5
  Momentum: [n] (max [n], reset [n])

Debilities: [list marked, or "none"]
Banes: [list marked, or "none"]

Companion: [Name] — Health: [n]/5 | [status note or "healthy"]

Active Vows:
  - [Vow Name] ([Rank]) — [n]/10 boxes

Active Bonds:
  - [Person or community name] ([[Wiki/NPCs/...]] or [[Wiki/Locations/...]])

Active Threats:
  - [[Wiki/Threats/...]]

Situation: [one sentence]

Last Session: [[Wiki/Sessions/Session-NN-Title]]
```

This page feeds the world arbiter at session start. Keep it minimal.
Companion line may be omitted if the PC has no active companion asset.

---

### Hub

Standard hub page. One per namespace. No changes from base schema.

```yaml
type: hub
namespace: Wiki/NamespaceName
```

---

## Confidence Levels

| Level | Meaning |
|-------|---------|
| `canon` | Directly stated in Ironsworn Badlands rules. Do not contradict without explicit campaign decision. |
| `established` | Confirmed in play, recorded in a Session page. Authoritative for this campaign. |
| `inferred` | Logical conclusion from canon or established facts. Flag when asserting. |
| `emergent` | Created by GM improvisation during play. Treat as established once filed and verified. |

Campaign facts supersede world canon where they conflict.

---

## Append and Contradiction Rules

- APPEND ONLY — never delete or overwrite existing content
- Exceptions:
  - `Wiki/State/Current-Session.md` is overwritten each session
  - Live YAML track fields on the PC page (health, spirit, supply, momentum,
    debilities, banes, xp_earned, xp_spent, active_vows) are overwritten each
    session — these are current-state fields, not history
- Tag all session additions to narrative sections: [Session N]
- If new information contradicts existing content, flag explicitly:

```
> ⚠️ CONTRADICTION [Session N]: [description of conflict]
```

Do not silently resolve contradictions. Surface them for human review.

---

## Cross-Reference Rules

- Every wiki page MUST have at least one `[[Wiki/...]]` link to another wiki page
- Hub pages MUST list ALL child pages in their namespace
- When a page mentions an entity that has its own page, use `[[Wiki/Entity/Name]]` link syntax
- Tags: use `#tag` for lightweight categorization (e.g., `#outlaw`, `#badlands`, `#vow`)
- External links: `[Text](URL)` for URLs outside the wiki

---

## Content Format Rules

- Flat Markdown (no outliner `- ` prefix required on every line)
- Properties: YAML frontmatter between `---` fences at the top of the file
- Sections: standard `## Heading` syntax
- Code blocks: fenced with triple backticks
- NEVER store credentials, passwords, or API tokens in wiki pages

---

## L1/L2 Architecture

- The standard llm-wiki L1 memory path is disabled for this wiki.
- No credentials or sensitive data exist in campaign knowledge.
- All content including session state lives in the git-tracked vault.

### L1 equivalent = Wiki/State/Current-Session.md

- Loaded explicitly at session start by the world arbiter.
- Contains: PC track values, debilities, companion status, active vows, active
  bonds, active threats, current situation, pointer to last session page.

### L2 = Everything else in the wiki

- Queried on demand: Locations, NPCs, PCs, Factions, Sessions,
  Threats, World, Lore.

### Boundary Rules

- If the arbiter needs it at the start of every session to orient itself → it belongs in the state page.
- Everything else → L2, queried when needed.

---

## Ingest Source Routing

Identify the source type and apply these rules before extracting.

**Character sheet / PC creation:**

- Create `Wiki/PCs/[Name].md` with `entity-type: pc`
- Set `confidence: established`, `source: player-created`
- Set all five stat fields to creation values
- Set health, spirit, supply to 5; momentum to +2; momentum_max to 10;
  momentum_reset to +2
- Set all debility and bane fields to false
- Set xp_earned and xp_spent to 0
- Record starting 3 assets in the Assets section; one must be Drifter
- Record companion asset in the Companion section if taken at creation
- Record starting vows in the Vows section and in the active_vows YAML block
- Record starting bonds in the Bonds section
- Update the `Wiki/PCs` hub
- Create `Wiki/State/Current-Session.md` with initial state

**Session transcript:**

- Create a session (project) page
- Overwrite all live track fields on the PC page to end-of-session values
- Overwrite the active_vows YAML block on the PC page
- Append to the Vows section for progress milestones or fulfilled/forsaken vows; tag [Session N]
- Append to the Bonds section for bonds forged, tested, or cleared; tag [Session N]
- Append to the Companion section for health changes or significant events; tag [Session N]
- Update `bond_held` on affected NPC and Location pages
- Update `last_location` on the PC page
- Update `npc_status`, `disposition_known`, and `disposition_true` on NPC pages
  where disposition changed
- Update `location_status` on Location pages where status changed
- Append to **History** on the PC page for significant events; tag [Session N]
- Append to **Advancement Log** on the PC page for any Advance taken; tag [Session N]
- Overwrite `Wiki/State/Current-Session.md` with updated state block
- File World State Changes as update candidates for affected pages

**Named threat (improvised or escalating):**

- Named antagonist with social presence and agenda → NPC entity page with
  `npc_scope: antagonist`
- Named gang or hostile outfit → Faction entity page
- Escalating danger with multiple actors → Threat knowledge page
- Generic unnamed foe (bandits, wolves, hired guns) belongs in the session
  note only, not this world wiki

**World canon sources (Ironsworn Badlands rules, setting material):**

- Create world (knowledge) pages only
- Set `confidence: canon`
- Do NOT overwrite established campaign facts with world canon
- Campaign layer supersedes world layer where they conflict

---

## Ingest Workflow

1. Identify source type (character sheet | session | threat | world canon)
2. Apply source routing rules above
3. Extract: named entities, facts, relationships, track changes, vow progress,
   bond changes, move outcomes
4. Classify each item by namespace and page type using definitions above
5. Scan wiki: identify existing pages to update, new pages to create
6. Target: 5–15 page touches per ingest
7. Create new pages with all required properties
8. Existing pages: APPEND only to narrative sections — never overwrite
   (exceptions: `Wiki/State/Current-Session.md` and live YAML fields on PC page)
9. Update hub pages to include new child pages
10. Add cross-references between all affected pages
11. Set `updated` on all changed pages
12. Git commit after ingest

---

## Lint Rules

**Standard rules:**

- **Orphan Detection** — pages with 0 incoming links (hubs excluded)
- **Stale Detection** — `updated` > 180 days AND confidence is canon or established
- **Missing Properties** — pages missing type-specific required properties
- **Broken References** — links to non-existent pages
- **Hub Completeness** — hub pages missing children in their namespace
- **Credential Leak** — scan for token/password/secret patterns
- **Empty Pages** — only properties, no content
- **Cross-Ref Minimum** — fewer than 1 outgoing link

**Campaign-specific rules:**

- PC page missing any stat field → flag
- PC page missing any track field (health, spirit, supply, momentum) → flag
- PC page missing any debility or bane field → flag
- PC page with active companion asset but no Companion section → flag
- PC page with `bane_maimed: true` or `bane_corrupted: true` and
  `track_momentum_reset` not adjusted → flag
- PC page with active debilities and `track_momentum_max` not adjusted → flag
- PC page with `xp_earned` minus `xp_spent` less than 0 → flag (XP error)
- Active vow in `active_vows` YAML block with no corresponding entry in Vows
  section → flag
- Vow marked `status: fulfilled` with no XP recorded → flag
- NPC page missing `disposition_true` → flag
- NPC page with `bond_held: true` where PC Bonds section has no matching entry
  → flag (Bond mismatch)
- Location page with `bond_held: true` where PC Bonds section has no matching
  entry → flag (Bond mismatch)
- Location page missing `location_status` → flag
- Session page missing Foe Progress Tracks section → flag
- Session page missing Moves Triggered section → flag
- Session page missing PC Status at End section → flag
- Threat page with fewer than 1 linked NPC or Faction page → flag
- NPC marked `npc_status: dead` appearing in sessions after their
  status-change session → flag
- PC page with `pc_status: active` not updated within last 3 sessions → flag
  (track values may be stale)
- State block referencing a PC name with no corresponding `Wiki/PCs/` page
  → flag
- Contradiction flags (⚠️) present → surface for human review
- Pages with `confidence: inferred` not updated in 10+ sessions → flag as
  potentially stale
