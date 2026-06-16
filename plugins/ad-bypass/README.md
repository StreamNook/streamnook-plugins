# Ad-Free Playback

A StreamNook plugin that resolves live streams through community playlist proxies so they play without stitched ads. It runs as a separate program that StreamNook starts and talks to, and it ships its own in-app settings panel (a hybrid plugin: a sidecar plus a native UI module).

## What it does

Streams you are already entitled to watch ad-free (Twitch Turbo, or a subscription to that channel) play directly and never touch this plugin. For everything else:

- When you start a stream, StreamNook invokes this plugin's `playback.resolve` hook. The plugin races a pool of community playlist proxies in its own process, over its own networking, and answers with the winning master playlist for the app's relay to serve.
- Anonymous proxy masters top out at 1080p, so when you are signed in the plugin merges the 1440p+ tiers from your own master back in (the splice).
- If an ad leaks through mid-stream, the plugin detects it in its own process (polling the playlist it resolved) and re-resolves through a different region, swapping the relay's upstream via `set_upstream`. The core app never scans for ads.

If every proxy is down the plugin declines, and StreamNook falls back to its own direct resolution (ads may appear). No login token is needed; the plugin requests no credentials.

## Settings panel

The panel on the Plugins page is the plugin's own UI, not a generic form:

- **Resolution:** toggle proxy resolution on or off, choose a preferred region (or Automatic, which races every region), and toggle the 1440p+ splice.
- **Proxy health checker:** a "Check all" probe times every proxy in the pool for reachability and latency, shown per row with a health dot, region tag, and color-coded latency. The fastest healthy proxy is flagged, the one in use right now is marked live, and any proxy can be pinned so it is raced first.
- **Custom proxies:** an add-and-remove list of your own proxy base URLs, tried before the bundled pool.

The panel drives the sidecar over the plugin action bridge; the sidecar does the actual probing and resolving in its own process.

## Building

The sidecar and the UI module build separately.

```
# sidecar (from this directory)
cargo build --release

# UI panel bundle (from the repository root)
node plugins/ad-bypass/ui/build.mjs
```

The packaged artifact carries the release exe and `ui/dist/main.js` (as `main.js`) alongside the manifest.
