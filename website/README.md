# Website

This website is built using [Docusaurus](https://docusaurus.io/), a modern static website generator.

> **Reading the docs on GitHub?** The Markdown under `docs/` is authored for the rendered site at
> <https://hermes-agent.nousresearch.com/docs/>. Cross-page links are relative Markdown paths, so they
> follow through on GitHub's file viewer too. Every page on the site has an **Edit this page** link
> that opens the source file here.

## Authoring links in `docs/`

- Link to another page with a relative Markdown path, anchors included:

  ```markdown
  [Profiles](../user-guide/profiles.md)
  [Bundles](../user-guide/features/skills.md#skill-bundles)
  ```

  Docusaurus turns the file path into the page route; GitHub follows the same path. Site routes
  (`/user-guide/profiles`, `/docs/user-guide/profiles`) only work on the rendered site — GitHub
  resolves them as repository paths and 404s, and the `/docs/` form also emits
  `/docs/zh-Hans/docs/...` 404s in the zh-Hans build because `baseUrl` is already `/docs/`.
- `python3 website/scripts/check_doc_links.py` fails on any route-style link in hand-authored pages
  (EN and the zh-Hans mirror); `--fix` rewrites them. It runs in the `Docs Site Checks` workflow.
  Generated pages (`user-guide/skills/{bundled,optional}`, `reference/*skills-catalog.md`) are
  produced by `scripts/generate-skill-docs.py`, which emits the same relative form.
- Pin `{#anchor}` on cross-linked headings so the zh-Hans mirror keeps the same id.

## Install and preview

Run website commands from `website/`. The checked-in `package-lock.json` and
CI use **npm**, not Yarn. The workspace requires Node.js >=20 and npm >=11.17;
`.npmrc` enforces engines and the dependency release-age policy. The docs CI
currently uses Node.js 26 and npm 12.

```bash
cd website
npm ci
npm start
```

Install Python 3 and PyYAML for the generated catalogs and LLM documentation
(`python3 -m pip install pyyaml` in your development virtual environment).
`npm start` and `npm run build` invoke `scripts/prebuild.mjs` automatically. It
extracts skills, automation blueprints and plugin metadata and creates the LLM
documentation files. It can also fetch catalog data from the live docs site.
Read prebuild warnings: a running preview with an empty fallback catalog does
not prove the generated data was complete.

## Build and verify

```bash
# From website/: full English + zh-Hans site, then a static preview
npm run build
npm run serve

# English-only build used by Docs Site Checks
npm run build:fast
```

`build:fast` has no `prebuild:fast` npm hook. On a fresh checkout, run
`node scripts/prebuild.mjs` before it, or perform the explicit catalog extraction
used in [Docs Site Checks](../.github/workflows/docs-site-checks.yml).

From the **repository root**, check hand-authored cross-page links and regenerate
the committed per-skill pages when skill content changes:

```bash
python3 website/scripts/check_doc_links.py
python3 website/scripts/generate-skill-docs.py
git diff -- website/docs website/sidebars.ts website/i18n
```

Review and commit generated changes together with their skill source. Do not
hand-edit the generated skill pages/catalogs. The site build writes `website/build/`.
Use `npm run typecheck` from `website/` for TypeScript validation.

## Publishing and forks

The [deployment workflow](../.github/workflows/deploy-site.yml) is the repository's
publishing path; the generic Docusaurus `npm run deploy` command is not a substitute
for its catalog generation and bilingual build steps.

The checked-in Docusaurus config still targets `NousResearch/hermes-agent` and
the upstream documentation domain. A fork can preview locally. Before publishing
it, review the URL, `/docs/` base URL, organization/project settings, deployment
permissions and catalog sources for the fork.

## Diagram Linting

CI runs `ascii-guard` to lint docs for ASCII box diagrams. Use Mermaid (````mermaid`) or plain lists/tables instead of ASCII boxes to avoid CI failures.
