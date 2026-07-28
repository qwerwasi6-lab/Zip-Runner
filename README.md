# Zip Game Runner

A single HTML file that lets you run browser games straight from a `.zip` — no local server, no `localhost`, no setup. Drop in a zip, and it runs the game in your browser using blob/data URIs.

This exists as a drop-in replacement for "just spin up a localhost server to test my game." Instead, you open one HTML file and load a zip.

## How it works

1. You pick or drag in a `.zip` file.
2. The runner unpacks it in-browser (via JSZip) and looks for an HTML entry point.
3. Every file in the zip is converted into a `data:` URI, and all relative paths (`<script src>`, `fetch()`, `XMLHttpRequest`, CSS `url()`, dynamic `import()`, etc.) are rewritten to point at those URIs.
4. The rewritten HTML is loaded into a sandboxed `<iframe>` and run as the game.

Games are saved to a local library (via IndexedDB) so you can reload them later without re-uploading.

## Features

- **Zip-only input.** Only `.zip` files are accepted (both by file picker and drag-and-drop). `.tar.gz` / other archive formats are **not** supported.
- **Static HTML/JS games** — the common case: an `index.html` plus JS/CSS/assets, run as-is.
- **React / Vite / TypeScript source projects** are also supported and tested — including React + Three.js projects. If the runner detects a source project (e.g. bare imports like `import React from "react"`, or `.ts`/`.tsx`/`.jsx` entry files), it bundles it in-browser using `esbuild-wasm` before running it, resolving both relative imports and bare package imports (pulled from a CDN). If bundling fails, it falls back to trying the zip as a static file set.
- **Asset support**: images, JSON, CSS, standalone glTF models (`.glb`, `.gltf`, and related `.bin`/texture files), and audio/video formats including `.mp3`, `.wav`, `.ogg`, `.mp4`, `.webm`, `.webp`, `.gif`, `.svg` are recognized and mapped to data URIs. Runtime `fetch()`/XHR requests—including the `Request` objects used by Three.js loaders—are redirected to the corresponding file from the zip.
- **Resolution control**: choose "fit to window" or a fixed internal resolution (independent of how large the game appears on screen) up to 7680×4320.
- **GPU / WebGL settings**: power preference, antialiasing, canvas alpha, preserve drawing buffer, and fail-on-low-perf-caveat — applied automatically to `getContext('webgl'/'webgl2')` calls inside the game.
- **Toggles**: mute audio, show FPS overlay, pixelated (nearest-neighbor) canvas scaling, and disable pause-on-blur.
- **Fullscreen support**, including in games that use the Fullscreen API themselves.
- **Persistent game library**: loaded zips are stored locally (IndexedDB) and appear in a sidebar for quick relaunching.
- **Debug console**: captures `console.log/warn/error`, uncaught errors, and unhandled promise rejections from both the runner and the game iframe, so you don't need to rely on browser devtools alone.
- Encrypted zip entries are rejected (not supported).

## What's untested / uncertain

- **WebAssembly (`.wasm`)** — not tested. May or may not work depending on how the game loads it.
- **Compressed glTF models** — ordinary standalone `.glb`/`.gltf` files are mapped from the zip, but models using Draco, Meshopt, or KTX2/Basis compression still depend on the game configuring the corresponding browser decoder. Decoder files must either be available in the zip or reachable over the network.
- Other frameworks with a similar Vite-based/source-project setup may work through the same esbuild path (React + Vite + Three.js is confirmed working), but this hasn't been broadly verified beyond that.

## Limitations

- This is an early-development build. It's estimated to work for roughly **90% of zipped browser games**, but odd bundler setups, non-relative asset loading tricks, or unusual build outputs may break it.
- Only `.zip` archives are supported — no `.tar.gz`, `.rar`, etc.
- Encrypted/password-protected zip entries won't load.

## Disclaimer

Games you test with this tool remain your property. This tool takes no responsibility for what you do with it or the content you run through it.
