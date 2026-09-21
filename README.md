# Carso Diligence Lookback Chain

A public, append-only, cryptographically timestamped log of Carso Diligence™ activity.

Each day, Carso Diligence commits one **Merkle root**. The root fixes two things for that date:

1. the **fingerprints of the diligence reports issued that day**, and
2. the **fingerprints of the public-data snapshots reviewed** for reports.

The daily root is timestamped by **two independent RFC-3161 timestamp authorities** and signed with a
Carso-controlled key. Together, that makes each day's record **tamper-evident** and **independently datable**.

## What this proves — and what it does not

This chain proves **when** a diligence report existed and **what its exact bytes were** on that date. It proves
that the data reviewed had a **fixed state** on that date. Anyone can check these facts without trusting Carso.

This chain does **not** state that a report's conclusions are correct, and it does **not** state that any carrier
is honest or trustworthy. A signed, timestamped record proves **who issued it and when** — not that its contents
are true. Each report states its own findings, with its own sources.

## What is published here

- `days/<YYYY-MM-DD>/root.json` — the day's Merkle root, the day's leaf hashes, and a link to the previous day's
  root (the chain link).
- `days/<YYYY-MM-DD>/*.tsr` — the RFC-3161 timestamp tokens for the day's root (one per authority).
- `keys/` — the public keys used to sign the daily roots and the reports. Keys are versioned; each record names
  the exact key that signed it.
- `VERIFY.md` — step-by-step verification you can run yourself.
- `FORMAT.md` — the exact structure of `root.json`.

## What is not published here

The list of data sources reviewed, the fetch methods, and any raw source data are **not** published. Source
snapshots appear only as **opaque hashes** (`type: "source-snapshot"`). This log publishes the **proof**, not the
method.

Report-level detail for a single report is on its own verification page:
`https://diligence.carso.cloud/verify/<report-id>/`.

## Leaf types

- `cdr` — an issued Carso Diligence Report. Fields: `report_id`, `pdf_sha256`, `json_sha256`,
  `evidence_bundle_sha256` (may be null), and the report's Ed25519 `signature` + `key_id`.
- `source-snapshot` — a public-data snapshot reviewed that day, published as an opaque `hash` only.

## Quick check

Confirm your copy of a report is the exact one recorded:

```
shasum -a 256 <report-id>.pdf
```

The printed SHA-256 must match the `pdf_sha256` of that report's `cdr` leaf in the day it was issued. Then follow
`VERIFY.md` to check the Merkle inclusion proof, the two timestamp tokens, and the signature.

---

Carso Diligence is a product of Carso Cybernetics. This repository is machine-generated and append-only.
