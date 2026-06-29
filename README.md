# docs-review-test

Test monorepo demonstrating [docs-review](https://github.com/mcanas/docs-review) — a markdown documentation review tool with Confluence-like inline commenting, backed by GitHub Discussions and deployed via GitHub Pages.

## Projects

| Project | Docs Path |
|---|---|
| [Todo App](projects/todo-app/docs/) | `projects/todo-app/docs/` |
| [Weather App](projects/weather-app/docs/) | `projects/weather-app/docs/` |

Each project contains a full documentation suite: BRD → URS → HLD → LLD → API Spec → ADRs.

## Setup

See the [docs-review README](https://github.com/mcanas/docs-review) for full installation instructions. For this repo:

1. Create a GitHub OAuth App with the Pages URL as the callback URL
2. Add `DOCS_REVIEW_OAUTH_CLIENT_ID` as a repository secret
3. Enable GitHub Pages (source: `gh-pages` branch)
4. Push to `main` — the workflow deploys automatically
