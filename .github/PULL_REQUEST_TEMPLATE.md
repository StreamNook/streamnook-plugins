<!-- Adding or updating a plugin in the StreamNook index. See CONTRIBUTING.md. -->

## Plugin

- Name:
- Id (your own namespace, not `app.streamnook.*`):
- Version:
- Tier (A / B / C):
- What it does (one line):
- Platforms included:

## Checklist

- [ ] Each platform's zip is hosted and signed; its `.minisig` is reachable at the `signature_url`.
- [ ] My minisign public key is in `author.pubkey`, and I hold the secret key.
- [ ] Every `platforms` entry has the correct `url`, `sha256`, `size`, and `signature_url`.
- [ ] `id`, `version`, and `tier` are identical in every platform's `plugin.toml` and match this entry.
- [ ] The tier is honest and the requested capabilities are only what the plugin needs.
- [ ] `icon_url` is a square PNG and `readme_url` is raw markdown (no HTML).
- [ ] I did not touch `index.json.minisig` (the maintainers re-sign on merge).
- [ ] Update only: signed with the same author key, or a key rotation proof is included.
