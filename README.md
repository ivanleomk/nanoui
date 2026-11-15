# nanoui (Offline / Internal Fork)

A zero-dependency, offline-first fork of [justrach/nanoui](https://github.com/justrach/nanoui) for building fast desktop and mobile applications in Zig—designed for local-first internal tools and **Obsidian-style knowledge/Markdown workspaces**.

## Hardening & Audit Summary

Compared to upstream `justrach/nanoui`, this fork removes all outbound networking and self-update machinery:

- **Zero external dependencies:** `build.zig.zon` has `.dependencies = .{}` (`http-zig` removed).
- **No HTTP client:** `src/http*.zig` and `nanoui.http` are removed.
- **No auto-updater:** `src/update*.zig`, `nanoui.Updater`, `NANOUI_UPDATE_FEED`, and `NANOUI_RESTORE` are removed.
- **No telemetry or analytics:** Confirmed via full source audit; idle apps do not poll or make background network connections.
- **OS-native rendering & webviews:** Uses Metal/CoreText (macOS/iOS), Direct3D 11/DirectWrite (Windows), and OpenGL/CPU + GTK/WebKitGTK (Linux) without bundling Chromium or Electron.

> **Windows WebView2 note:** If you use `Page` / `Window.dockPage` on Windows and the target machine lacks the WebView2 runtime, [`window_notice.zig`](src/windows/window_notice.zig) shows a fallback dialog with a button linking to Microsoft's WebView2 download page. Native-UI-only apps do not use WebView2.

---

## Building an Obsidian-Style App on `nanoui`

`nanoui` provides the core primitives needed for a local-first Markdown/notes workspace out of the box:

| Capability | `nanoui` Module / API | Notes |
| --- | --- | --- |
| **Split panes & tab groups** | `nanoui.ui.dock` | Resizable split trees, tab bars, drag-and-drop pane docking. |
| **Text & note editing** | `nanoui.ui.textarea`, `textfield`, `textedit` | Multi-line editing, undo/redo, IME composition, Unicode grapheme/word selection. |
| **UI kit & command palette** | `nanoui.kit` | Buttons, overlays, modals, tabs, forms, toasts, and theme tokens (light/dark). |
| **Custom graph / canvas** | `nanoui.ui.canvas` | GPU-accelerated 2D drawing for node graphs, backlink maps, and canvases. |
| **Local vault I/O & watching** | `std.fs`, `nanoui.fdwatch`, `nanoui.dialog` | Native folder/file pickers and event-loop-integrated file/process watching. |
| **Local state & settings** | `nanoui.kv` | Lightweight local key-value persistence (`NSUserDefaults` on Apple, file-backed elsewhere). |
| **Optional WebView preview** | `Window.dockPage`, `App.bind`, `nanoui.Event` | Dock a system WebView alongside native panes with typed Zig ↔ JS/TS IPC. |
| **Headless UI testing** | `nui snapshot`, `nanoui.ui.Tester` | Offscreen PNG snapshots and scripted interaction tests (`click:`, `type:`, etc.) without opening a window. |

### Security Guidance for Hybrid (WebView) Panes

If your app only uses **Native UI**, no HTML/JS runtime is loaded at all. If you embed a WebView (`Page` / `dockPage`):

1. **Serve local assets only:** Use `PageOptions.assets` so requests resolve strictly against compile-time/in-memory assets on `nanoui://localhost` (`https://nanoui.local` on Windows).
2. **Validate IPC origin:** If a WebView can ever navigate to external URLs, check `call.origin()` inside every `App.bind` service method and reject untrusted origins.

---

## Get Started

Requires **Zig 0.17.0** (plus Xcode/macOS SDK on macOS, or GTK 3 at runtime on Linux).

```sh
git clone https://github.com/ivanleomk/nanoui
cd nanoui
zig build run-gallery    # native UI component tour
zig build run-hello      # webview + typed IPC example
zig build test           # unit and widget tests
```

### Scaffolding a New App with `nui`

```sh
zig build nui
./zig-out/bin/nui start my-vault --local .
cd my-vault
../zig-out/bin/nui test
../zig-out/bin/nui snapshot --out shot.png
../zig-out/bin/nui run
```

## License

MIT — see [LICENSE](LICENSE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
