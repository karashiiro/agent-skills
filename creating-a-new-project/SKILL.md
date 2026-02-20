---
name: creating-a-new-project
description: Use when creating a new project, repository, package, or application from scratch - ensures official scaffolding tools are used instead of hand-writing boilerplate.
---

# Creating a New Project

## Overview

Always search for and use official project generators or scaffolding tools before writing project boilerplate by hand. Generators encode community best practices, produce standardized structure, and stay current with ecosystem conventions.

## When to Use

- Creating a new application, library, package, or repository
- Setting up a new workspace or monorepo
- Adding a new package to an existing monorepo
- Bootstrapping any project where a `create-*`, `init`, or `new` command likely exists

## Core Pattern

**STOP before writing any config or boilerplate files.** Follow this order:

1. **Search** for the official scaffolding tool for the framework/language/platform
2. **Run** the generator with appropriate options (use non-interactive flags — see below)
3. **Customize** the generated output to fit the specific needs

Never hand-write files that a generator would produce (package.json, tsconfig.json, Cargo.toml, pyproject.toml, Dockerfile templates, CI configs, etc.) unless no generator exists.

## Handling Interactive Generators

Most generators prompt interactively by default. Since CLI agents cannot respond to interactive prompts, **always use non-interactive flags** to specify all options upfront.

**Strategy:**
1. Check the generator's `--help` output for non-interactive flags
2. Specify the template/preset and all required options via CLI flags
3. If no non-interactive mode exists, ask the user to run the generator themselves

### Non-Interactive Flags by Generator

| Generator | Key Flags | Example |
|-----------|-----------|---------|
| Vite | `--template`, `--no-interactive` | `npm create vite@latest my-app -- --template react-ts --no-interactive` |
| Next.js | `--yes`, `--ts`, `--app`, `--tailwind`, `--src-dir`, `--eslint` | `npx create-next-app@latest my-app --yes --ts --app` |
| Nuxt | `--template`, `--no-modules`, `--packageManager` | `npm create nuxt@latest my-app -- --no-modules --packageManager=pnpm` |
| SvelteKit | `--no-add-ons`, `--no-install`, `--add` | `npx sv create my-app --no-add-ons --no-install` |
| Astro | `-y` / `--yes`, `--template` | `npm create astro@latest my-app -- --template minimal -y` |
| React Router | `--template` (required) | `npx create-react-router@latest my-app --template remix-run/react-router-templates/default` |
| Expo | `--yes`, `--template` | `npm create expo my-app -- --yes --template blank` |
| Tauri | `--template` | `npm create tauri-app@latest my-app -- --template react` |
| Electron Forge | `--template` | `npx create-electron-app@latest my-app --template webpack` |
| Turborepo | `-m`, `--skip-install` | `npx create-turbo@latest my-monorepo -m pnpm` |
| Nx | `--interactive=false`, `--preset` | `npx create-nx-workspace@latest myorg --preset=empty --interactive=false --nxCloud skip` |
| Rust | Already non-interactive | `cargo new my-project` |
| Go | Already non-interactive | `go mod init github.com/user/my-module` |
| Python (uv) | Already non-interactive | `uv init my-project` |
| .NET | Already non-interactive | `dotnet new webapi -n MyProject` |

When unsure about flags, run `<generator-command> --help` first.

## Quick Reference

| Ecosystem | Generator Command | Notes |
|-----------|------------------|-------|
| Vite (React/Vue/Svelte/etc.) | `npm create vite@latest` | Covers most frontend frameworks |
| Next.js | `npx create-next-app@latest` | Includes App Router setup |
| Nuxt | `npm create nuxt@latest` | |
| SvelteKit | `npx sv create` | Unified CLI replacing old create-svelte |
| Astro | `npm create astro@latest` | |
| React Router (formerly Remix) | `npx create-react-router@latest` | Remix merged into React Router v7 |
| Node.js (plain) | `npm init` / `pnpm init` | Minimal, but sets up package.json correctly |
| Rust | `cargo init` / `cargo new` | |
| Go | `go mod init` | |
| Python | `uv init` or framework-specific (e.g. `django-admin startproject`) | |
| .NET | `dotnet new` | Templates: `webapi`, `console`, `classlib`, etc. |
| Java (Spring) | Spring Initializr (`start.spring.io`) | Web UI or IDE plugins; `spring init` CLI also available |
| Expo (React Native) | `npm create expo` | `npx create-expo-app` also still works |
| Tauri | `npm create tauri-app@latest` | |
| Electron | `npx create-electron-app@latest` | Via Electron Forge |
| Turborepo | `npx create-turbo@latest` | Monorepo setup |
| Nx | `npx create-nx-workspace@latest` | Monorepo setup |

This table is not exhaustive. If the ecosystem is not listed, **search the web** for the official generator before proceeding manually.

## Workflow

1. **Identify the ecosystem.** What language, framework, or platform?
2. **Search for the generator.** Check official docs or search the web for `create <framework>` / `<tool> init` / `<tool> new`.
3. **Check `--help`** for non-interactive flags before running.
4. **Run the generator non-interactively** with all options specified via flags. Prefer the latest version (`@latest`).
5. **Review the output.** Generators may include files the project doesn't need - remove those. They may also miss project-specific needs - add those.
6. **Adapt to existing conventions.** If adding to a monorepo, align the generated config with the repo's existing style (linter config, tsconfig paths, package manager, etc.).

## When Hand-Writing IS Appropriate

- No official generator exists for the ecosystem
- The project is intentionally minimal (e.g., a single-file script with no build tooling)
- The user explicitly requests writing from scratch
- The generator output would need so many modifications that starting fresh is faster (rare - usually customizing is still faster and safer)

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Running a generator without non-interactive flags | Hangs waiting for input | Always check `--help` and pass flags |
| Writing `tsconfig.json` by hand | Miss new compiler options, get strictness wrong | Use generator, then adjust |
| Hand-crafting `package.json` with scripts | Miss standard scripts, get module config wrong | `npm init` or framework generator |
| Guessing at `.gitignore` contents | Miss platform/framework-specific entries | Generator includes correct ignores; or use gitignore.io |
| Copying config from another project | Carries over project-specific quirks | Fresh generator output is cleaner |
| Skipping the search step | "I know how to set this up" - but conventions evolve | Always check for a generator, even for familiar ecosystems |
