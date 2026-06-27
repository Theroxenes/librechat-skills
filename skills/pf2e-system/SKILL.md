---
name: pf2e-system
description: >
  Work with the Pathfinder 2nd Edition (PF2e) game system for Foundry VTT. Use this
  skill when the user asks about PF2e-specific APIs, data structures, or code —
  including actor/item preparation, PF2e document schemas, the game.pf2e namespace,
  compendium pack manipulation specific to PF2e, spell/feat/action data, conditions,
  DC calculations, chat card rendering, or any PF2e system-internal behavior. Also
  triggers when the user mentions "game.pf2e", "actor.system""TraitsPF2e",
  "ChatCardPF2e", If the task is about general Foundry VTT APIs (hooks, documents,
  modules) without PF2e specifics, use the foundry-vtt-developer skill instead.
---

# PF2e System Developer

Work with the Pathfinder 2nd Edition game system for Foundry VTT v14. The PF2e system is a TypeScript project built with Vite. All code runs in the Foundry client context.

## Authoritative References

| Resource | URL |
|---|---|
| pf2e-types (TypeScript types) | https://github.com/7H3LaughingMan/pf2e-types |
| PF2e System GitHub | https://github.com/foundryvtt/pf2e |
| PF2e Wiki | https://github.com/foundryvtt/pf2e/wiki |
| Foundry VTT API docs (v14) | https://foundryvtt.com/api/ |

Always check the PF2e source repo or the pf2e-types repository before assuming behavior — the system has extensive custom APIs beyond core Foundry.

## Project Structure

The PF2e system follows this layout (from `github.com/foundryvtt/pf2e`):

```
src/
├── pf2e.ts              # Entry point — imports hooks, styles, exports public API
├── global.ts            # Global type augmentations
├── module/
│   ├── actor/           # ActorPF2e, character helpers, party logic
│   ├── item/            # ItemPF2e subclasses (spells, feats, equipment)
│   ├── apps/            # ApplicationV2 UI components
│   ├── canvas/          # Canvas overlays and token rendering
│   ├── chat-message/    # Chat card rendering
│   ├── collection/      # Document collection overrides
│   ├── sheet/           # Actor/Item sheet implementations
│   └── system/          # System configuration and helpers
├── scripts/
│   └── hooks/           # Hook listeners, each with .listen() method
└── util/                # Shared utilities
```

Entry point pattern (`src/pf2e.ts`):
```typescript
import { HooksPF2e } from "@scripts/hooks/index.ts";
import "./styles/main.scss";

HooksPF2e.listen();

export { RuleElement } from "@module/rules/index.ts";
```

Hook registration collects all listeners in a central registry. Each hook is a class with a static `.listen()` method that calls `Hooks.on` or `Hooks.once`.

## Key PF2e APIs

### game.pf2e Namespace

The system exposes custom APIs under `game.pf2e`. Access this only after the `ready` hook — it's populated during initialization. Common sub-namespaces include settings, rules engine state, and utility functions.

### Actor & Item Data

PF2e documents use a `.system` namespace for game-specific data:

```javascript
// PF2e actor properties
actor.system.attributes.hp.value;
actor.system.attributes.ac.perception;
actor.system.level;

// Items have type-specific system data
spell.system.actionDC.dc;
feat.system.traits;
```

Always read through the document's public API — never assume internal structure without checking the source.

### Chat Cards

PF2e renders rich chat cards for actions, spells, and effects. The `ChatMessagePF2e` class handles rendering:

```javascript
// Create a PF2e chat card
await ChatMessagePF2e.whisperToGMs({ content: "..." });
```

### Conditions & DC Calculations

PF2e has dedicated helpers for conditions and difficulty classes:

```javascript
// Check if actor has a condition
actor.hasCondition("fear");

// Calculate a DC
const dc = new DC({ value: 19, kind: "class", proficiency: 3 });
```

### i18n Localization

Use `_loc()` for string localization in PF2e context. The system ships with multiple language packs (main, actions, rules elements, Kingmaker).

## TypeScript Configuration & Types

The PF2e project targets ES2024 with strict mode and path aliases. Use `@7h3laughingman/pf2e-types` for comprehensive type definitions covering all PF2e classes, global augmentations, and system APIs:

```json
{
  "devDependencies": {
    "@7h3laughingman/foundry-types": "~14.360.8",
    "@7h3laughingman/pf2e-types": "^8.0.2"
  }
}
```

### Looking Up PF2e Types and Class Definitions

When you need to verify a PF2e class definition, method signature, `game.pf2e` namespace shape, or any system-specific type — check the pf2e-types repository at https://github.com/7H3LaughingMan/pf2e-types before running a web search. It mirrors the PF2e source structure with full `.d.ts` declarations and is the most reliable source for exact API surfaces.

Key files to check:
- `src/global.d.ts` — Foundry global namespace augmentations (`game`, `canvas`, `ui`)
- `src/global-exports.d.ts` — Exported class and interface declarations
- `src/global-functions.d.ts` — Global helper function signatures
- `src/module/` — PF2e module type definitions (actor, item, scene, system)

Always check the pf2e-types source for exact class definitions and method signatures before coding.

## Common Pitfalls

1. **PF2e data lives in `.system`** — never mutate `actor.system` directly; use the document's update API
2. **Check PF2e source or pf2e-types before coding** — the system has deep custom APIs that differ from core Foundry patterns
3. **`game.pf2e` is undefined before ready** — guard access with `Hooks.once("ready")` or check for existence
4. **For general Foundry APIs (hooks, documents, modules), use the foundry-vtt-developer skill** — this skill covers only PF2e-specific behavior
