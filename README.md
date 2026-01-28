# Arklets

**Small arks. Self-contained vessels of knowledge encoded directly into URLs.**

Encode text directly into URLs. No server. The link is the content.

**[Live Demo](https://www.briandell.xyz/arklets/)**

## How It Works

```
Text → Deflate compress → Base64 encode → URL #fragment
```

The `#fragment` part of a URL is never sent to any server — it stays entirely in the browser. That means content encoded in an Arklet is:

| Property | Description |
|----------|-------------|
| **No Server** | Runs entirely in the browser — no backend, no hosting, no database |
| **Permanent** | The URL *is* the content — nothing to expire, no link rot |
| **Private** | The `#fragment` never leaves the browser (per RFC 3986) |
| **Portable** | Works offline, in QR codes, on paper, in bookmarks |

## Quick Start

Open the live demo at **[briandell.xyz/arklets](https://www.briandell.xyz/arklets/)**, or clone the repo and open `src/index.html` in your browser:

```bash
git clone https://github.com/itsbdell/arklets.git
open arklets/src/index.html
```

No build step. No dependencies. Just HTML files.

## Files

| File | Description |
|------|-------------|
| [`src/index.html`](src/index.html) | Landing page with live demo |
| [`src/encoder.html`](src/encoder.html) | Encode text, generate QR codes, download Decoder QR (Card #0) |
| [`src/decoder.html`](src/decoder.html) | Minimal decoder (~1.4KB) — reads URL fragment or pasted data |
| [`src/batch.html`](src/batch.html) | Batch card creator — load multiple items, preview QR codes, print card sheets |
| [`examples/sample-content.json`](examples/sample-content.json) | Example content cards (recipe, contact card, poem) |
| [`docs/how-it-works.md`](docs/how-it-works.md) | Technical explanation of the encoding pipeline |

## Use Cases

- **Sharing text** — Send a URL that contains the content itself, with no hosting needed
- **QR-encoded content** — Generate QR codes that decode to text in any browser
- **Offline-readable cards** — Print card sheets with QR codes that work without internet
- **Privacy-preserving sharing** — Content never touches a server
- **Printable knowledge cards** — Create self-contained card sets with the decoder included (Card #0)

## Technical Details

- Uses browser-native `CompressionStream('deflate-raw')` / `DecompressionStream('deflate-raw')` — zero external dependencies for encoding/decoding
- URL-safe Base64: `+` → `-`, `/` → `_`, no padding
- Decoder is under 1,500 bytes — small enough to fit in a QR code as a `data:text/html;base64,...` URI
- Card #0: the decoder itself encoded as a QR code, making any printed card sheet self-bootstrapping

See [docs/how-it-works.md](docs/how-it-works.md) for the full technical explanation.

## Browser Compatibility

| Browser | Version |
|---------|---------|
| Chrome | 80+ |
| Edge | 80+ |
| Firefox | 113+ |
| Safari | 16.4+ |

No frameworks, no build tools, no npm — pure HTML/CSS/JS that opens directly in a browser.

## Origin

Arklets grew from a "Resilient Knowledge" exploration by [Brian Dell](https://github.com/itsbdell) — extracting the core URL-encoding system into a standalone, general-purpose tool.

## Attribution

Inspired by [topaz/paste](https://github.com/topaz/paste) and [otherjoel/poest](https://github.com/otherjoel/poest).

## License

[MIT](LICENSE)