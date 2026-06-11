# StreamNook Plugins

The community plugin index for [StreamNook](https://github.com/StreamNook). Add it as a source and the app reads it directly, so approved plugins show up in the marketplace ready to install.

## Add this source

In StreamNook, open Settings, then Plugins, then Sources, and add:

```
https://raw.githubusercontent.com/StreamNook/streamnook-plugins/main/index.json
```

The app pins this index's signing key the first time you add it and shows you its fingerprint. After that, every listing and update is verified against that key before any plugin code runs.

## How it works

- The index (`index.json`) is signed by StreamNook; each plugin's downloadable file is signed by its author. The app verifies both signatures and the file's checksum before installing.
- First-party plugin files are attached to this repository's [Releases](https://github.com/StreamNook/streamnook-plugins/releases); third-party authors host their own. The index points at whichever by URL and checksum.
- Each plugin's detail page in the app is rendered from its `README.md` under `plugins/`.

## Publishing a plugin

Anyone can submit a plugin by pull request. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full process: build a signed package per platform, host it, add your entry to `index.json`, and open a PR. The maintainers review the manifest and capabilities, then re-sign and merge.

## Available plugins

| Plugin | Tier | What it does |
|---|---|---|
| [Autopilot](plugins/drops-farmer/README.md) | Advanced | Earn Twitch drops and channel points in the background. |

## Tiers

Standard and Extended plugins are lighter add-ons. Advanced plugins run background automation on your account and are distributed here as community add-ons rather than as part of the built-in first-party set.
