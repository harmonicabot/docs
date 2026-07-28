# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Harmonica documentation site built with [Mintlify](https://mintlify.com). Deployed at [help.harmonica.chat](https://help.harmonica.chat). Part of the Harmonica ecosystem.

## Commands

```bash
npx mint dev             # Local preview at http://localhost:3000
npx mint broken-links    # Validates openapi.yaml — the only reason to run it (see Publishing)
npx mint update          # Update Mintlify CLI
```

## Publishing

Auto-deploys to production on push to the default branch via the [Mintlify GitHub app](https://dashboard.mintlify.com/settings/organization/github-app).

**A broken build freezes the whole site silently.** Mintlify keeps serving the last successful build; any build failure (most often an invalid `api-reference/openapi.yaml`) fails with no error surfaced in the docs — pushed pages just never appear / 404. Classic trigger: an **inline YAML flow-mapping description with commas** — `'400': { description: Validation error (a, b, or c) }` parses the commas as key separators, so a word like `oversized` reads as an unexpected property and the spec fails OpenAPI validation. Quote the scalar: `{ description: 'Validation error (a, b, or c)' }`. Check locally with `npx mint broken-links` (the OpenAPI error prints at the top). This froze the site 2026-06-19 → 07-14 (~4 weeks, 11 failed builds) until one quote fixed it (`e5a1de0`).

**That OpenAPI error is the only useful thing `mint broken-links` reports.** Its link output is noise: as of 2026-07-28 it flags 77 broken links across 17 files and **all 77 are false positives.** It compares links written `/guides/hosts/chains` (correct, and the live URL) against `docs.json` entries written `guides/hosts/chains`, without normalising the leading slash, so every absolute internal link fails. Fragment-only links (`#anchor`) are unaffected. Treat the non-zero exit and the long link list as normal; read only the top of the output.

**It also cannot find dead pages.** A page absent from `docs.json` navigation is still built and publicly served. Eleven Mintlify scaffold files sat live for ~6 months this way, and only 3 ever appeared in any check — the 3 that happened to contain a broken link. All were removed 2026-07-28 (`bf2355a`, `d156248`, `a0ebbef`, `b17e90f`, and the commit adding this note), retiring `essentials/`, `snippets/` and `ai-tools/`. HAR-1436 tracks a replacement check that compares `.mdx` files on disk against the nav tree; note it must skip `"METHOD /path"` nav entries, which Mintlify generates from `openapi.yaml` rather than from files.

**The Mintlify MCP `query_docs_filesystem_harmonica` is a stale cached index of *published* docs** — not the repo, not the live site. To check "is X live?", use the live site (e.g. a cache-busted fetch) + git, not the MCP index.

### Verifying a deploy

Mintlify reports a `Mintlify Deployment` check run per commit:

```bash
gh api repos/harmonicabot/docs/commits/<sha>/check-runs --jq '.check_runs[] | "\(.name): \(.status) \(.conclusion)"'
```

`completed success` means the build is healthy, which is what separates a real silent freeze from ordinary lag. **The CDN then trails by several minutes**, so a changed page can still serve stale HTML — or a deleted page still return 200 with its old metadata — after a successful deploy. To read what actually shipped, hit the raw markdown endpoint, which bypasses the rendered-page cache: `https://help.harmonica.chat/<path>.md`.

## Structure

- `docs.json` — Site config: navigation, theme, logo, navbar, fonts
- `api-reference/openapi.yaml` — OpenAPI 3.1 spec for Harmonica REST API v1 (Mintlify auto-generates endpoint pages from this)
- `api-reference/introduction.mdx` — API overview (auth, base URL, key concepts)
- `api-reference/mcp-server.mdx` — MCP server setup and tools reference
- `guides/` — User guides (hosts, participants)
- `index.mdx` — Landing page
- `quickstart.mdx`, `faq.mdx` — Getting started content
- `privacy.mdx` — Privacy policy
- `custom.css` — Site CSS overrides, wired via `docs.json` → `custom.css`

## OpenAPI Spec

The `api-reference/openapi.yaml` is the **canonical API spec** for the docs site. Mintlify generates endpoint pages from it automatically. The endpoint navigation in `docs.json` uses the format `"METHOD /path"` (e.g., `"GET /sessions/{id}"`, `"PATCH /sessions/{id}"`).

A parallel spec exists at `harmonica-web-app/docs/api-spec.yaml` (OSS repo). It is **OSS-core only** — it does NOT carry Pro-only endpoints (grounding, context-sources, scratchpad, knowledge). Sync only endpoints that exist in both; never add a Pro-only endpoint there.

## Adding New Endpoints

1. Add the operation to `api-reference/openapi.yaml`
2. Add the navigation entry to `docs.json` under `"Endpoints"` group (format: `"METHOD /path"`)
3. If relevant, update `api-reference/mcp-server.mdx` tools table

## Content Format

Pages use MDX with Mintlify components (`<CodeGroup>`, `<Card>`, `<Tabs>`, etc.). Frontmatter requires `title` and `description`.
