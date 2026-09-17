# Ad-Free Playback

A StreamNook plugin that resolves live streams from somewhere Twitch does not
serve ads to, so they play without stitched ads. It runs as a separate program
that StreamNook starts and talks to, and it ships its own in-app settings panel
(a hybrid plugin: a sidecar plus a native UI module).

## What it does

Streams you are already entitled to watch ad-free (Twitch Turbo, or a
subscription to that channel) play directly and never touch this plugin. For
everything else:

- When you start a stream, StreamNook invokes this plugin's `playback.resolve`
  hook. The plugin works down a chain of sources in its own process, over its
  own networking, and answers with a master playlist for the app's relay to
  serve.
- Anonymous masters top out at 1080p, so when you are signed in the plugin
  merges the higher tiers from your own master back in (the splice).
- If an ad leaks through mid-stream, the plugin detects it in its own process
  and re-resolves through another source, swapping the relay's upstream via
  `set_upstream`. The core app never scans for ads.

If every source fails the plugin declines, and StreamNook falls back to its own
direct resolution (ads may appear). No login token is needed; the plugin
requests no credentials.

## The two kinds of source

| | What it is | Who runs it |
| --- | --- | --- |
| **v1 relay** | A server that answers `/playlist/{channel}.m3u8` with the playlist itself | Shared community addresses ship with the plugin. `luminous-ttv` to self-host |
| **v2 proxy** | An ordinary HTTP forward proxy that only carries the requests | You rent or run it. Squid is enough |

A relay does the work for you. A proxy only carries traffic, so this machine
does the work and the region shift happens here. Both end up at the same place:
a master playlist fetched from a country Twitch leaves alone.

With a proxy, the plugin keeps the media-playlist reloads on the same route as
the original fetch, because Twitch re-decides the ad splice on every reload.
Video segments are never proxied, so a metered proxy costs almost nothing to
use.

See [SELF_HOSTING.md](SELF_HOSTING.md) for a step-by-step guide to running
either one, including which countries work.

## Settings panel

The panel on the Plugins page is the plugin's own UI, not a generic form. Two
cards:

- **Settings:** switch ad-free playback on or off, choose a preferred region for
  the relays, toggle the splice, and order your own proxies against the shared
  relays. Every explanation sits behind a help glyph rather than on the page.
- **Sources:** everything the plugin can use, with a health dot, where it comes
  out, and its verdict. One field adds a new source, with a switch picking
  whether it is a v1 relay or a v2 proxy; hovering either half lists the formats
  it accepts.

Relays and proxies are checked differently, because "working" means different
things for each. A relay is probed for reachability and ranked on latency. A
proxy is probed by actually fetching through it: the row reports whether the
tunnel opened, whether the credentials were taken, **which country it comes out
in**, whether it stays there, and whether Twitch accepts it. Proxies are
deliberately excluded from the "Fastest" badge, because the nearest proxy is
always the quickest and the least useful.

The panel drives the sidecar over the plugin action bridge; the sidecar does the
actual probing and resolving in its own process.

## Credentials

A bought proxy comes with a username and password. They are kept in separate
fields and handed to `reqwest::Proxy::basic_auth`, so no `user:pass@host` URL is
ever built: such a URL breaks silently on a password containing `@ : / # %`, and
it is the kind of value that ends up in a log line. Every log line the plugin
emits passes through one scrubbing chokepoint, `Debug` on the proxy type is
hand-written so it cannot print the password, and a test drives a real failed
request through a credentialed proxy to prove nothing carries it.

They are stored unencrypted in the panel's own storage, which the panel says
plainly. Encrypting them would need a plugin-facing secret store that does not
exist yet.

## Building

The sidecar and the UI module build separately.

```bash
# sidecar (from this directory)
cargo build --release

# UI panel bundle (from the repository root)
node plugins/ad-bypass/ui/build.mjs
```

The packaged artifact carries the release exe and `ui/dist/main.js` (as
`main.js`) alongside the manifest.

## Tests

```bash
cargo test
```

One test is ignored by default because it needs a real proxy. It reads the proxy
from the environment, so no credential is ever committed:

```bash
STREAMNOOK_TEST_PROXY='host:port:user:pass' \
STREAMNOOK_TEST_CHANNEL='someone_live' \
  cargo test live_proxy -- --ignored --nocapture
```

It reports each stage separately: tunnel, exit country, stickiness, whether
Twitch mints a token, the resolved quality ladder, and whether a media-playlist
reload goes through the proxy.
