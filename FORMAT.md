# Daily record format

Each day is one directory: `days/<YYYY-MM-DD>/`, containing `root.json` and the timestamp tokens it names.

## `root.json` (genesis example — the real 2026-09-21 record)

```json
{
  "chain": "Carso Diligence Lookback Chain",
  "seq": 1,
  "date": "2026-09-21",
  "prev_root": "",
  "merkle_root": "6c694a034dd1741fed5936b37f51112175b0fdb0cce8edc97c51adad053bdf56",
  "leaf_count": 1,
  "hash_alg": "sha256",
  "merkle_rule": "rfc6962-v2",
  "root_key_id": "diligence-chain-root-2026-09",
  "root_signature": "<base64 Ed25519 over the RAW 32-byte merkle_root digest>",
  "timestamps": [
    { "tsa": "freetsa",  "token": "root.freetsa.tsr" },
    { "tsa": "digicert", "token": "root.digicert.tsr" }
  ],
  "leaves": [
    { "type": "source-snapshot", "hash": "6c694a03…053bdf56", "proof": [] }
  ]
}
```

At genesis `prev_root` is `""` and `seq` is 1. On every later day `prev_root` is the previous day's
`merkle_root`, forming the chain link.

## Leaf shapes

- **`cdr`** — an issued Carso Diligence Report, published **fingerprint-only** (non-enumerable — no report_id and
  no signature in the public leaf):
  ```json
  { "type": "cdr", "pdf_sha256": "<hex>", "json_sha256": "<hex>", "proof": ["<sibling>", "…"] }
  ```
  `pdf_sha256` is the same fingerprint on the report's own verification page. A report holder proves inclusion by
  matching THEIR OWN `pdf_sha256` (from their verify page) to a leaf, then following the Merkle proof to the signed
  daily root. The report_id and the issuer signature live on the verify page, **not** in the public leaf — so the
  chain is provable per-report but the full set of reports is **not enumerable** (you cannot walk the chain to build
  a directory of who has been investigated).
- **`source-snapshot`** — a public-data snapshot reviewed that day, published as an **opaque hash only**:
  ```json
  { "type": "source-snapshot", "hash": "<hex>", "proof": ["<sibling>", "…"] }
  ```

## Rules

- **Merkle rule: `rfc6962-v2`** (RFC 6962 hashing). Leaf hash = `SHA-256(0x00 || leaf_data)`; internal node =
  `SHA-256(0x01 || left || right)`. The top node is `merkle_root`. A single-leaf tree's root equals its leaf hash.
- **Signatures** are Ed25519 over the **raw 32-byte digest** (not the hex string): `root_signature` over
  `merkle_root`. (A report's own signature over its `pdf_sha256` lives on its verify page, not in the chain leaf.)
- **Leaf-data.** A `cdr` leaf's leaf-data is `pdf_sha256 || json_sha256`; a `source-snapshot` leaf's leaf-data is its
  `hash`. The rfc6962 `0x00` leaf prefix is applied to leaf-data (the chain builder is authoritative on the exact
  byte-encoding).
- **Inclusion proof.** `proof` lists the sibling hashes from the leaf to the root, in order. The reference verifier
  applies the `0x01` node rule at each step. A genesis single leaf has an empty `proof`.
- **Opacity.** `source-snapshot` leaves publish a hash only — never a source name, id, endpoint, table, or path.
  This log publishes proof, not method.
- **Privacy.** No raw source data, no report bodies, and no personal data appear here — only hashes, RFC-3161
  tokens, and public keys.

> The chain builder is the authority on the exact byte-encoding. This file documents the shape so third parties can
> independently verify.
