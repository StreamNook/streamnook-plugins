# Publishing a plugin

This repo is the index StreamNook reads to populate its in-app marketplace. To get a plugin listed, you open a pull request that adds an entry to `index.json`. This page is the full process.

## How trust works

Two signatures, two roles:

- **You sign your plugin file.** Each plugin is a zip, signed with your own key. That proves the file is yours and unaltered.
- **StreamNook signs the index.** We sign `index.json` with the project's operator key. That signature is what the app trusts, so a listing only goes live once we have reviewed and signed it.

The app verifies both on every install: our signature on the index, then your file's checksum and your signature, both pinned by the entry we signed. You cannot forge a listing (you do not have our key), and we never have to vouch for your binary's bytes (your signature and the checksum do that). The first time a user installs your plugin, the app pins your public key; every later update must be signed with the same key.

## 1. Build the plugin

A plugin is a native executable that speaks StreamNook's plugin protocol over stdio (`docs/plugins/PROTOCOL.md` in the app repo). Any language that compiles to a native binary works.

**Cross-platform:** ship one build per platform you support. A target is named `<os>-<arch>`, matching Rust's `std::env::consts`:

| Target | OS / arch |
|---|---|
| `windows-x86_64` | Windows, 64-bit |
| `macos-x86_64` | macOS, Intel |
| `macos-aarch64` | macOS, Apple Silicon |
| `linux-x86_64` | Linux, 64-bit |
| `linux-aarch64` | Linux, ARM64 |

The app installs the build matching the user's platform. If you ship none for a platform, the plugin shows there as unavailable rather than installing the wrong binary.

## 2. Package each build

One zip per platform target:

```
your-plugin-windows-x86_64.zip
├── plugin.toml      (manifest, at the root)
├── your-plugin.exe  (the entry named in plugin.toml; bare name on macOS/Linux)
└── assets/          (anything else it needs)
```

`plugin.toml` is per-platform (the `runtime.entry` filename differs across platforms), but `id`, `version`, and `tier` must be identical in every platform's zip and must match your index entry exactly. Manifest fields are in `docs/plugins/MANIFEST.md`.

Use your own id namespace (for example `community.yourname.plugin`). `app.streamnook.*` is reserved for first-party plugins and is rejected from third parties.

## 3. Sign each zip

Generate a [minisign](https://jedisct1.github.io/minisign/) keypair once and keep the secret key safe. Sign each platform zip:

```
minisign -Sm your-plugin-windows-x86_64.zip
```

That produces `your-plugin-windows-x86_64.zip.minisig`. Put the public key in your index entry.

## 4. Host the files

Host each zip and its `.minisig` yourself, normally as assets on a GitHub release in your own repo. Your index entry points at those URLs by `https`. Because the checksum lives in the index we sign, a self-hosted file still cannot be swapped after approval.

## 5. Add your entry to `index.json`

Add one object to the `plugins` array. `platforms` maps each target to its hosted, signed zip:

```json
{
  "id": "community.yourname.plugin",
  "name": "Your Plugin",
  "version": "1.0.0",
  "tier": "B",
  "description": "One concise line about what it does.",
  "homepage": "https://github.com/yourname/your-plugin",
  "host_min": "7.0.0",
  "author": { "name": "yourname", "pubkey": "RW...your minisign public key..." },
  "platforms": {
    "windows-x86_64": {
      "url": "https://github.com/yourname/your-plugin/releases/download/v1.0.0/your-plugin-windows-x86_64.zip",
      "sha256": "<sha256 hex of that zip>",
      "size": 123456,
      "signature_url": "https://github.com/yourname/your-plugin/releases/download/v1.0.0/your-plugin-windows-x86_64.zip.minisig"
    }
  },
  "icon_url": "https://raw.githubusercontent.com/yourname/your-plugin/main/icon.png",
  "readme_url": "https://raw.githubusercontent.com/yourname/your-plugin/main/README.md"
}
```

- `tier`: `A` standard, `B` extended, `C` advanced. Pick honestly; it must match your manifest. See the app's `docs/plugins/CAPABILITIES.md`.
- `icon_url`: a square PNG. `readme_url`: raw markdown rendered as your detail page (no HTML).
- Leave `official` out; it marks first-party plugins only.

Do not edit `index.json.minisig`. You cannot produce a valid one (only we hold the operator key); we regenerate it when we merge.

## 6. Open the pull request

Fill in the PR template. We then:

1. Download each platform zip, confirm the `sha256` matches, and verify your `.minisig` against the `pubkey` you declared.
2. Unpack and check the manifest: that `id`, `version`, and `tier` match the entry, that the namespace is yours, and that the capabilities it requests are honest for what it claims to do. Manifest and capability review is our floor.
3. Re-sign `index.json` on our side and merge. Your plugin appears in the app on its next index refresh.

## Updates

Bump `version`, publish new signed zips, and open a PR updating your entry. Sign with the same key. If you must change keys, list the old one in `author.previous_pubkeys` and provide a second signature by the old key (`<artifact>.minisig.prev`); see `docs/plugins/SIGNING.md`.
