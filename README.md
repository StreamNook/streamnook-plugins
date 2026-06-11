# StreamNook Plugins

The community plugin index for [StreamNook](https://github.com/StreamNook). Plugins are opt-in add-ons that live outside the core app, so the app stays lean and you run only the ones you want.

## Add this source

In StreamNook, open Settings, then Plugins, then Sources, and add:

```
https://raw.githubusercontent.com/StreamNook/streamnook-plugins/main/index.json
```

The app pins this index's signing key the first time you add it and shows you its fingerprint. After that, every listing and update is verified against that key before any plugin code runs.

## How it works

- The index (`index.json`) is signed; each plugin's downloadable file is signed by its author. The app verifies both signatures and the file's checksum before installing.
- Plugin files are attached to this repository's [Releases](https://github.com/StreamNook/streamnook-plugins/releases). The index points at them.
- Each plugin's detail page in the app is rendered from its `README.md` under `plugins/`.

## Available plugins

| Plugin | Tier | What it does |
|---|---|---|
| [Drops and Points Farmer](plugins/drops-farmer/README.md) | Advanced | Mines drops and farms channel points in the background. |

## Tiers

Standard and Extended plugins are lighter add-ons. Advanced plugins run background automation on your account and are distributed here as community add-ons rather than as part of the built-in first-party set.
