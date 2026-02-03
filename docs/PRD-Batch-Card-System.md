# Arklets Batch Card System — Product Requirements Document

## Overview

The Arklets Batch Card System generates, indexes, and distributes collections of self-contained knowledge artifacts. Each artifact is a URL-encoded document that works offline, requires no server, and can be printed as a QR code card.

**Core insight**: Card #0 is a decoder/reader. Cards 1–N are content. Once someone has Card #0, they can access any content card forever—offline, without accounts, without dependencies.

---

## Problem Statement

Distributing knowledge in hostile or degraded environments (no internet, censorship, disaster zones, archival scenarios) requires:

1. Content that survives without infrastructure
2. A reader/decoder that can be distributed alongside content
3. A way to organize and verify content without opening it
4. Batch generation from a single source of truth

Existing solutions (PDFs, websites, apps) fail when connectivity fails. Arklets encode content directly into URLs, making the content itself the distribution medium.

---

## User Personas

### Creator (Author)
- Wants to produce a collection of related documents
- Needs batch generation from an index file (deterministic, repeatable)
- Needs interactive authoring for ad-hoc collections
- Wants print-ready output for physical distribution

### Consumer (Recipient)
- Receives Card #0 once, saves it locally
- Scans or opens content cards as needed
- Works entirely offline after initial Card #0 setup
- Can verify card integrity using visible checksums

### Distributor
- Hands out physical cards (printed QR codes)
- Ensures Card #0 reaches recipients first
- Uses visible metadata (number, series, checksum) to organize inventory

---

## Functional Requirements

### Index Format

A JSON file defines a named, versioned collection:

```json
{
  "series": "Field Guide",
  "version": "1.2",
  "cards": [
    {
      "number": 1,
      "title": "Water Purification",
      "description": "Methods for making water safe",
      "content": "Full text content here..."
    }
  ]
}
```

| Field | Purpose |
|-------|---------|
| `series` | Groups related items into a named collection |
| `version` | Tracks revisions to the collection |
| `number` | Sequential position within the series |
| `title` | Human-readable name (visible without decoding) |
| `description` | Summary visible in card preview |
| `content` | Full body, encoded at generation time |

### Generation Modes

**Batch Mode** (from index):
1. Load JSON index
2. Iterate through all items in order
3. For each item: compress content, encode to URL-safe Base64, compute CRC32 checksum
4. Render card with footer: `Item #N | Checksum: XXXX | Series: Name`
5. Generate QR code for each card
6. Output entire set in one action

**Interactive Mode** (build as you go):
1. Fill in item fields (number, title, description, content)
2. Click "Add to Batch" to queue item
3. Number auto-increments, form clears, batch counter updates
4. Items accumulate until "Generate All"
5. Batch can be cleared and rebuilt at any time

Both modes converge on identical output.

### Card #0 (Decoder)

- Contains the entire decoder application (~1.4KB HTML)
- Encoded as a `data:text/html;base64,...` URI
- Rendered as a QR code
- Scannable by any QR reader → opens decoder in browser
- Works offline after first scan (can be bookmarked/saved)
- Required once per recipient; unlocks all content cards

### Content Cards (1–N)

Each card contains:
- QR code encoding: `decoder.html#[compressed-base64-content]`
- Visible metadata: number, title, description preview
- Footer: `Item #N | Checksum: XXXX | Series: Name`
- Size validation (warns if exceeds QR byte limit of ~2,953 chars)

### Integrity Verification

- CRC32 checksum computed on encoded (post-compression) data
- Printed on every card footer
- Allows verification without decoding
- Deterministic: same content always produces same checksum

### Print Output

- 2-column card grid optimized for letter/A4
- Page breaks avoid splitting cards
- UI chrome hidden in print
- Cards include all metadata needed for physical organization

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| First-time setup (Card #0) | < 30 seconds |
| Content access | < 1 minute |
| Dependencies | None (pure browser JS) |
| Offline capability | Full (after Card #0 saved) |
| Accounts/permissions | None |
| Server requirement | None (static files only) |
| QR code capacity | 2,953 bytes (Low ECC) |
| Compression | deflate-raw (browser-native) |

---

## Technical Architecture

### Encoding Pipeline

```
Text → UTF-8 bytes → Deflate-raw compress → URL-safe Base64 → URL fragment
```

### Decoding Pipeline

```
URL fragment → Restore Base64 → Decompress → UTF-8 text
```

### URL-Safe Base64 Alphabet

| Standard | URL-Safe |
|----------|----------|
| `+` | `-` |
| `/` | `_` |
| `=` | (stripped) |

### Checksum Algorithm

CRC32 with polynomial 0xEDB88320, computed on the encoded string (after compression and Base64), output as 8-character uppercase hex.

### Browser Compatibility

| Feature | Chrome | Edge | Firefox | Safari |
|---------|--------|------|---------|--------|
| CompressionStream | 80+ | 80+ | 113+ | 16.4+ |
| DecompressionStream | 80+ | 80+ | 113+ | 16.4+ |

---

## User Interface

### Creator Tool (batch.html)

**Top section:**
- Series name input
- Version input (defaults to "1.0")

**Tabs:**
1. **Batch (JSON)** — Paste JSON index, auto-populates series/version from JSON
2. **Upload File** — Load `.json` file, same behavior as paste
3. **Interactive** — Form with number/title/description/content, "Add to Batch" button, queued items list, batch counter

**Actions:**
- "Generate All" — Produces all cards from active input source
- "Test Round-Trip" — Encodes and decodes every item, reports pass/fail
- "Print Cards" — Opens print dialog with optimized layout

**Options:**
- Checkbox: Include Card #0 (Decoder) as first card

**Output:**
- Card grid showing number, title, content preview, size, QR code, footer

### Decoder (decoder.html)

- Minimal (~1.4KB minified)
- Auto-decodes URL fragment on load
- Paste input for manual decode
- Copy button for decoded content
- No external dependencies

---

## Distribution Strategy

1. **Card #0 first**: Ensure recipients have the decoder before content cards
2. **Wide availability**: Card #0 can be posted publicly, shared freely, printed in bulk
3. **Content independence**: Each content card is self-contained; can be distributed individually
4. **Physical organization**: Footer metadata (number, checksum, series) enables sorting without tooling
5. **Version tracking**: Series version in footer identifies which batch a card belongs to

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Card #0 scan-to-working decoder | < 30 seconds |
| Content card scan-to-readable | < 1 minute |
| Round-trip encode/decode accuracy | 100% |
| Print layout: cards per page | 4–6 |
| Offline functionality | Full |

---

## Future Considerations

- **Multi-QR for large content**: Split content across multiple QR codes with reassembly
- **Index card**: A special card that lists all cards in the series (table of contents)
- **Encryption**: Optional password-protected content
- **Versioned updates**: Mechanism to indicate superseded cards
- **Mobile app**: Native scanner with built-in decoder for better UX

---

## Appendix: Example Session

**Batch mode:**
1. Open batch.html
2. Enter series name: "Emergency Protocols"
3. Paste JSON index with 10 items
4. Click "Generate All"
5. Review cards, run round-trip test
6. Print card sheet

**Interactive mode:**
1. Open batch.html, switch to Interactive tab
2. Enter series name: "Quick Reference"
3. Fill in Item #1: title, content
4. Click "Add to Batch" (counter shows 1)
5. Form clears, number auto-increments to 2
6. Repeat for items 2, 3, 4
7. Click "Generate All"
8. Print or distribute digitally

**Consumer workflow:**
1. Receive Card #0 (decoder QR)
2. Scan with phone camera → opens decoder in browser
3. Bookmark/save decoder page
4. Scan any content card → decoder shows content
5. Works offline indefinitely
