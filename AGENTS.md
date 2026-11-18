# AGENTS.md

Declarative VS Code extension pack: no runtime code, no build step, no tests. The entire functional definition is the `extensionPack` array in `package.json`; everything else is Marketplace metadata and documentation.

## Commands

- Preview files that will ship: `npx --yes @vscode/vsce@4.1.1 ls`
- Build the VSIX: `npx --yes @vscode/vsce@4.1.1 package` → `vscode-wev-dev-extension-pack-<version>.vsix`
- Test locally: press F5 (`.vscode/launch.json` starts an Extension Development Host), or build and run `code --install-extension <vsix>`
- There is no `npm install`, test, lint, typecheck, or build command. The only workflow is publishing; no CI validates changes.

## Never change

- `package.json` `"name": "vscode-wev-dev-extension-pack"` ("wev", not "web") and `"publisher": "c0der-himel"`. This misspelled ID is the permanent, published Marketplace identity; renaming breaks existing installs and all README/Marketplace links.
- Do not add `main`, activation code, dependencies, or a bundler. Extension packs are declarative by design.

## Changing the pack

1. Edit `extensionPack` in `package.json` (IDs use `publisher.extension`).
2. Keep `README.md` in sync: the count "23 extensions" appears in ~10 places, plus the "What's Included" table and area groupings. The README is also the Marketplace listing, so edits are user-facing.
3. Add a `CHANGELOG.md` entry and bump `version`; `package-lock.json` has no dependencies but mirrors the version, so bump it too.
4. Only add maintained extensions that support VS Code `^1.76.2` (the low engine is deliberate for reach).
5. Re-run `vsce ls` after touching `.vscodeignore` or assets to confirm the shipped file list.

## Packaging gotchas

- `.vscodeignore` excludes `assets/*.svg` and dev files. The Marketplace rejects SVG in READMEs, so `assets/hero.svg` is the editable source and `assets/hero.png` is what renders and ships.
- The VSIX currently contains only `package.json`, `README.md`, `CHANGELOG.md`, `LICENSE`, `icon.png`, and `assets/hero.png`. When adding files, check `vsce ls`; keep dev-only files listed in `.vscodeignore`.
- `vsc-extension-quickstart.md` is untouched generator boilerplate, not a source of truth.

## Releasing

- Publish only via GitHub Actions: **Actions → Publish to Marketplace** (`.github/workflows/publish.yml`). It uses OIDC Trusted Publishing (`vsce publish --oidc`), so no PAT or secret is involved.
- Bump the version above the latest Marketplace release first; an already-published version cannot be republished.
- Push to `main`, then run the workflow from the latest commit. The workflow rejects stale runs, and re-running an old run will fail by design.
