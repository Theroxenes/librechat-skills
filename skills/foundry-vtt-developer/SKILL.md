---
name: foundry-vtt-developer
description: >
  Develop modules, macros, and code for Foundry VTT v14. Use when writing, debugging,
  or explaining Foundry VTT JS/TS code — module manifests, ESModules, Hooks,
  Document/DataModel, ApplicationV2, canvas, compendiums, macros. Triggers:
  "Foundry", "module.json", "game.ready", "Hooks.callOnce", "DocumentSheet",
  "Compendium", "canvas.tokens". Do NOT use for PF2e system internals — use
  pf2e-system skill instead.
---

# Foundry VTT Developer (v14) — all code runs in browser context

## Authoritative References

- pf2e-types: <https://github.com/7H3LaughingMan/pf2e-types>
- API docs (v14): <https://foundryvtt.com/api/>
- GitHub: <https://github.com/foundryvtt/foundryvtt>
- Module dev guide: <https://foundryvtt.com/article/module-development/>

Always check pf2e-types or official docs before guessing at class names/signatures.

## Core Concepts

### Package Types

- **Modules** (`module.json`): loaded via `esmodules` (preferred) or `scripts`
- **Macros**: JS snippets in Macro editor, no manifest needed

### Module Manifest (`module.json`)

Minimum viable:

```json
{
  "name": "my-module",
  "title": "My Module",
  "description": "Does something useful.",
  "version": "1.0.0",
  "compatibility": {
    "minimum": "14",
    "verified": "14.363",
    "maximum": "14"
  },
  "esmodules": ["my-module.mjs"]
}
```

- `name` MUST match folder name (silent failure if not). `compatibility.verified` = latest tested patch.

### Hook System

Register with `Hooks.on` (recurring), `Hooks.once` (one-time init), or `Hooks.callAll`. Never access game data before `ready`:

| Hook | When |
|---|---|
| `load` | Before world data loads |
| `setup` | After data loads, before ready |
| `ready` | Everything initialized — safe to access game data |
| `renderApplicationV2` | App renders |
| `createDocument`/`updateDocument`/`deleteDocument` | Doc lifecycle |
| `getSceneControlButtons` | Toolbar buttons |

### Document & DataModel Architecture

Always use `.update()`, never mutate directly:

```javascript
const actor = game.actors.get(actorId);
await actor.update({ name: "New Name" });
await actor.update({ "system.hp.value": 10 });
const newActor = await Actor.create({ name: "NPC", type: "character", system: {} });
```

### ApplicationV2 (v12+)

Prefer over legacy `Application`:

```typescript
import { ApplicationV2, FormApplication } from "foundry";

export class MySettings extends ApplicationV2 {
  static PARTS = { /* form parts */ };
}
```

### Canvas & Token Interaction

```javascript
const scene = game.scenes.viewed;
const tokens = canvas.tokens.controlled;
const token = tokens[0];
const actor = token?.actor;
await ChatMessage.create({ content: "Hello!", user: game.user.id });
```

### Compendium Packs

```javascript
const pack = game.packs.get("pf2e.spells-srd");
const documents = await pack.getIndex();
const doc = await pack.getDocument(documentId);
```

### Socket Listeners (v11+)

```javascript
// Register listener in module entry point
game.socket.on("module.my-event", (data) => { /* handle */ });
// Emit from any context
game.socket.emit("module.my-event", { key: "value" }, { scope: "world" });
```

### TypeScript Configuration

ES2024, strict, NodeNext modules:

```json
{
  "compilerOptions": {
    "target": "es2024",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "lib": ["ES2024", "DOM", "DOM.Iterable"]
  }
}
```

### Looking Up Types

Always check pf2e-types (<https://github.com/7H3LaughingMan/pf2e-types>) before web search. Key files: `src/global.d.ts` (namespace augmentations), `src/global-exports.d.ts` (class declarations), `src/global-functions.d.ts` (helpers). Install:

```json
{
  "devDependencies": {
    "@7h3laughingman/foundry-types": "~14.360.8",
    "@7h3laughingman/pf2e-types": "^8.0.2"
  }
}
```

## Pitfalls: never mutate docs directly (use `.update()`), guard GM ops with `game.user.isGM`, init in `Hooks.once("ready")`, module `name` must match folder, no server-side code (use `game.socket`), PF2e internals → use pf2e-system skill.
