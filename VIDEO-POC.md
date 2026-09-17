# Video PoC recording guide

This is a non-weaponized recording plan. The public GitHub Pages site contains only a benign WebP. The malformed decoder fixture stays local and is exercised only by an isolated AddressSanitizer harness.

## Correct classification

The decoder finding is a **heap-buffer-overflow (out-of-bounds write)**, not a memory leak.

## Shot 1 — public delivery path

1. Open `https://adolphoramirez.github.io/battlenet-webp-delivery-poc/?v=3`.
2. Show the page source containing the absolute `og:image` and `twitter:image` URL.
3. Send that page URL in the isolated one-member Battle.net group.
4. Record the benign test-pattern preview rendering in Battle.net.

## Shot 2 — exact-byte proof

Run locally:

```sh
python3 /Users/adamdjemai/battlenet-webp-poc/verify_delivery.py \
  --payload /Users/adamdjemai/battlenet-webp-pages/safe.webp \
  --url-fragment 'adolphoramirez.github.io/battlenet-webp-delivery-poc/safe.webp?v=3'
```

Expected evidence:

```text
payload_size=8592
payload_sha256=ae0954add6749b4354b022df1960ec97c774d71f98ce1902cd7c3a73bcd7f0ab
cache_entry=.../59f17f4428836191_0
riff_offset=102 size=8592 chunk=VP8
exact_match=true
summary matching_entries=1 webp_responses=1 exact_matches=1
```

## Shot 3 — local vulnerable decoder

In the local lab only, run the malformed fixture against the vulnerable ASan harness:

```sh
cd /Users/adamdjemai/battlenet-webp-poc
/tmp/bnet-webp-harness-vuln web/crash.webp
```

Record the AddressSanitizer report showing:

```text
ERROR: AddressSanitizer: heap-buffer-overflow
WRITE of size 4
ReplicateValue
BuildHuffmanTable
The write is located 152 bytes after a 12072-byte heap region.
```

## Shot 4 — patched control

Run the identical local fixture against the patched build:

```sh
/tmp/bnet-webp-harness-patched web/crash.webp
```

Expected result: `decoder rejected input`, with no AddressSanitizer finding.

## Scope statement for the video

The recording proves two separate facts: attacker-selected benign WebP bytes reach Battle.net's preview decoder unchanged, and the vulnerable decoder build has a reproducible out-of-bounds heap write. It does not claim code execution. No malformed image, shellcode, or command payload is hosted or sent through chat.
