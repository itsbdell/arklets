# Arklets — Product Requirements Document

## Executive Summary

Arklets are self-contained vessels of knowledge encoded entirely within URLs. No server required. No database. No accounts. The content *is* the URL.

A single URL can carry a recipe, a poem, contact info, emergency instructions, or any text—compressed, encoded, and readable by anyone with a browser. Print it as a QR code, and the knowledge survives without electricity, connectivity, or infrastructure.

---

## Problem Statement

Knowledge distribution assumes infrastructure: servers, databases, connectivity, accounts. When infrastructure fails—disaster, censorship, remote locations, archival scenarios—knowledge becomes inaccessible.

**Current solutions fail because:**
- Websites require servers and connectivity
- PDFs require readers and file storage
- Apps require installation and updates
- Cloud services require accounts and trust

**Arklets solve this by:**
- Encoding content directly into the URL itself
- Using browser-native compression (no dependencies)
- Working entirely offline after a one-time decoder save
- Surviving as printed QR codes (no electricity needed)

---

## Product Vision

**"Knowledge that survives in a URL."**

Arklets enable:
1. **Serverless distribution** — The URL is the content; no hosting required
2. **Offline access** — Works without connectivity once decoder is saved
3. **Physical resilience** — QR codes on paper survive power outages
4. **Privacy by design** — URL fragments never leave the browser (RFC 3986)
5. **Zero dependencies** — Pure browser JavaScript, no npm, no build step

---

## User Personas

### The Prepper
Wants knowledge that survives infrastructure collapse. Prints QR code cards for first aid, water purification, emergency contacts. Needs content that works when the grid is down.

### The Activist
Distributes information in censored environments. Needs content that can't be blocked at the server level because there is no server. Shares via printed cards, peer-to-peer, or any channel.

### The Archivist
Preserves knowledge for the long term. Wants content that doesn't depend on a company staying in business. A URL in a text file or printed page is the distribution format.

### The Educator
Creates portable reference materials for students without reliable internet. Prints card sets covering key topics. Students scan once, save decoder, access forever.

### The Field Worker
Operates in remote areas with no connectivity. Carries printed quick-reference cards. Scans QR codes with phone's offline decoder to access protocols, guides, data.

---

## Core Concepts

### The Arklet

An **arklet** is a URL where the fragment (everything after `#`) contains compressed, encoded content:

```
https://example.com/decoder.html#eNpLSS0u0UtJTSxRAQAYxgOY
```

- The **base URL** points to a decoder page
- The **fragment** contains the actual content
- Per RFC 3986, fragments are **never sent to the server**
- Content stays entirely in the browser—private, local, serverless

### The Decoder

A minimal HTML page (~1.4KB) that:
1. Reads the URL fragment
2. Decodes Base64 to bytes
3. Decompresses using browser-native `DecompressionStream`
4. Displays the original text

The decoder is small enough to:
- Fit inside a QR code as a `data:text/html;base64,...` URI
- Be saved/bookmarked for offline use
- Work on any modern browser without installation

### Card #0

A special QR code containing the entire decoder as a data URI. Scan it once, and your phone/computer has the decoder forever. This is the "key" that unlocks all content cards.

**Distribution strategy:** Get Card #0 to people first. Once they have it, any content card works.

### Content Cards (1–N)

QR codes containing arklet URLs. Each card is self-contained:
- Encodes one piece of content
- Includes visible metadata (title, number, checksum, series)
- Works with the decoder from Card #0
- Can be distributed independently

---

## Technical Architecture

### Encoding Pipeline

```
Text → UTF-8 bytes → Deflate-raw compress → URL-safe Base64 → URL fragment
```

1. **Input**: UTF-8 text string
2. **Compression**: Browser-native `CompressionStream('deflate-raw')`
   - Typically 30-50% reduction for natural language
   - Highly repetitive content compresses extremely well
3. **Base64 encoding**: Standard Base64 with URL-safe substitutions
4. **URL construction**: `decoder.html#[encoded-data]`

### Decoding Pipeline

```
URL fragment → Restore Base64 → Decompress → UTF-8 text
```

1. Extract fragment from URL (or accept pasted input)
2. Restore standard Base64 characters
3. Decode Base64 to bytes
4. Decompress using `DecompressionStream('deflate-raw')`
5. Decode UTF-8 bytes to text string

### URL-Safe Base64

| Standard | URL-Safe | Reason |
|----------|----------|--------|
| `+` | `-` | `+` is space in URLs |
| `/` | `_` | `/` is path separator |
| `=` | (stripped) | Padding is reconstructible |

### Compression Choice: Deflate-raw

| Algorithm | Browser Support | Decoder Size | Compression |
|-----------|-----------------|--------------|-------------|
| deflate-raw | Native | ~1.4KB | Good |
| gzip | Native | ~1.4KB | Similar (+headers) |
| brotli | Native | ~1.4KB | Better, but encoding unsupported |
| lzma | Requires library | +50KB | Best |

**Decision:** Deflate-raw. Native support for both encode and decode, no library overhead, decoder fits in a QR code.

### Size Constraints

| Constraint | Value | Notes |
|------------|-------|-------|
| QR code capacity (Low ECC) | ~2,953 bytes | Maximum for largest QR version |
| Practical content limit | ~2,500 chars | After compression, before hitting QR limit |
| Decoder HTML size | ~1,400 bytes | Fits in QR as data URI |
| URL length (practical) | ~2,000 chars | Some browsers/tools truncate longer |

### Privacy Model

Per RFC 3986, the URL fragment (`#...`) is:
- **Never sent to the server** in HTTP requests
- **Not logged** by web servers, proxies, or CDNs
- **Processed entirely client-side**

This means:
- Hosting the decoder reveals nothing about content
- Content can be shared via any channel (email, QR, text) without server knowledge
- No analytics, no tracking, no logs of what was decoded

---

## Product Components

### 1. Landing Page (index.html)

**Purpose:** Introduce Arklets, demonstrate the concept, provide navigation.

**Features:**
- Interactive demo: type text, see it encoded, decode it back
- Compression statistics (bytes saved)
- "Copy Link" to generate shareable arklet URL
- Links to all tools

### 2. Encoder (encoder.html)

**Purpose:** Create individual arklets with QR codes.

**Features:**
- Text input area
- Encode button with size feedback
- QR code generation for the arklet URL
- Card #0 generator (decoder-as-QR)
- Size warnings when exceeding QR limits

### 3. Decoder (decoder.html)

**Purpose:** Minimal, portable decoder for arklets.

**Features:**
- Auto-decodes URL fragment on page load
- Manual input for pasted encoded data
- Copy button for decoded content
- ~1.4KB total size (fits in QR code)

**Design constraints:**
- Single HTML file, no external dependencies
- Minified for size
- Works offline when saved/bookmarked

### 4. Batch Card Creator (batch.html)

**Purpose:** Generate indexed collections of cards for printing/distribution.

**Features:**

*Input methods:*
- **Batch (JSON):** Paste or load an index file
- **File upload:** Load `.json` index
- **Interactive:** Build cards one at a time with auto-increment

*Index format:*
```json
{
  "series": "Emergency Protocols",
  "version": "1.0",
  "cards": [
    {
      "number": 1,
      "title": "Water Purification",
      "description": "Making water safe to drink",
      "content": "Full instructions here..."
    }
  ]
}
```

*Generation:*
- Optional Card #0 (decoder) as first card
- QR code for each content card
- CRC32 checksum per card for integrity verification
- Footer metadata: `Item #N | Checksum: XXXX | Series: Name`

*Output:*
- Card grid preview
- Round-trip test (encode/decode verification)
- Print-optimized layout (2-column, page-break aware)

---

## User Workflows

### Individual Arklet Creation

1. Open encoder.html
2. Type or paste content
3. Click "Encode"
4. Copy generated URL or scan QR code
5. Share via any channel

### Batch Card Production

1. Open batch.html
2. Enter series name and version
3. Either:
   - Paste/upload JSON index (batch mode)
   - Add items one by one (interactive mode)
4. Click "Generate All"
5. Optionally run round-trip test
6. Print card sheet

### First-Time Consumer Setup

1. Receive Card #0 (decoder QR)
2. Scan with phone camera
3. Browser opens decoder page
4. Bookmark or "Add to Home Screen"
5. Setup complete (~30 seconds)

### Content Access

1. Open saved decoder (bookmark/home screen)
2. Scan content card QR
3. Decoder shows content
4. Works offline, no connectivity needed

### Physical Distribution

1. Print card sheets from batch tool
2. Cut into individual cards
3. Distribute Card #0 widely (it's the key)
4. Distribute content cards as needed
5. Recipients can verify cards using footer checksums

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Decoder size | < 1.5KB |
| First-time setup | < 30 seconds |
| Content access time | < 1 minute |
| Browser support | Chrome 80+, Firefox 113+, Safari 16.4+, Edge 80+ |
| External dependencies | Zero |
| Server requirements | Static file hosting only (or none for local use) |
| Build step | None |
| Offline capability | Full (after decoder saved) |
| Round-trip accuracy | 100% (content in = content out) |

---

## Browser Compatibility

| Feature | Chrome | Edge | Firefox | Safari |
|---------|--------|------|---------|--------|
| CompressionStream | 80+ | 80+ | 113+ | 16.4+ |
| DecompressionStream | 80+ | 80+ | 113+ | 16.4+ |
| TextEncoder/Decoder | All | All | All | All |
| QRCode.js (CDN) | All | All | All | All |

**No polyfills.** Unsupported browsers show an error message.

---

## Security Considerations

### Strengths

- **No server-side data:** Content never touches a server
- **Fragment privacy:** URL fragments not transmitted in HTTP
- **No accounts:** Nothing to breach, no credentials to steal
- **Offline operation:** No network = no network attacks
- **Transparency:** All code is client-side, inspectable

### Limitations

- **No encryption:** Content is encoded, not encrypted (anyone with the URL can decode)
- **QR code visibility:** Physical cards can be photographed/copied
- **Trust in decoder:** Users must trust the decoder source

### Future: Optional Encryption

Could add password-based encryption layer:
```
Text → Encrypt(password) → Compress → Encode → URL
```
Not currently implemented. Would increase complexity and decoder size.

---

## Deployment

### GitHub Pages (Current)

- Push to `main` branch triggers deployment
- Static files served from `src/` directory
- No build step, no CI complexity
- Live at: https://www.briandell.xyz/arklets/

### Self-Hosting

Copy `src/` directory to any static file server:
- Apache/Nginx
- S3/CloudFront
- Netlify/Vercel
- Local file:// (works for decoder)

### Fully Offline

1. Save decoder.html locally
2. Open with `file://` URL
3. Paste encoded data or scan QR codes
4. No server needed at all

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Decoder fits in QR | < 2,953 bytes | File size after base64 data URI |
| Setup time | < 30 seconds | User test: scan Card #0 to working decoder |
| Access time | < 1 minute | User test: scan content card to readable text |
| Compression ratio | 30-50% | Average on natural language content |
| Round-trip accuracy | 100% | Automated test: encode(decode(x)) == x |
| Offline functionality | Full | Test: airplane mode after setup |

---

## Roadmap

### Current (v1.0)
- [x] Core encoding/decoding
- [x] Encoder tool with QR generation
- [x] Minimal decoder (~1.4KB)
- [x] Batch card creator
- [x] Series/version metadata
- [x] CRC32 checksums
- [x] Interactive batch mode
- [x] Print-optimized output

### Future Considerations

**Multi-QR for large content**
- Split content across multiple QR codes
- Reassemble on decode
- Increases capacity beyond single-QR limit

**Index card**
- Special card listing all cards in a series
- Table of contents for physical collections

**Encryption**
- Optional password protection
- Key derivation from passphrase
- Trade-off: increased decoder size

**Mobile app**
- Native QR scanner with built-in decoder
- Better UX than browser bookmark
- Offline-first architecture

**Content types**
- Structured data (JSON/YAML decode + render)
- Markdown rendering
- Simple images (base64, size-limited)

**Version management**
- Indicate when cards supersede previous versions
- Migration paths for updated content

---

## Appendix A: File Structure

```
arklets/
├── src/
│   ├── index.html      # Landing page + demo
│   ├── encoder.html    # Single-item encoder + QR
│   ├── decoder.html    # Minimal decoder (~1.4KB)
│   └── batch.html      # Batch card creator
├── docs/
│   ├── how-it-works.md # Technical deep-dive
│   └── PRD-Arklets.md  # This document
├── examples/
│   └── sample-content.json
├── .github/workflows/
│   └── pages.yml       # GitHub Pages deployment
├── LICENSE             # MIT
└── README.md
```

## Appendix B: Example Arklet

**Input text:**
```
CLASSIC PANCAKES

1 cup flour, 1 tbsp sugar, 1 tsp baking powder, 1/2 tsp salt
Mix dry. Add 1 cup milk, 1 egg, 2 tbsp melted butter.
Stir until just combined (lumps OK).

Heat griddle to 375°F. Pour 1/4 cup per pancake.
Flip when bubbles form and edges set (~2 min).
Cook 1-2 min more. Makes ~8 pancakes.
```

**Encoded (compressed, Base64):**
```
eNpNjsEKwjAQRO_5markup...
```
*(truncated for readability)*

**Full arklet URL:**
```
https://example.com/decoder.html#eNpNjsEKwjAQRO_5...
```

**Card footer:**
```
Item #1 | Checksum: DFF07CB3 | Series: Sample Collection
```

## Appendix C: Why "Arklets"?

An **ark** preserves knowledge through catastrophe. An **arklet** is a small, self-contained vessel—a single piece of knowledge encoded to survive without infrastructure.

The name suggests:
- **Resilience:** Survives when systems fail
- **Self-contained:** Everything needed is in the vessel
- **Portable:** Small enough to carry, share, print
- **Preservation:** Knowledge that endures

---

*Document version: 1.0*
*Last updated: 2026-02-03*
