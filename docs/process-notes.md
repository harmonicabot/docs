# Process notes

Session-by-session decisions for the Harmonica docs site. Append-only, newest last.

## 2026-08-19 — widget_spec, chain outcome, and one unpublished phantom
- **Done:** Documented `widget_spec` on `ChatResponse` (#3) and the `chain` bootstrap outcome on `SessionCreated` (#4), both companions to Pro contract changes. Unpublished `GET /sessions/public` (#5).
- **Decisions:** The unpublish was a PR rather than a direct push on purpose — removing a documented endpoint is a visible change to a public surface, so the merge is the decision. `PublicSessionListItem` and the `RateLimited` response were deliberately kept so restoring it is a paste plus one nav line; that instruction lives on HAR-93.
- **Why it mattered:** the endpoint was not merely in the spec file. It rendered live at `/api-reference/public/list-public-sessions` with an interactive playground and `security: []`, while returning 401 in production identically to a nonsense path — so the error an integrator actually hit was not even a documented outcome. Its audience is community integrations trying it cold, with no auth, exactly as instructed.
- **State:** All three merged; every Mintlify build reported success, so no silent freeze. This repo is now the mirror — Pro's `docs/api-spec.yaml` is canonical as of HAR-1598.
- **Note for future edits:** every description here is block style (`>-`). An inline flow-mapping description with commas is what froze this site for four weeks in 2026-06.
- **Next:** none outstanding.

## 2026-08-27 — distribution enum mirrored
- **Done:** `api-reference/openapi.yaml` `DistributionTarget.channel` widened from `enum: [telegram]` to the reserved vocabulary (telegram, community-admin, slack, discord, buzz), mirroring Pro's canonical spec for HAR-1600 (`00bd015`). Description states that the enum is a namespace, not a delivery promise, and that non-dispatching values are rejected at write time.
- **Decisions:** none — Pro is canonical, this repo mirrors.
- **State:** In sync with Pro. Parity is now machine-enforced from Pro's CI across operations, schema names, property sets and enum values, so silent divergence is no longer possible.
- **Next:** none. Note `raw.githubusercontent.com` trails a push by minutes; a parity red straight after a mirror push is CDN lag, verifiable via the GitHub API.
