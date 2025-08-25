# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `@samrum/vite-plugin-web-extension`, a Vite plugin for generating cross-browser platform, ES module-based web extensions. It supports both Manifest V2 and V3, provides complete ES module support (including content scripts), and offers HMR (Hot Module Replacement) support for all manifest entry points.

## Development Commands

### Package Management

Uses pnpm as the package manager:

```bash
pnpm install
```

### Build Commands

- `pnpm build` - Build the plugin using unbuild
- `pnpm dev` - Development build with stub mode using unbuild
- `pnpm prepublishOnly` - Runs before publishing (executes build)

### Testing Commands

- `pnpm test` - Run tests in watch mode using vitest
- `pnpm test:run` - Run tests once using vitest
- Run single test: `pnpm test <test-file-pattern>`
- For specific manifest tests: `pnpm test test/manifest/`

### Code Quality

- `pnpm lint` - Format code using prettier
- `pnpm lint:check` - Check code formatting without making changes

### Release

- `pnpm release` - Create signed release using standard-version (runs tests and build first)

## Code Architecture

### Core Plugin Structure

The plugin is built around a factory pattern for handling different manifest versions:

- **`src/index.ts`** - Main plugin entry point, implements Vite plugin lifecycle
- **`src/manifestParser/`** - Core manifest processing logic
  - `manifestParserFactory.ts` - Creates appropriate parser for V2/V3
  - `manifestParser.ts` - Base abstract parser class
  - `manifestV2.ts` - Manifest V2 specific implementation
  - `manifestV3.ts` - Manifest V3 specific implementation

### Key Components

- **DevBuilder (`src/devBuilder/`)** - Handles development mode builds with separate implementations for V2/V3
- **Middleware (`src/middleware/`)** - Vite client modifications for extension support
- **Utils (`src/utils/`)** - Utility functions for file handling, Rollup integration, virtual modules, and Vite transformations

### Extension Architecture Concepts

1. **Content Script Wrapping**: Original content scripts are moved to `web_accessible_resources` and replaced with dynamic import wrappers to enable ES module support
2. **HMR Integration**: Custom HMR client modifications enable hot reloading for content scripts and shadow DOM styles
3. **Asset Handling**: Preserves `import.meta.url` instead of rewriting to `self.location` for proper content script asset handling

### Testing Strategy

Comprehensive snapshot testing in `test/manifest/` with separate test files for each feature across both manifest versions:

- Tests follow pattern: `featureName.v2.test.ts` and `featureName.v3.test.ts`
- Snapshots verify generated manifest output
- Resource files in `test/manifest/resources/` provide test fixtures

### TypeScript Configuration

- Uses `moduleResolution: "bundler"` for modern module resolution
- Targets ES2021 with strict TypeScript settings
- Type definitions in `types/index.d.ts` define plugin options interface

### Build System

- Uses `unbuild` for building both CJS and ESM outputs
- Exports both main plugin and client utilities
- Pre-commit hooks with husky and lint-staged ensure code quality

## Development Guidelines

When working with this codebase:

1. **Manifest Version Support**: Always consider both V2 and V3 when making changes to core functionality
2. **Testing**: Add corresponding snapshot tests for new features in both manifest versions
3. **Virtual Modules**: The plugin uses virtual modules for dynamic content - see `src/utils/virtualModule.ts`
4. **Development vs Build**: Separate logic paths exist for dev and build modes, especially in manifest parsers
5. **Web Accessible Resources**: Understanding how the plugin handles web accessible resources is crucial for content script functionality

## Plugin Options

Key configuration options when using the plugin:

- `manifest` - The base manifest object (required)
- `useDynamicUrlWebAccessibleResources` - Controls `use_dynamic_url` property (default: false for V3, not applicable for V2)
- `optimizeWebAccessibleResources` - Optimizes resource definitions on build (default: true)
- `additionalInputs` - Process additional scripts/HTML/styles as extension inputs
