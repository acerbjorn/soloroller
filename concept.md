# Concept: OSR Hexmap & Character Tracker (Solo Roleplay)

A single-page app for solo OSR play (Old School Essentials, Basic/Expert D&D and
lookalikes): a hexmap in the center for wilderness exploration, and a character
tracker where each character is a card you can move between hexes or group with
other characters via drag and drop.

## Core UX loop

1. Hexmap fills the center of the screen.
2. Click a hex -> side panel lists the characters in that hex.
3. Characters show as small cards, collapsed by default (portrait, name, key
   stats). Click a card to expand it into a full character sheet.
4. Drag and drop is the main interaction:
   - Drag a character card onto another hex -> move there.
   - Drag a character card onto another character -> form/join a group.
   - Grouped characters share their group's hex; dragging the group moves all.

## Character sheet (OSR-flavored fields)

- Identity: name, class, level, XP, alignment, race/ancestry, portrait
- Ability scores: STR, DEX, CON, INT, WIS, CHA (with modifiers)
- Combat: HP (current/max), AC, THAC0 (or attack bonus), damage, initiative
- Saving throws: Death, Wands, Paralysis, Breath Attacks, Spells (or the modern 5)
- Gear: equipment list, encumbrance, gold/treasure, carried items
- Magic-user extras: spell slots by level, memorized spells, languages
- Notes: free-text, conditions, wounds, session log

## Hexmap & OSR play features

- Axial-coordinate hex grid (q, r), pointy-top rendering in SVG.
- Per-hex data: terrain type, notable feature, cleared/explored flag.
- Random encounter hook: per-terrain encounter table, roll on entering a hex
  (solo-mode toggle).
- Travel & time: Paradox-style movement — set a destination, an arrow is drawn,
  and the marker advances along it as the in-app clock ticks (see below).

## Travel & in-app time (Paradox-style)

- The app has a clock with pause / 1x / 2x / 3x speed controls; all movement and
  events advance with it. Time granularity: hours (grouped into watches).
- Setting a destination: select a character or group, then click a destination
  hex. The app computes a path over the hex grid (A*, terrain-weighted) and
  draws an arrow along the path from the current hex to the destination,
  labeled with the ETA.
- The marker advances along the arrow as time ticks. Speed per hex comes from a
  terrain-cost table modified by pace (cautious / normal / forced march),
  encumbrance, and mounts.
- Multiple groups can have active orders simultaneously; each shows its own
  arrow. Changing or canceling the destination recomputes or removes the arrow;
  an unreachable destination (impassable terrain) shows an invalid arrow and
  issues no order.
- Entering a new hex triggers a random-encounter roll for that terrain; on a
  hit, time pauses and the encounter is resolved before travel resumes.
- Resting / making camp sets pace to zero and advances the clock in watches.
- Drag-and-drop onto a hex remains an instant move (useful for setup); travel
  orders via destination-click are the in-fiction way to move.

## Tech stack

- React + TypeScript + Vite (single-page app, no backend).
- SVG hexmap: one `<polygon>` per hex; click/hover hit-testing for free.
- dnd-kit for pointer-based drag and drop (mouse + touch), with drop-target
  highlighting (hex = move, character card = group).
- State: Zustand or plain React context; normalized entities with stable ids.
- Persistence: autosave to IndexedDB via Dexie (localStorage fallback), plus
  JSON export/import of the whole campaign. Optional PWA for offline use.

## Data model sketch

```ts
interface Hex { q: number; r: number; terrain: string; feature?: string; explored: boolean }
interface Character {
  id: string; name: string; groupId: string | null;
  hex: { q: number; r: number };          // own hex; effective hex = group's if grouped
  level: number; xp: number; class: string; alignment: string;
  abilities: Record<'str'|'dex'|'con'|'int'|'wis'|'cha', number>;
  hp: { current: number; max: number }; ac: number; thac0: number;
  saves: Record<string, number>;
  equipment: string[]; gold: number; spells: string[];
  notes: string; portrait?: string;
}
interface Group { id: string; name: string; hex: { q: number; r: number } }
```

Effective position of a character = its group's hex if `groupId` set, else its own.

## Phases

1. Static hexmap renders; click hex -> empty panel. Save/load works.
2. Add/edit character; collapsed card list in panel; expandable full sheet.
3. Drag and drop: move to hex, group with character, move groups.
4. OSR mechanics: movement orders + in-app clock, encounter rolls, death & replacement.
5. Polish: export/import, print/PDF sheet, PWA offline.

## Open questions

- THAC0 vs. ascending AC / attack bonus (support both? default THAC0 for OSE feel).
- Encounter tables: built-in generic OSR tables, or user-importable only?
- One campaign at a time, or multiple campaigns in one install?
- Should groups also be drag targets onto hexes (move whole party)?

## Decisions

- Both movement modes are intentional: drag-and-drop onto a hex = instant move
  (setup/admin); destination-click = Paradox-style travel order with arrow.
