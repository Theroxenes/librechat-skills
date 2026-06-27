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
actor.system.attributes.hp.value;
actor.system.attributes.ac.perception;
actor.system.level;
spell.system.actionDC.dc;
feat.system.traits;
```

Read via public API only — check source before assuming internal structure.

### Chat Cards

```javascript
await ChatMessagePF2e.whisperToGMs({ content: "..." });
```

### Conditions & DC Calculations

```javascript
actor.hasCondition("fear");
const dc = new DC({ value: 19, kind: "class", proficiency: 3 });
```

### i18n Localization

Use `_loc()`. Ships with main, actions, rules elements, Kingmaker packs.

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
