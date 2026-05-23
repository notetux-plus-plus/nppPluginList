# nppPluginList

The official plugin registry for [Notetux++](https://github.com/notetux-plus-plus/notetux-plus-plus).

## Why this repo exists

Notetux++ ships a **Plugins Admin** dialog that lets users browse, install, and update plugins
without leaving the editor. To do that, it needs a machine-readable catalogue it can fetch over
HTTPS. This repo is that catalogue: a single JSON file (`v1/notetux_plugin_list.json`) that
the app downloads and renders as a browsable list.

Keeping the catalogue in its own repo means:
- Plugin authors can submit entries via a normal pull request.
- CI validates every change before it lands, so the app never sees broken JSON.
- The catalogue is versioned and auditable independently of the app or individual plugins.

## How plugins are installed

When a user clicks **Install** in Plugins Admin, Notetux++:

1. Reads the `repository.download` URL for that plugin from this catalogue.
2. Downloads the compiled `.so` file.
3. Verifies its SHA-256 against `repository.sha256` — installation is aborted if they do not match.
4. Copies the `.so` to `~/.config/notetux/plugins/<Name>/<Name>.so`.
5. Prompts the user to restart the editor so the plugin is loaded.

System-wide installations land in `/usr/lib/notetux/plugins/<Name>/<Name>.so` instead.

## Catalogue format

`v1/notetux_plugin_list.json` is a JSON array sorted alphabetically by `name`. Each element:

```json
{
  "name": "HelloPlugin",
  "displayName": "Hello Plugin",
  "version": "1.0.0",
  "author": "Andrea",
  "description": "A minimal example plugin for Notetux++.",
  "homepage": "https://github.com/notetux-plus-plus/hello_world",
  "minAppVersion": "1.0.0",
  "repository": {
    "download": "https://github.com/notetux-plus-plus/hello_world/releases/download/v1.0.0/HelloPlugin.so",
    "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  }
}
```

| Field | Required | Description |
|---|---|---|
| `name` | yes | Plugin identifier — must match the `.so` filename |
| `displayName` | yes | Human-readable name shown in Plugins Admin |
| `version` | yes | SemVer (`MAJOR.MINOR.PATCH`) |
| `author` | yes | Author name or GitHub username |
| `description` | yes | One-line description, max 120 characters |
| `homepage` | yes | URL of the plugin's repo or docs page |
| `minAppVersion` | yes | Minimum Notetux++ version required (SemVer) |
| `repository.download` | yes | HTTPS URL to the compiled `.so` release asset |
| `repository.sha256` | yes | SHA-256 hex digest of the `.so` (lowercase, 64 chars) |

## Adding a new plugin

1. Build the `.so` and upload it as a GitHub Release asset in the plugin's own repo.
2. Compute the SHA-256:
   ```sh
   sha256sum YourPlugin.so
   ```
3. Add a new entry to `v1/notetux_plugin_list.json` in alphabetical order by `name`.
4. Open a pull request — CI validates the JSON and checks that the download URL is reachable.
5. Merge to `main`; the plugin appears in Plugins Admin on the next catalogue refresh.

## Updating a plugin version

1. Upload the new `.so` as a new GitHub Release asset in the plugin's repo.
2. Recompute the SHA-256.
3. Update `version`, `repository.download`, and `repository.sha256` in `v1/notetux_plugin_list.json`.
4. Open a PR and merge.

## Validation rules

Every PR is checked by a GitHub Actions workflow. It enforces:

- Valid JSON with no trailing commas or comments.
- All required fields present and correctly typed.
- `name` is unique across entries.
- `version` and `minAppVersion` are valid SemVer strings.
- `repository.download` is an HTTPS URL ending in `.so`.
- `repository.sha256` is a 64-character lowercase hex string.
- Entries are sorted alphabetically by `name` (case-insensitive).

**The `main` branch must always contain valid, parseable JSON.**

## Repository layout

```
nppPluginList/
├── v1/
│   └── notetux_plugin_list.json   ← the catalogue
├── schemas/
│   └── plugin_entry.schema.json   ← JSON Schema (informational)
└── .github/
    └── workflows/
        └── validate.yml           ← CI validation on every PR
```

## Related repos

| Repo | Purpose |
|---|---|
| [notetux-plus-plus](https://github.com/notetux-plus-plus/notetux-plus-plus) | Main application |
| [hello_world](https://github.com/notetux-plus-plus/hello_world) | Reference plugin implementation |
