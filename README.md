# NovaLuce

**A modular Electron framework with strict separation between core logic, IPC, and UI.**

NovaLuce grew out of a real desktop application and was extracted into a standalone, general-purpose framework once the underlying architecture proved itself worth sharing. It's opinionated on purpose: Electron apps tend to blur together window management, inter-process communication, and business logic until nobody can say where one ends and the next begins. NovaLuce draws hard lines between those concerns so your codebase stays predictable as it grows.

---

## Why NovaLuce

Most Electron starter kits give you a working app and let architecture happen organically. NovaLuce takes the opposite approach: it gives you a small set of enforced boundaries so that six months in, you can still answer "where does this belong?" without guessing.

- **Core logic never talks to IPC directly.** Business logic is portable, testable, and doesn't care whether it's running in a main process or a test harness.
- **IPC describes *what* happens, not *how*.** Window mechanics live in one place, not scattered across every handler.
- **Established abstractions replace raw Node.js calls.** File access, window control, and event tracking all go through framework-level modules instead of ad hoc `fs`/`path`/`BrowserWindow` calls.
- **Documentation and code are never allowed to drift.** Comments track code, not the other way around, and stale ("ghost") code is treated as a bug.

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

These rules aren't suggestions — they're what keeps the three layers honest:

1. **IPC files express *what* happens.** All `BrowserWindow` mechanics belong in the windowing module, not inline in IPC handlers.
2. **Core modules have zero IPC.** If a function needs to know about `ipcMain` or `ipcRenderer`, it doesn't belong in eCORE.
3. **Use framework abstractions over raw Node.js calls.** Prefer the provided `mod.dir`, `mod.window`, and similar helpers to calling `fs`, `path`, or `BrowserWindow` directly.
4. **Source code is the source of truth.** Comments describe intent; they never override what the code actually does.
5. **No Ghost Code.** Dead code paths and stale references get removed, not commented out and forgotten.
6. **Comments Must Track Code.** A comment that no longer matches its function is treated as a defect.

---

## Getting Started

```bash
npm install novaluce
```

```js
const { core, window, ipc } = require('novaluce');

// eCORE: pure logic, no IPC awareness
const data = core.dir.read('./config.json');

// eDepend: window lifecycle through framework abstractions
const win = window.create({ width: 1200, height: 800 });
win.onceLoaded(() => win.show());

// IPC stays declarative — describes what happens, not how
ipc.handle('config:get', () => data);
```

> Full setup instructions, a step-by-step walkthrough, and a JavaScript primer for contributors newer to async patterns are available in the docs.

---

## Documentation

Complete developer documentation — including a beginner-friendly guide and a full function reference — lives in [`How-To-Use.html`](./docs/How-To-Use.html).

- **Zero to Running** — quick-start steps, plus a deeper "Understand" walkthrough for those who want the *why*
- **Learn The Language** — a primer on the JS patterns the framework leans on (async/await, Promises, modules, pub/sub)
- **Function Reference** — every real function across the codebase

---

## Contributing

Contributions are welcome. Before opening a PR, please make sure changes respect the layer boundaries above — a good rule of thumb: if you're not sure which layer something belongs in, it probably needs to be split rather than placed in the most convenient file.

1. Fork the repo and create a feature branch
2. Keep core logic free of IPC/Electron dependencies
3. Update documentation alongside any code change ("Comments Must Track Code" applies to docs too)
4. Open a PR with a clear description of what changed and why

---

## License

This project is licensed under the **GNU General Public License v3.0 (GPLv3)**.
