# Elyntix AI API Docs

Standalone Mintlify documentation site for the public `/v1` API.

## What this repo is for

- Developer-facing docs for the public API
- Independent GitHub repo and deployment target
- Custom domain deployment at `docs.<domain>`

## Local development

1. Install dependencies with `pnpm install`.
2. Run `pnpm dev`.
3. Open the local Mintlify preview at the URL shown by the CLI.

## Deployment

1. Push this folder to its own GitHub repository.
2. Connect the repository in Mintlify at `https://mintlify.com/start`.
3. Set the custom domain to `docs.<domain>` in the Mintlify dashboard.
4. Keep the dashboard app separate; this repo should only contain documentation.

## Content coverage

- Introduction
- Quickstart
- Authentication
- Agents API
- Calls API
- Errors
- Rate limits
