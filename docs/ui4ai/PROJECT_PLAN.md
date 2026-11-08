# UI4AI — Project Plan

## Product direction

UI4AI is a Windows-first, local-first AI development control center built by directly forking AiderDesk. The upstream project remains the application foundation; UI4AI-specific capabilities are added incrementally without creating a parallel backend or replacing working AiderDesk subsystems unnecessarily.

## Repository strategy

- Upstream: https://github.com/magiksoft/aider-desk
- Origin: https://github.com/kokoosabzi/Ui4
- `main`: stable, releasable UI4AI baseline
- `develop`: integration branch for accepted work
- Feature branches: focused changes merged into `develop`
- Keep the upstream remote and periodically review upstream changes before syncing.

Do not make UI4AI product changes directly on `main`.

## Confirmed baseline observations

1. The repository is an Apache-2.0 fork of AiderDesk.
2. The application is an Electron multi-process desktop application with React/TypeScript renderer, Node.js main process, typed preload bridge, and a Python connector for Aider integration.
3. The existing Ollama provider adapter discovers installed models through Ollama's `/api/tags` endpoint, creates Ollama model clients, and maps models to Aider using `ollama_chat/<model>`.
4. AiderDesk already has project/task management, Git worktrees, agent orchestration, model-provider adapters, REST APIs, MCP, and extension infrastructure. Prefer extending these facilities over duplicating them.
5. The upstream telemetry ADR records product telemetry as enabled by default, with additional environment-configured LLM tracing paths. UI4AI must audit and change these defaults and disclosures before claiming offline/privacy-first operation.
6. The upstream Electron security ADR records security exceptions in the primary window, including `sandbox: false` and `webSecurity: false`. UI4AI must review these boundaries and must not expand privileged renderer access casually.

These are source-review findings, not a claim that the application has been built or runtime-tested in this environment.

## Product principles

- Windows-first; no Linux compatibility requirement is part of the initial scope.
- Local-first and offline-capable for core workflows.
- No separate FastAPI backend unless a concrete requirement cannot be met cleanly in the existing architecture.
- n8n is optional and minimal; core features must not depend on it.
- Custom UI/UX built on the existing React renderer.
- Persian and English UI, including correct RTL layout for Persian.
- Telemetry and external network access disabled by default in the UI4AI product configuration.
- Preserve AiderDesk's review/approval controls and make destructive actions explicit.

## Target modules

### 1. AI Workspace
Retain and refine AiderDesk project, task, agent, context, Git, diff-review, and model-routing workflows.

### 2. Ollama Control Center
Add runtime status and lifecycle controls, endpoint health, installed model inventory, model details, pull/download with progress and cancellation, unload/delete operations where supported, running-model visibility, and safe configuration.

### 3. Hardware Center
Provide live CPU, RAM, GPU, and VRAM telemetry. Start with reliable Windows/NVIDIA support and isolate provider-specific collectors so additional hardware can be added later.

### 4. Model Routing
Allow task profiles to select suitable local models based on task type, context capacity, availability, and hardware constraints. Keep manual override available.

### 5. Workflow Center
Expose a small set of useful local workflows and optional n8n integration. n8n must remain nonessential.

### 6. Privacy and Offline Center
Provide a visible offline/privacy status, disable product analytics and remote tracing by default, explain any feature that requires network access, and prevent accidental external calls in core workflows.

## Delivery phases

| Phase | Scope | Exit criteria |
|---|---|---|
| 0 | Repository baseline and development workflow | Fork verified; `develop` branch created; upstream/origin strategy documented |
| 1 | Build and runtime audit | Windows install/build, typecheck, unit tests, and baseline launch results recorded |
| 2 | UI4AI identity and shell | Product branding, navigation, responsive dashboard, theme system, Persian/English and RTL foundations |
| 3 | Ollama Control Center | Runtime health, model inventory/details, pull progress, delete/unload and error handling |
| 4 | Hardware Center | Reliable CPU/RAM/GPU/VRAM readings with refresh and graceful unsupported-device behavior |
| 5 | Model routing and profiles | Task-to-model profiles, capability metadata, context and resource-aware suggestions |
| 6 | Privacy/offline hardening | Telemetry default-off, tracing audit, network-call inventory, offline acceptance tests |
| 7 | Workflow integration | Optional n8n/MCP workflows without coupling core operation to external services |
| 8 | Release readiness | Windows installer, clean-machine test, upgrade/data-preservation test, documentation and release checklist |

## Engineering rules

- Read the relevant `AGENTS.md`, architecture decision records, and local conventions before changing a subsystem.
- Make small, reviewable feature branches; avoid broad rewrites.
- Add tests alongside behavior changes and update documentation/ADRs when architecture or privacy semantics change.
- Keep process execution and privileged filesystem operations in the main process behind the typed preload/API boundary.
- Validate all renderer and REST inputs at trusted boundaries.
- Never log prompts, source contents, credentials, tokens, or raw tool inputs/outputs.
- Do not claim a phase complete until its exit criteria have been tested and recorded.
- Preserve upstream attribution, Apache-2.0 notices, and third-party license obligations.

## Immediate next actions

1. Inspect Windows build prerequisites and scripts, Python connector setup, and the exact Electron process/security configuration.
2. Run baseline install, typecheck, test, and Windows build in a local Windows environment; record actual results rather than inferred results.
3. Map all Ollama-related UI, provider, settings, and process-management code before implementing runtime controls.
4. Implement the first Ollama health/status slice with tests before adding download and destructive model operations.
