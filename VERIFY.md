# How to verify

You need no account and no Carso software. You need `openssl`, `shasum` (or `sha256sum`), and `curl`.

## 1. Your report copy is unaltered

Compute the fingerprint of your PDF:

```
shasum -a 256 <report-id>.pdf
```

Find the report's `cdr` leaf in `days/<issue-date>/root.json`. The printed value must equal that leaf's
`pdf_sha256`, exactly. If it matches, your copy is byte-for-byte the recorded report.

## 2. The report was issued by Carso Diligence

Each `cdr` leaf carries an Ed25519 `signature` over the PDF fingerprint, plus the `key_id` that signed it. Get the
named public key from `keys/`, then verify (run in an empty directory):

```
printf '%s' '<pdf_sha256>' | xxd -r -p > msg.bin
printf '%s' '<signature>'  | base64 -d  > sig.bin
openssl pkeyutl -verify -pubin -inkey keys/<key_id>.pub.pem -rawin -in msg.bin -sigfile sig.bin
```

Expected output: `Signature Verified Successfully`. Use the exact key named in the leaf — keys rotate, and each
record verifies against the key that signed it.

## 3. The report is inside that day's tree

Each `cdr` and `source-snapshot` leaf lists its Merkle **inclusion proof** (the sibling hashes from the leaf up to
the day's root). The tree uses RFC 6962 hashing (`merkle_rule: "rfc6962-v2"`): leaf = `SHA-256(0x00 || data)`,
node = `SHA-256(0x01 || left || right)`. Recompute the path from the leaf through each sibling; the result must
equal `merkle_root` in `root.json`. A genesis single-leaf tree's root equals its leaf hash. See `FORMAT.md`.

## 4. The day's root existed on that date

Each day's root is timestamped by two independent RFC-3161 authorities. Verify each token against the root:

```
openssl ts -verify -digest <merkle_root> -sha256 \
  -in days/<date>/root.<tsa>.tsr -CAfile <authority-ca>.pem
```

Expected output: `Verification: OK`. The token is signed by the authority's own key over the root plus its clock.
Carso cannot issue, alter, or backdate it. Two independent authorities mean no single timestamp source has to be
trusted.

## 5. The chain is continuous

Each `root.json` records `prev_root` — the previous day's `merkle_root`. Follow the links back to genesis. A break
or a rewrite is visible: a changed past day would change every root after it, and the timestamp tokens would no
longer verify.

---

What this establishes: the report existed, unaltered, on the recorded date, issued by the named key, inside a
continuous public chain. It does not establish that the report's conclusions are correct — read the report for
that.
