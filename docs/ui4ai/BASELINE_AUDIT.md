# UI4AI — Baseline Source Audit

Audit date: 2026-11-08  
Repository: `kokoosabzi/Ui4`  
Audited baseline: `main` at `c547347b5cd83bafaff867fbc6e4916da23018dc`

## Scope and confidence

This is a static source/configuration review of the GitHub repository. No Windows machine, local Ollama instance, GPU telemetry, dependency installation, automated test run, or packaged application was executed as part of this audit. Runtime and build claims remain unverified until Phase 1 is run on Windows.

## Confirmed findings

### Application and build

- Root package is `aider-desk@0.91.0-dev`.
- The project uses Electron Vite, React 19, TypeScript, npm workspaces, Vitest, and Electron Builder.
- Existing scripts include `dev`, `typecheck`, `test`, `build`, and `build:win`.
- Windows packaging is configured through NSIS.
- Python connector source is packaged from `resources/connector`.
- The Python connector imports Aider's Python APIs directly; the connector environment/version compatibility must be checked against the lock/setup scripts.

### Important release-identity issue

The current `electron-builder.yml` still contains upstream product identity and publishing settings:

- `appId: eu.magiksoft.aiderdesk`
- `productName: aider-desk`
- Windows executable name: `aider-desk`
- GitHub publishing target: `magiksoft/aider-desk`

Before any UI4AI release, these must be changed to UI4AI-owned identity and release destination. Do not run a publish/release workflow while upstream publishing configuration remains unchanged. Keep this change isolated and review installer upgrade/data-directory implications before changing the application ID.

### Ollama integration

- `src/main/models/providers/ollama.ts` discovers installed models through `/api/tags`.
- The provider maps models to Aider with `ollama_chat/<model>`.
- `src/renderer/src/hooks/useOllamaModels.ts` independently fetches `/api/tags` for a renderer hook.
- Existing Ollama tests cover model discovery and context-length mapping, but do not establish runtime lifecycle management.
- No complete runtime manager, model pull workflow, running-model view, or hardware monitor was confirmed in the reviewed paths.

Implementation implication: consolidate new Ollama management operations behind the existing typed `ApplicationAPI` / main-process boundary. Avoid adding direct component-level HTTP calls or a parallel backend.

### Main/preload/browser API architecture

The architecture decision record specifies a shared `ApplicationAPI` contract in `packages/common/src/api.ts`, implemented through both Electron preload IPC and browser REST. Every new renderer-to-main operation should follow this contract, with matching implementations and tests where required.

### Privacy and security

- The telemetry ADR documents product telemetry as enabled by default and distinguishes it from separately configured LLM tracing/exporters. UI4AI must not claim a universal opt-out until both paths are handled.
- The Electron security ADR documents a trusted-but-exposed renderer posture and exceptions including disabled sandboxing and web security in the primary window. Review actual BrowserWindow configuration before making security changes; do not infer the full posture from one window.
- UI4AI should default product telemetry and remote tracing to off, remove upstream analytics identifiers from the product build, and inventory network calls before making an offline guarantee.

## Priority actions

1. **P0 — Build identity and publishing safety:** prepare UI4AI app ID, product name, executable name, icons, data directory strategy, update channel, and GitHub release target. Keep publishing disabled until validated.
2. **P0 — Windows baseline verification:** install dependencies on Windows; run typecheck, unit tests, locale validation, development launch, and `build:win`; record exact results.
3. **P1 — Runtime compatibility:** verify bundled Python connector, Aider version, Ollama endpoint discovery, and process shutdown behavior on a clean Windows environment.
4. **P1 — Ollama API service:** design typed main-process operations for health, tags, running models, model details, pull stream/progress/cancel, delete, and safe unload behavior.
5. **P1 — Hardware collector:** choose a Windows-first collection strategy with explicit NVIDIA support and graceful fallback for unsupported hardware.
6. **P1 — Privacy audit:** inventory product telemetry, LLM tracing, update checks, extension downloads, external links, and other network paths; implement and test offline mode.
7. **P2 — UI shell and localization:** add UI4AI branding, theme system, Persian/English locale files, RTL behavior, and dashboard navigation after baseline stability.

## Do not do yet

- Do not rewrite the application architecture or introduce FastAPI/SQLite as a parallel backend.
- Do not replace AiderDesk's existing model provider registry.
- Do not modify `main` directly.
- Do not enable automatic releases or publish artifacts to the upstream repository.
- Do not claim offline support based only on successful local-model inference.
