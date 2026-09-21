# Public keys

Public keys only. No private key material is ever stored here.

Two key roles:

- **Chain root key** — signs each day's `merkle_root` (`root_key_id` in `root.json`). Carso-controlled.
- **Report signing key** — signs each report's PDF fingerprint (`key_id` in a `cdr` leaf), e.g.
  `carso-diligence-2026-08`. Also published at `https://diligence.carso.cloud/verify/`.

Keys are versioned and never overwritten. Each daily root and each report names the exact key that signed it, so
old records stay verifiable after a key rotation.

Files are named `<key_id>.pub.pem`.
