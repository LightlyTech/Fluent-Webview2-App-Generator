---
name: fluent-webview2-app
description: |
  Generates desktop applications that follow the Windows 11 Fluent design language and render their interface with WebView2 (the Microsoft Edge Chromium engine). The host software framework is unconstrained: Python/pywebview, WPF, WinForms, WinUI 3, Electron, Tauri, C++/Win32, Rust/wry, and others can all serve as the shell, while the UI is uniformly injected into WebView2 as HTML/CSS/JS (either as inline strings or as embedded resources). Use this when the user mentions "Windows 11 Fluent desktop app", "WebView2 interface", "WPF/WinForms/WinUI/Electron/Tauri/pywebview Fluent UI", or "embedding HTML as the interface inside a desktop window". Always output complete, runnable code, and choose the correct string boundaries for the host language to prevent unterminated triple quotes or quotes.
---


# Windows 11 Fluent + WebView2 Desktop App Generator

## When to Use

Enable this skill when the user requests any of the following:

- A desktop application must present the **Windows 11 Fluent Design** visual language
- The interface is rendered with **WebView2** (Microsoft Edge / Chromium) rather than opened in a browser
- The host software framework is specified by the user and may be Python/pywebview, WPF, WinForms, WinUI 3, Electron, Tauri, C++/Win32, Rust/wry, etc.
- HTML/CSS/JS is distributed with the host program as inline strings or embedded resources
- Light/dark theme switching is needed, following the Windows system theme

## Technology Stack Definition

| Dimension | Constraint |
| --- | --- |
| Visual language | Windows 11 Fluent Design (see the tokens below) |
| Rendering engine | WebView2 (Evergreen Runtime, Chromium) |
| Host framework | Specified by the user; must not be replaced with a browser or another rendering engine without authorization |
| UI carrier | HTML + CSS + JavaScript |
| Bridge | WebView2 native messaging channel, or an equivalent mechanism provided by the host framework |

## Framework Adaptation Matrix

Before generating, confirm the host framework specified by the user and integrate it according to the table below. WebView2's **rendering behavior is completely identical**; the differences lie only in window creation, UI injection method, and bridge API.

| Host framework | WebView2 integration | UI injection | Bridge mechanism |
| --- | --- | --- | --- |
| Python / pywebview | `webview.create_window(html=...)` | HTML string inline | `js_api=` + `window.pywebview.api.xxx()`, wait for `pywebviewready` |
| WPF (.NET 6+) | `Microsoft.Web.WebView2.Wpf.WebView2` control | `NavigateToString` or `SetVirtualHostNameToFolderMapping` for local resources | `CoreWebView2.WebMessageReceived` + `PostWebMessageAsJson`; or `AddHostObjectToScript` |
| WinForms (.NET 6+) | `Microsoft.Web.WebView2.WinForms.WebView2` control | Same as WPF | Same as WPF |
| WinUI 3 / Windows App SDK | `Microsoft.UI.Xaml.Controls.WebView2` (or `BlazorWebView`) | `NavigateToString` / local resources | `CoreWebView2.WebMessageReceived` |
| Electron | `BrowserWindow.loadURL` / `loadFile` | `loadURL('data:text/html,...')` or local files | `preload` + `contextBridge` + `ipcRenderer` |
| Tauri v2 | `WebviewWindowBuilder` | `dist/` directory or the `html` parameter | `window.__TAURI__.invoke` + Rust command |
| C++ / Win32 | `ICoreWebView2` (COM) | `NavigateToString` | `WebMessageReceived` event |
| Rust / wry | `wry::WebViewBuilder::with_html` | Inline HTML | `ipc_handler` |

Generation rules:

1. **The user has specified a framework**: generate a minimal runnable project for that framework.
2. **The user has not specified one**: ask once; if the user explicitly asks for "quick validation", default to **Python + pywebview** (fewest dependencies); if the goal is native Windows integration, default to **WPF**.
3. **Never** replace WebView2 with `WebBrowser` (IE engine), CEF, or a direct browser call.

## Hard Output Requirements

1. Always output **complete, directly buildable and runnable** code; multi-file frameworks such as WPF/Electron/Tauri must include every necessary file (`.csproj`, `package.json`, `Cargo.toml`, entry file, UI file, etc.).
2. `...`, `omitted`, `the rest is the same as above`, `TODO`, and `see above` are forbidden.
3. A comment at the top of each file must state the environment dependencies:
   - Python: `pip install pywebview`, plus the WebView2 Runtime download address
   - .NET: SDK version, NuGet package (`Microsoft.Web.WebView2`)
   - Electron: `npm install`, `electron` version
   - Tauri: `cargo tauri dev`, Rust version
   - C++: MSVC version, Windows SDK
4. All string boundaries must be closed in pairs, and the correct raw string syntax must be chosen for the language:

   | Language | Recommended syntax |
   | --- | --- |
   | Python | `"""..."""`; do not use `str.format()` on HTML containing CSS `{}`; use `str.replace()` placeholders instead |
   | C# | `"""..."""` (raw string literal, C# 11+) or `@"..."` (escape inner `"` as `""`) |
   | JS/TS | Template literals; backticks or `${}` inside must be escaped, or use `String.raw` |
   | Rust | `r#"..."#` |
   | C++ | `R"html(...)html"` |

5. All HTML tags, `<style>`, and `<script>` must be closed.
6. Write functional comments only, not tutorial comments.
7. Curly braces in CSS must not be interpreted by any language's formatting function.

## Fluent Design Specification

### Color Tokens

Light:
- Background `#F3F3F3`
- Card `rgba(255,255,255,0.72)`
- Primary text `#1A1A1A`, secondary `#616161`, tertiary `#8A8A8A`
- Stroke `rgba(0,0,0,0.06)`, strong stroke `rgba(0,0,0,0.11)`
- Accent `#0067C0`, accent text `#FFFFFF`
- Hover `rgba(0,0,0,0.037)`, pressed `rgba(0,0,0,0.074)`

Dark:
- Background `#202020`
- Card `rgba(255,255,255,0.045)`
- Primary text `#FFFFFF`, secondary `#C8C8C8`, tertiary `#9A9A9A`
- Stroke `rgba(255,255,255,0.06)`, strong stroke `rgba(255,255,255,0.11)`
- Accent `#60CDFF`, accent text `#00334D`
- Hover `rgba(255,255,255,0.055)`, pressed `rgba(255,255,255,0.09)`

All colors must go through CSS variables, and theme switching only changes `html[data-theme]`.

### Typography

```css
font-family: "Segoe UI Variable Text", "Segoe UI",
             "Microsoft YaHei UI", "Microsoft YaHei",
             system-ui, -apple-system, sans-serif;
```

Use `"Cascadia Code", "Consolas", monospace` for monospaced content.

### Sizing and Corner Radius

- Navigation item height 38px, corner radius 5px
- Button height 32px, corner radius 4px
- Card corner radius 8px, padding 16px 18px
- Switch 40×20, corner radius 10px, thumb 14px
- Page title 28px / 600, body 14px / 400, auxiliary 12px

### Motion

- General easing `cubic-bezier(0.2, 0, 0, 1)`
- Theme switch `transition: background-color .25s ease, color .25s ease`
- Hover / pressed 0.1s ~ 0.16s
- Page transition fade-in and shift-up 0.24s

### Shadows and Glass

- Card shadow `0 1px 2px rgba(0,0,0,0.04), 0 2px 8px rgba(0,0,0,0.045)`
- Overlay shadow `0 8px 28px rgba(0,0,0,0.16)`, deepened in dark mode
- Translucent solid colors to simulate acrylic

### Interaction Details

- Text selection is disabled by default (`user-select: none`), except for input areas and information values
- Close button hover color `#C42B1C`
- Focus state uses a stroke rather than an outer glow

### Backend Responsibilities

Implement functionality in HTML wherever possible, and use the framework language to fill in what cannot be implemented.

### Default Window Configuration

- Displayed centered on screen, default 800*600
- The window must have a progressive animation on startup
- Title bar (WPF: extend Fluent into the title bar and make it draggable; others: use the native title bar)

## Self-Check List (Check Each Item Before Output)

- [ ] All `"""` are closed in pairs
- [ ] Colors go through CSS variables, and theme switching has a transition

## Packaging Configuration

After generating the code, ask the user whether to package:

| Framework | Command |
| --- | --- |
| Python | `pyinstaller --onefile --windowed --name AppName app.py` |
| WPF / WinForms | `dotnet publish -c Release -r win-x64 --self-contained` |
| WinUI 3 | `dotnet publish` (MSIX or unpackaged + Windows App SDK Runtime) |
| Electron | `electron-builder --win nsis` |
| Tauri | `cargo tauri build` |
| C++ | MSIX packaging or `.exe` + WebView2 Loader |
