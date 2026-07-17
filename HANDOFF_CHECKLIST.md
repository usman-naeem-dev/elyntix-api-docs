# Docs-site Handoff Checklist

This checklist explains how to split the `docs-site/` folder into a standalone GitHub repository, run it locally, deploy it to a custom domain (docs.<domain>), and keep it in sync with the `/v1` API.

1. Create the new GitHub repository

- Create a new empty repo on GitHub (e.g. `elyntix-api-docs`).
- On your local machine, from the repo root run:
  ```bash
  cd docs-site
  git init
  git remote add origin git@github.com:<org>/elyntix-api-docs.git
  git add .
  git commit -m "chore(docs): initial Mintlify site scaffold"
  git push -u origin main
  ```

2. Required files (must be present in the repo root)

- `docs.json` — Mintlify site config (navigation, branding, colors).
- `package.json` — dev scripts and build commands for local preview and CI.
- `docs/index.mdx` (Introduction) — high-level overview.
- `docs/quickstart.mdx` — quickstart walkthrough (create key, list agents, test call, read calls).
- `docs/authentication.mdx` — Bearer API key guidance and security notes.
- `docs/agents/*` — Agents API pages (GET, POST, GET/:id, PATCH, DELETE, test-call).
- `docs/calls/*` — Calls API pages (GET list, GET/:id, POST /:id/end).
- `docs/errors.mdx` — public error envelope and examples.
- `docs/rate-limits.mdx` — placeholder for rate limit policy.
- `public/*` — site logos, favicon, and other static assets.

3. How to run locally

- Prereqs: Node.js (LTS), pnpm or npm.
- Install dependencies:
  ```bash
  cd docs-site
  pnpm install
  ```
- Run preview:
  ```bash
  pnpm dev
  ```
- Open the local preview at `http://localhost:3000` (CLI will print URL).

4. How to deploy to docs.<domain> (Mintlify)

- Sign into Mintlify and choose "Create site" → Connect GitHub repository `elyntix-api-docs`.
- Configure build settings (defaults work for Mintlify CLI): build command: `mint build`, output: `.mintlify` (Mintlify handles this automatically).
- In Mintlify dashboard, add a custom domain: `docs.<domain>` and follow DNS instructions (CNAME/ALIAS record to Mintlify host).
- Enable automatic deploys on push to `main`.

5. How to update docs when `/v1` API changes

- Workflow:
  - Make the API change in the main repo and add/update an OpenAPI spec or API schema PR in the same repo when possible.
  - Update the docs in `docs-site/docs/` with the new endpoint, request/response examples, or versioned notes.
  - Open a PR from a docs branch in `elyntix-api-docs` and deploy the PR preview in Mintlify.
  - Merge when verified and allow Mintlify to publish to `docs.<domain>`.

6. Keeping examples synced with real request/response schemas

- Short term (manual): copy the real example payloads from the backend route mappers and schema files when authoring docs. Prefer the `map*ToResponse` helper outputs as canonical examples.
- Medium term (recommended): add a small CI step that extracts example responses from the orchestrator routes (or shared schema package) into `docs/openapi.json` or `docs/examples/` and fail the docs PR if examples are stale.
- Long term: maintain an OpenAPI spec for `/v1` and reference it from Mintlify to auto-generate the endpoint reference. (See note below.)

7. OpenAPI note

- Add an OpenAPI spec once the `/v1` surface stabilizes. Mintlify can ingest an OpenAPI file and generate endpoint pages automatically. Keep the OpenAPI in the docs repo at `openapi/v1.yaml`.

8. Dashboard integration note

- Do not add docs pages to the dashboard app. The dashboard should include a single external link pointing to `https://docs.<domain>` for developer documentation.

9. PR / CI recommendations

- Protect `main` branch and require PR reviews for documentation changes.
- Enable Mintlify preview deploys for PRs so authors can verify content visually.

10. Contact and ownership

- Add a `CODEOWNERS` entry in repo root pointing to the docs owners/team for reviews and alerting.

This checklist is part of the `docs-site/` scaffold in the monorepo. After moving the folder to a new GitHub repo, keep this file in the root of the docs repository.
