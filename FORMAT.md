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

- **`cdr`** — an issued Carso Diligence Report:
  ```json
  { "type": "cdr", "report_id": "CDR-YYYYMMDD-XXXX",
    "pdf_sha256": "<hex>", "json_sha256": "<hex>", "evidence_bundle_sha256": null,
    "signature": "<base64 Ed25519 over the raw PDF digest>", "key_id": "carso-diligence-2026-08",
    "proof": ["<sibling>", "…"] }
  ```
  `pdf_sha256` is the same fingerprint on the report's own verification page.
- **`source-snapshot`** — a public-data snapshot reviewed that day, published as an **opaque hash only**:
  ```json
  { "type": "source-snapshot", "hash": "<hex>", "proof": ["<sibling>", "…"] }
  ```

## Rules

- **Merkle rule: `rfc6962-v2`** (RFC 6962 hashing). Leaf hash = `SHA-256(0x00 || leaf_data)`; internal node =
  `SHA-256(0x01 || left || right)`. The top node is `merkle_root`. A single-leaf tree's root equals its leaf hash.
- **Signatures** are Ed25519 over the **raw 32-byte digest** (not the hex string): `root_signature` over
  `merkle_root`, and a `cdr` leaf's `signature` over its `pdf_sha256`.
- **Inclusion proof.** `proof` lists the sibling hashes from the leaf to the root, in order. The reference verifier
  applies the `0x01` node rule at each step. A genesis single leaf has an empty `proof`.
- **Opacity.** `source-snapshot` leaves publish a hash only — never a source name, id, endpoint, table, or path.
  This log publishes proof, not method.
- **Privacy.** No raw source data, no report bodies, and no personal data appear here — only hashes, RFC-3161
  tokens, and public keys.

> The chain builder is the authority on the exact byte-encoding. This file documents the shape so third parties can
> independently verify.
