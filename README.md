# NovaLuce

**A modular Electron framework with strict separation between core logic, IPC, and UI.**

NovaLuce grew out of a real desktop application and was extracted into a standalone, general-purpose framework once the underlying architecture proved itself worth sharing. It's opinionated on purpose: Electron apps tend to blur together window management, inter-process communication, and business logic until nobody can say where one ends and the next begins. NovaLuce draws hard lines between those concerns so your codebase stays predictable as it grows.

---

## Why NovaLuce

Most Electron starter kits give you a working app and let architecture happen organically. NovaLuce takes the opposite approach: it gives you a small set of enforced boundaries so that six months in, you can still answer "where does this belong?" without guessing.

- **Core logic never talks to IPC directly.** Business logic is portable, testable, and doesn't care whether it's running in a main process or a test harness.
- **IPC describes *what* happens, not *how*.** Window mechanics live in one place, not scattered across every handler.
- **Established abstractions replace raw Node.js calls.** File access, window control, and event tracking all go through framework-level modules instead of ad hoc `fs`/`path`/`BrowserWindow` calls.

---

## Architecture

NovaLuce is split into three layers, each with a single responsibility:

```
┌─────────────┐
│  Interface  │  Renderer / UI layer
└──────┬──────┘
       │
┌──────┴──────┐
│   eDepend   │  Electron-specific IPC + preload bridge
└──────┬──────┘
       │
┌──────┴──────┐
│    eCORE    │  Framework core — zero IPC awareness
└─────────────┘
```

### `eCORE`
The framework's core modules. Pure logic, file/directory abstractions, event tracking, database helpers, i18n, and other foundational pieces. eCORE has **zero knowledge of IPC or Electron's process model** — it can be reasoned about and tested in isolation.

### `eDepend`
The Electron-specific layer: IPC channels, the preload bridge, and window lifecycle management. This is where `BrowserWindow` mechanics live (via `mod.window` methods like `onceLoaded()` and `setBounds()`), and where IPC handlers stay thin, delegating real work down to eCORE.

### `Interface`
The renderer/UI layer. Talks to the rest of the app only through the bridge exposed by `eDepend` — never directly to Node.js or Electron internals.

---

## Core Principles

These rules aren't suggestions — they're what keeps the three layers safe:

1. **IPC files express *what* happens.** All `BrowserWindow` mechanics belong in the windowing module, not inline in IPC handlers.
2. **Core modules have zero IPC.** If a function needs to know about `ipcMain` or `ipcRenderer`, it doesn't belong in eCORE.
3. **Use framework abstractions over raw Node.js calls.** Prefer the provided `mod.dir`, `mod.window`, and similar helpers to calling `fs`, `path`, or `BrowserWindow` directly.

---

## Getting Started

NovaLuce doesn't work like a typical npm package you `require()` into your own code — there's no import step and no manual wiring. It's convention-based: you drop a file in the right folder, and the framework's registry discovers, validates, and boots it automatically.

Application startup (`main.js`) does two things:

```js
await load();   // eCORE: scans lib/ and registers every module (mod.dir, mod.window, mod.ipc, ...)
await init();   // eCORE: boots each module in dependency order
await start();  // eDepend: scans eDepend/ipc/ and wires every *.ipc.js it finds
```

You never touch `main.js`. All app behavior lives in `eDepend/ipc/*.ipc.js` files — each one exports a `name`, its allowed IPC channels, optional payload `schemas`, and a `register(mod)` function. Drop a new file in that folder and it's live on next boot; no imports, no registration calls.

### Hello World

Create `eDepend/ipc/hello.ipc.js`:

```js
module.exports = {

    name: 'hello',

    allowedChannels: {
        ReceivingFromUI: ['hello:get-greeting'],
        SendingToUI:     [],
    },

    schemas: {
        'hello:get-greeting': null, // no payload expected
    },

    register(mod) {
        // mod.ipc, mod.window, mod.dir — every core module is already
        // loaded and handed to you here. Nothing to import.
        mod.ipc.RequestWithReply('hello:get-greeting', () => {
            return 'Hello from NovaLuce!';
        });
    },

};
```

From the renderer (`Interface/`), call it through `window.bridge` — the only thing exposed to renderer code, wired up by the preload layer:

```js
const greeting = await bridge.awaitReply('hello:get-greeting');
console.log(greeting); // "Hello from NovaLuce!"
```

That's the whole workflow: declare the channel, implement `register(mod)`, call it from the renderer through `bridge`. The framework handles discovery, security allowlisting, and payload validation for you.

---

## Documentation

Complete developer documentation — including a beginner-friendly guide and a full function reference — lives in How-To-Use.html.

- **Zero to Running** — quick-start steps, plus a deeper "Understand" walkthrough for those who want the *why*
- **Learn The Language** — a primer on the JS patterns the framework leans on (async/await, Promises, modules, pub/sub)
- **Function Reference** — every real function across the codebase

---

## License

This project is licensed under the **GNU General Public License v3.0 (GPLv3)**.
