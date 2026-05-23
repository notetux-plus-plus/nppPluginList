# CLAUDE.md — nppPluginList

This repo is the **plugin registry** for Notetux++. It contains a single JSON file that the
Plugins Admin dialog (`pluginsadmin.c` in the main app) fetches over HTTPS to show users the
catalogue of installable plugins.

**This repo must always have valid, parseable JSON on `main`.**
A GitHub Actions workflow validates every PR before merge.

---

## File structure

```
nppPluginList/
├── v1/
│   └── notetux_plugin_list.json   ← the catalogue (array of plugin objects)
├── schemas/
│   └── plugin_entry.schema.json   ← JSON Schema for validation (informational)
└── .github/
    └── workflows/
        └── validate.yml           ← validates JSON on every PR
```

---

## JSON catalogue format

`v1/notetux_plugin_list.json` is a JSON array. Each element is a plugin object:

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

### Field reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Plugin identifier — must match the `.so` filename (e.g. `HelloPlugin` → `HelloPlugin.so`) |
| `displayName` | string | yes | Human-readable name shown in Plugins Admin |
| `version` | string | yes | SemVer string (`MAJOR.MINOR.PATCH`) |
| `author` | string | yes | Author name or GitHub username |
| `description` | string | yes | One-line description shown in Plugins Admin (max 120 chars) |
| `homepage` | string | yes | URL of the plugin's GitHub repo or docs page |
| `minAppVersion` | string | yes | Minimum notetux version required (SemVer) |
| `repository.download` | string | yes | Direct URL to the compiled `.so` file (GitHub Release asset preferred) |
| `repository.sha256` | string | yes | SHA-256 hex digest of the `.so` file (lowercase, 64 chars) |

### Invariants to enforce

- `name` must be unique across all entries.
- `version` must be valid SemVer.
- `repository.download` must be a valid HTTPS URL ending in `.so`.
- `repository.sha256` must be a 64-character lowercase hex string.
- The array must be sorted alphabetically by `name` (case-insensitive).
- No trailing commas, no comments (standard JSON only).

---

## How to add a new plugin

1. Build the release `.so` and upload it to the plugin's GitHub Releases.
2. Compute the SHA-256 of the `.so`:
   ```sh
   sha256sum HelloPlugin.so
   ```
3. Add a new object to `v1/notetux_plugin_list.json` in alphabetical position.
4. Open a PR — CI will validate the JSON and check the URL is reachable.
5. Merge to `main` after review.

## How to update a plugin version

1. Upload the new `.so` as a new GitHub Release asset in the plugin's own repo.
2. Recompute the SHA-256.
3. In `v1/notetux_plugin_list.json`, update `version`, `repository.download`, and
   `repository.sha256` for that plugin entry.
4. PR + merge.

## How the app fetches this list

`pluginsadmin.c` in notetux reads `g_prefs.plugin_list_url`. The default value (set in `prefs.c`)
should point to the raw GitHub URL of `v1/notetux_plugin_list.json` on the `main` branch:

```
https://raw.githubusercontent.com/notetux-plus-plus/nppPluginList/main/v1/notetux_plugin_list.json
```

The app downloads the JSON, parses it, and displays it in the Plugins Admin tree view.
Install copies the `.so` to `~/.config/notetux/plugins/<Name>/` after verifying the SHA-256.
