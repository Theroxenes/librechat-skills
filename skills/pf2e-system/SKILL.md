---
name: pf2e-system
description: >
  Work with the Pathfinder 2nd Edition (PF2e) game system for Foundry VTT. Use for
  PF2e-specific APIs, data structures, code — actor/item prep, document schemas,
  game.pf2e namespace, compendium manipulation, spell/feat/action data, conditions,
  DCs, chat cards. Triggers: "game.pf2e", "actor.system", "TraitsPF2e",
  "ChatCardPF2e". For general Foundry APIs (hooks, docs, modules) without PF2e
  specifics, use foundry-vtt-developer instead.
---

# PF2e System Developer (v14, TS/Vite, browser context)

## Authoritative References

- pf2e-types: <https://github.com/7H3LaughingMan/pf2e-types>
- PF2e System GitHub: <https://github.com/foundryvtt/pf2e>
- Wiki: <https://github.com/foundryvtt/pf2e/wiki>
- Foundry API (v14): <https://foundryvtt.com/api/>

Always check PF2e source or pf2e-types before assuming — extensive custom APIs beyond core Foundry.

## Project Structure

Entry: `src/pf2e.ts` (imports hooks/styles, exports public API). Key dirs: `module/actor`, `module/item`, `module/apps`, `module/canvas`, `module/chat-message`, `module/sheet`, `module/system`, `scripts/hooks`, `util/`.

Hooks: each listener is a class with static `.listen()` calling `Hooks.on`/`Hooks.once`, collected centrally.

## Key PF2e APIs

### game.pf2e Namespace

Access only after `ready` hook (populated during init). Contains settings, rules engine state, utilities.

### Actor & Item Data

```javascript
actor.system.attributes.hp.value;   // hit points
actor.system.attributes.ac.value;   // armor class
actor.system.perception.value;      // perception
actor.system.details.level.value;   // creature level
feat.system.traits;
```

Statistics are also exposed as objects on the actor (e.g. `actor.perception.dc`, `actor.armorClass.value`). Read via public API only — verify exact paths against source/pf2e-types before assuming.

### Chat Cards

```javascript
await ChatMessagePF2e.create({
  content: "...",
  whisper: game.users.filter((u) => u.isGM).map((u) => u.id),  // GM-only whisper
});
```

or from a PF2e system object:
```typescript
const mySpell = actor.itemTypes.spell.find((s) => s.slug === "my-spell");
await mySpell?.toMessage(event, { rollMode: "publicroll", create: true });
```

### Conditions & DC Calculations

```javascript
actor.hasCondition("frightened");          // boolean; accepts one or more condition slugs
actor.getCondition("frightened")?.value;   // active condition document, or null
actor.armorClass.value;                    // DCs/statistics are exposed as objects (AC, saves, perception)
```

### i18n Localization

Use `game.i18n.localize(key)` / `game.i18n.format(key, data)`. PF2e ships translation packs for the system, actions, and rule elements.

## TypeScript Configuration & Types

ES2024, strict, path aliases. Install types:

```json
{
  "devDependencies": {
    "@7h3laughingman/foundry-types": "~14.360.8",
    "@7h3laughingman/pf2e-types": "^8.0.2"
  }
}
```

### Looking Up Types

Always check pf2e-types (<https://github.com/7H3LaughingMan/pf2e-types>) before web search. Key files: `src/global.d.ts` (namespace augmentations), `src/global-exports.d.ts` (class declarations), `src/global-functions.d.ts` (helpers), `src/module/` (PF2e-specific types). Check source for exact signatures before coding.

## Common Pitfalls

1. Never mutate `.system` directly — use document update API. 2. Check source/pf2e-types before coding — deep custom APIs differ from core Foundry. 3. `game.pf2e` undefined before ready — guard with `Hooks.once("ready")`. 4. General Foundry APIs → use foundry-vtt-developer skill.
