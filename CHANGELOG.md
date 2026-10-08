# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project uses semantic versioning.

## [Unreleased]

## [0.5.1] - 2026-10-08

### Fixed

- Declare `typebox` as a `*` peer dependency instead of a regular dependency, matching Pi 1.0.0's host-provided module contract; the pinned copy remains a development dependency for local typechecking and tests.

## [0.5.0] - 2026-08-07

### Breaking Changes

- Replaced extension-owned `modelOverrides` with regex-based `modelApiOverrides`. Move metadata and compatibility overrides, including `sendSessionAffinityHeaders`, to Pi's `<agentDir>/models.json`.
- Removed `settings.sendSessionAffinityHeaders`; session-affinity behavior is now configured through Pi model or provider `compat` settings.

### Added

- Added the `/newapi-config-recover` prompt template to reconcile settings from legacy or malformed config backups, merge `models-generated.json` templates into Pi's `models.json`, confirm exact backup deletion, and guide the subsequent `/reload`.
- Added `/newapi-generate-models-json` to reload available catalogs and atomically write unknown-model override templates to `<agentDir>/models-generated.json` for manual merging into Pi's `models.json`.
- Added ordered regular-expression API routing through `modelApiOverrides`; the first matching rule is authoritative over advertised gateway endpoint metadata.

### Changed

- Require Pi v0.84.0 or newer and migrate dynamic catalog persistence to generation-checked `context.publish()`, preventing stale refreshes from overwriting newer model lists.
- Pin the Pi SDK packages at v0.84.0 as development dependencies so local typechecking and tests use the supported extension API.
- Refactored the extension into focused `src/` modules while retaining the root `index.ts` Pi entry point and stable startup display name.
- Split tests into focused suites under `test/` and kept internal model helpers private to the package implementation.
- Changed unknown-model defaults to a 128,000-token context window and 32,768 maximum output tokens.
- Pi now applies all user model metadata and compatibility patches after dynamic NewAPI model discovery.
- Moved extension configuration from `<agentDir>/extensions/` to `<agentDir>/extension-settings/`; unified path/schema detection through `getConfigVersion()` and added top-level schema `version: 1`, with automatic migration for existing configuration.
- Config JSON is now validated during deserialization with explicit TypeBox schemas for versions `0` and `1`, selected solely by the declared `version` field, with field-specific validation errors. Schema `1` treats `modelApiOverrides` as optional and an omitted value as no routing overrides. Legacy-path, schema `0`, and invalid configuration files are moved to timestamped `extension-settings/provider-newapi.YYMMDD-HHMMSS.json.bak` archives, and all successful archive messages direct users to `/newapi-config-recover`.
- Replaced the shared 3-second HTTP timeout with endpoint-specific limits: 15 seconds for model discovery, 10 seconds for optional ratio metadata, and 5 seconds for the add-provider reachability check.

### Removed

- Removed automatic unknown-model template persistence from `provider-newapi.json`; the extension never edits Pi's user-owned `models.json`.

## [0.4.0] - 2026-06-06

### Added

- Dynamic NewAPI catalog discovery through Pi v0.80.8 `refreshModels(context)`, including `/model` background refresh and forced `pi update --models` refresh support.
- Provider-scoped catalog persistence in Pi's `models-store.json`, offline catalog restoration, and abort-aware NewAPI requests.
- Atomic, serialized configuration updates so concurrently discovered unknown-model templates do not erase other provider entries.
- Deterministic unit tests for ratio matching, cost conversion, defensive response parsing, and model construction, plus `typecheck` and `test` npm scripts.
- Added `settings.sendSessionAffinityHeaders` to opt discovered OpenAI Chat Completions and Anthropic Messages models into Pi session-affinity headers. OpenAI Responses models retain their built-in cache-enabled session-affinity behavior.

### Changed

- Require `@earendil-works/pi-ai` and `@earendil-works/pi-coding-agent` v0.80.8 or newer.
- Providers now register immediately with an intentional empty catalog, making `/login <provider>` available before the first model discovery.
- `/newapi-provider-add` saves only the gateway configuration and directs users through Pi-owned `/login <provider>` credential entry.
- `/newapi-provider-list` now uses Pi's supported provider authentication status API.
- Successful model catalogs retain existing enrichment, cost calculation, API routing, configured overrides, and generated unknown-model templates.
- Model discovery now parses each entry's `owned_by` and `supported_endpoint_types`, and routes models (including generated unknown-model templates) to an API the gateway actually advertises, falling back to enriched/default APIs only when no advertised endpoint is usable.
- `modelOverrides` entries for known (enriched) models are now applied as partial patches: only the fields present in the JSON are overridden, and unspecified fields keep their built-in values.

### Removed

- Direct `auth.json` reads and all removed `modelRegistry.authStorage` access.
- API-key collection and direct credential mutation from `/newapi-provider-add`.
- Credential deletion from `/newapi-provider-remove`; run `/logout <provider>` before removing the extension configuration because Pi v0.80.8 does not expose extension-safe logout.

## [0.3.0] - 2026-06-05

### Added

- Added `api` to generated `modelInfo` templates so unknown models can explicitly choose a supported NewAPI backend.
- Added enriched model provider/source tracking for debugging model metadata provenance.
- Added VS Code launch configuration for running the extension through pi.

### Changed

- Route models by enriched built-in model API metadata instead of hard-coded model ID prefixes.
- Register discovered NewAPI models with their enriched `api`, endpoint, and compatibility settings so pi can use the appropriate built-in stream handler.
- Expanded enrichment sources to include `zai` and `xiaomi`, while preserving provider priority.
- Apply `supportsDeveloperRole` compatibility during enrichment: enabled for `anthropic` and `openai` sources, disabled for other sources.
- Updated English and Chinese documentation to describe API-based routing and enriched model metadata.

### Fixed

- Skip built-in models with APIs unsupported by NewAPI during enrichment lookup construction. NewAPI enrichment now only uses `openai-completions`, `openai-responses`, and `anthropic-messages` models.
- Fall back to `anthropic-messages` with a warning when an enriched model has no API value.

## [0.2.0] - 2026-06-05

### Added

- Added prioritized model enrichment from multiple built-in providers.
- Added config-vs-upstream diff warnings when a previously unknown model becomes known.
- Added `compat` passthrough from enriched built-in model metadata.
- Added a dedicated `ModelLookupItem` type for enrichment lookup entries.

### Changed

- Lowered fetch timeout to 3 seconds to avoid UI freezes during NewAPI discovery.
- Updated Chinese documentation link to use an absolute GitHub URL.

## [0.1.1] - 2026-06-05

### Changed

- Updated `package.json` keywords for pi package gallery discoverability.

## [0.1.0] - 2026-06-05

### Added

- Initial `pi-provider-newapi` extension for self-hosted NewAPI gateways.
- Dynamic model discovery from `GET /v1/models`.
- NewAPI ratio config fetching from `GET /api/ratio_config` with quota-to-cost calculation.
- Model metadata enrichment and generated templates for unknown models.
- Graceful unconfigured fallback provider when discovery or authentication fails.
- `fetchWithTimeout` helper for model discovery and ratio config requests.
- English and Chinese README documentation.
- MIT license and package metadata for pi extension installation.

### Fixed

- Corrected `/login` references and installation instructions.
- Added graceful fallback behavior on fetch failures.
