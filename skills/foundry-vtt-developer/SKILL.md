---
name: foundry-vtt-developer
description: >
  Develop modules, macros, and code for Foundry VTT v14. Use this skill whenever
  the user asks to write, debug, review, or explain Foundry VTT JavaScript/TypeScript
  code — including module.json manifests, ES module entry points, Hooks registration
  (game.ready, render*, createDocument, updateDocument), Document/DataModel patterns,
  ApplicationV2 UI, canvas interactions, chat messages, compendium packs, socket
  listeners, or any Foundry-specific API. Also triggers for Foundry macro scripts
  (JavaScript snippets run from the hotbar). If the user mentions "Foundry",
  "FoundryVTT", "module.json", "game.ready", "Hooks.callOnce", "DocumentSheet",
  "Compendium", "TokenHUD", "canvas.tokens", or any Foundry-specific class name,
  invoke this skill. Do NOT use this for PF2e system internals — use the pf2e-system
  skill for that instead.
---

# Foundry VTT Developer (v14)

Write modules, macros, and client-side code for Foundry Virtual Tabletop v14. All code runs in the browser context.

## Authoritative References

| Resource | URL |
|---|---|
| Foundry VTT API docs (v14) | https://foundryvtt.com/api/ |
| Foundry VTT GitHub | https://github.com/foundryvtt/foundryvtt |
| Module development guide | https://foundryvtt.com/article/module-development/ |

When unsure about an API surface, search the official docs — do not guess at class names or method signatures.

## Core Concepts

### Package Types

- **Modules** (`module.json`): Add-ons that extend Foundry functionality. Loaded via `esmodules` or `scripts` arrays in the manifest.
- **Macros**: JavaScript snippets executed in the Foundry client context. No manifest needed — code runs directly from the Macro editor.

### Module Manifest (`module.json`)

Every module needs a `module.json` at its root. Minimum viable:

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

Key rules:
- `name` must match the folder name exactly — mismatch causes silent load failures
- Use `esmodules` for ES module entry points (preferred); use `scripts` only for legacy non-module JS
- `compatibility.verified` should be the latest patch you tested against

### Hook System

Foundry's event system is hook-based. Register callbacks with `Hooks.on`, `Hooks.once`, or `Hooks.callAll`. Common hooks:

| Hook | When |
|---|---|
| `load` | Before world data loads; earliest client hook |
| `setup` | After world data loads, before ready |
| `ready` | Everything initialized; safe to access game data |
| `renderApplication` / `renderApplicationV2` | When an application renders |
| `createDocument` / `updateDocument` / `deleteDocument` | Document lifecycle events |
| `getSceneControlButtons` | Customize scene toolbar buttons |
| `hotbarDrop` | Handle drag-and-drop to hotbar |

Use `Hooks.once("ready")` for one-time initialization. Use `Hooks.on` for recurring event handlers. Never access game data before the `ready` hook fires.

### Document & DataModel Architecture

Foundry uses a Document/DataModel split:

- **Document** — handles persistence, permissions, and database operations
- **DataModel** — holds the actual data with validation and defaults

Access patterns:
```javascript
// Get a document
const actor = game.actors.get(actorId);
const character = await Actor.fromId(actorId);

// Read data through direct properties
actor.name;
actor.system.hp.value;

// Modify — always use update(), never mutate directly
await actor.update({ name: "New Name" });
await actor.update({ "system.hp.value": 10 });

// Create
const newActor = await Actor.create({ name: "NPC", type: "character", system: {} });
```

### ApplicationV2 (v12+)

Foundry v12+ introduced ApplicationV2 as the modern UI framework. Prefer it over legacy Application when building new UI:

```typescript
import { ApplicationV2, FormApplication } from "foundry";

export class MySettings extends ApplicationV2 {
  static PARTS = { /* form parts */ };
}
```

### Canvas & Token Interaction

```javascript
// Current scene and tokens
const scene = game.scenes.viewed;
const tokens = canvas.tokens.controlled;
const token = tokens[0];
const actor = token?.actor;

// Create a chat message
await ChatMessage.create({ content: "Hello!", user: game.user.id });
```

### Compendium Packs

Access via `game.packs`:
```javascript
const pack = game.packs.get("pf2e.spells-srd");
const documents = await pack.getIndex();
const doc = await pack.getDocument(documentId);
```

### Socket Listeners (v11+)

For server-client communication:
```javascript
// Register listener in module entry point
game.socket.on("module.my-event", (data) => { /* handle */ });

// Emit from any context
game.socket.emit("module.my-event", { key: "value" }, { scope: "world" });
```

### TypeScript Configuration

Target ES2024 with strict mode. Recommended tsconfig:

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

## Common Pitfalls

1. **Never mutate Document data directly** — always use `.update()` or `.create()`
2. **Check `game.user.isGM` / `game.user.isActiveGM`** before running GM-only operations
3. **Use `Hooks.once("ready")` for initialization** — accessing game data before ready will fail
4. **Module `name` must match folder name** — mismatch causes silent load failures
5. **No server-side code** — Foundry modules run client-side only (with socket-based server communication via `game.socket`)
6. **PF2e system internals are out of scope** — for PF2e-specific APIs, data structures, or the Rule Element system, use the pf2e-system skill instead
