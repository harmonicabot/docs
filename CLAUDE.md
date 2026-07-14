# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Harmonica documentation site built with [Mintlify](https://mintlify.com). Deployed at [help.harmonica.chat](https://help.harmonica.chat). Part of the Harmonica ecosystem.

## Commands

```bash
npx mint dev        # Local preview at http://localhost:3000
npx mint update     # Update Mintlify CLI
```

## Publishing

Auto-deploys to production on push to the default branch via the [Mintlify GitHub app](https://dashboard.mintlify.com/settings/organization/github-app).

**A broken build freezes the whole site silently.** Mintlify keeps serving the last successful build; any build failure (most often an invalid `api-reference/openapi.yaml`) fails with no error surfaced in the docs — pushed pages just never appear / 404. Classic trigger: an **inline YAML flow-mapping description with commas** — `'400': { description: Validation error (a, b, or c) }` parses the commas as key separators, so a word like `oversized` reads as an unexpected property and the spec fails OpenAPI validation. Quote the scalar: `{ description: 'Validation error (a, b, or c)' }`. Check locally with `npx mint broken-links` (the OpenAPI error prints at the top; broken *links* are warnings, not build failures). This froze the site 2026-06-19 → 07-14 (~4 weeks, 11 failed builds) until one quote fixed it (`e5a1de0`).

**The Mintlify MCP `query_docs_filesystem_harmonica` is a stale cached index of *published* docs** — not the repo, not the live site. To check "is X live?", use the live site (e.g. a cache-busted fetch) + git, not the MCP index.

## Structure

- `docs.json` — Site config: navigation, theme, logo, navbar, fonts
- `api-reference/openapi.yaml` — OpenAPI 3.1 spec for Harmonica REST API v1 (Mintlify auto-generates endpoint pages from this)
- `api-reference/introduction.mdx` — API overview (auth, base URL, key concepts)
- `api-reference/mcp-server.mdx` — MCP server setup and tools reference
- `guides/` — User guides (hosts, participants)
- `index.mdx` — Landing page
- `quickstart.mdx`, `faq.mdx` — Getting started content

## OpenAPI Spec

The `api-reference/openapi.yaml` is the **canonical API spec** for the docs site. Mintlify generates endpoint pages from it automatically. The endpoint navigation in `docs.json` uses the format `"METHOD /path"` (e.g., `"GET /sessions/{id}"`, `"PATCH /sessions/{id}"`).

A parallel copy exists at `harmonica-web-app/docs/api-spec.yaml` (OSS repo) — keep both in sync when updating API endpoints.

## Adding New Endpoints

1. Add the operation to `api-reference/openapi.yaml`
2. Add the navigation entry to `docs.json` under `"Endpoints"` group (format: `"METHOD /path"`)
3. If relevant, update `api-reference/mcp-server.mdx` tools table

## Content Format

Pages use MDX with Mintlify components (`<CodeGroup>`, `<Card>`, `<Tabs>`, etc.). Frontmatter requires `title` and `description`.
