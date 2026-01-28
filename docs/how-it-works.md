# How Arklets Works

Arklets encodes text directly into URL fragments. No server stores your content — the URL itself *is* the content.

## The Encoding Pipeline

```
Text → Deflate compress → Base64 encode → URL #fragment
```

### Step by step

1. **Text input** — Content enters as a UTF-8 string.

2. **Deflate compression** — The browser's native `CompressionStream('deflate-raw')` compresses the text. This typically achieves 30–50% reduction, depending on the content. Natural language compresses well; random data does not.

3. **URL-safe Base64 encoding** — The compressed bytes are Base64-encoded using a URL-safe alphabet:
   - `+` → `-`
   - `/` → `_`
   - Trailing `=` padding is stripped

4. **URL fragment** — The encoded string is placed after `#` in the URL:
   ```
   decoder.html#eNpLSS0u0UtJTSxRAQAYxgOY
   ```

5. **Fragment privacy** — Per [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986#section-3.5), the fragment identifier is never sent to any server. When a browser requests `https://example.com/page.html#data`, the server only sees a request for `page.html`. The `#data` part stays entirely in the browser.

6. **Decoding** — The decoder reverses the process: strip the `#`, restore standard Base64 characters, decode to bytes, decompress with `DecompressionStream('deflate-raw')`, and decode the resulting UTF-8 bytes back to text.

## Why Deflate?

The decoder must fit inside a QR code as a `data:text/html;base64,...` URI. QR codes can hold roughly **2,953 bytes** of data (at the highest capacity level with low error correction).

| Algorithm | Decoder size | Fits in QR? |
|-----------|-------------|-------------|
| **Deflate** (browser-native) | ~1.4 KB | Yes |
| LZMA (JavaScript library) | ~5 KB+ | No |
| Brotli (browser-native decode) | ~1.4 KB | Yes, but no `CompressionStream` for encoding |

Deflate via `CompressionStream`/`DecompressionStream` is the clear winner:
- Zero external dependencies — it's built into every modern browser
- The decoder is under 1,500 bytes of raw HTML
- Both encoding and decoding are natively supported

## QR Code Capacity

QR codes have a maximum data capacity that depends on the error correction level and encoding mode:

| Error Correction | Numeric | Alphanumeric | Binary (bytes) |
|-----------------|---------|--------------|----------------|
| Low (L) | 7,089 | 4,296 | **2,953** |
| Medium (M) | 5,596 | 3,391 | 2,331 |
| Quartile (Q) | 3,993 | 2,420 | 1,663 |
| High (H) | 3,057 | 1,852 | 1,273 |

Since Base64-encoded data is binary from QR's perspective, the practical limit for content URLs is around **2,953 bytes** at Low error correction. Content that compresses to more than ~2,900 bytes of Base64 output won't fit in a single QR code.

## Card #0: The Self-Bootstrap

Card #0 is a QR code that contains the **decoder itself** as a `data:text/html;base64,...` URI.

When you scan Card #0:
1. Your device reads the QR code and gets a `data:` URI
2. Opening the URI loads a fully functional HTML decoder page directly in the browser
3. No internet connection required — the entire decoder is in the QR code

This makes a printed set of Arklets cards fully **self-contained**:
- **Card #0** = the decoder (the key)
- **Cards #1, #2, ...** = the encoded content

Anyone with a phone camera can scan Card #0 to get the decoder, then scan any other card to read its content. No app installation, no internet, no server.

### How it's generated

1. Read the `decoder.html` file (raw HTML, ~1.4 KB)
2. Base64-encode the entire HTML: `data:text/html;base64,[base64-of-html]`
3. Render the data URI as a QR code
4. The data URI is roughly `22 + (1.33 × file_size)` bytes — for a 1,100-byte decoder, that's ~1,485 bytes, well within QR limits

## Batch Workflow

The Batch Card Creator (`batch.html`) generates printable card sheets:

1. **Input** — Paste JSON, upload a `.json` file, or add items one by one
2. **Card #0 toggle** — Optionally include the decoder QR as the first card (on by default)
3. **Encoding** — Each content item is compressed and encoded independently
4. **QR generation** — Each card gets its own QR code containing `decoder.html#[encoded-data]`
5. **Size validation** — Cards that exceed the QR byte limit are flagged with a warning
6. **Round-trip test** — The "Test" button encodes and decodes every card, verifying the round-trip matches
7. **Print** — Print-optimized CSS produces a clean card grid with no UI chrome

A printed card sheet with Card #0 included is a fully self-contained knowledge artifact — everything needed to read the content is on the paper itself.

## Browser Compatibility

The Compression Streams API (`CompressionStream` / `DecompressionStream`) is supported in:

| Browser | Version |
|---------|---------|
| Chrome | 80+ (Jan 2020) |
| Edge | 80+ (Jan 2020) |
| Firefox | 113+ (May 2023) |
| Safari | 16.4+ (Mar 2023) |

The decoder is read-only (decompression only), so it works in any browser with `DecompressionStream` support. The encoder requires `CompressionStream`, which has the same support matrix.

No polyfills are used — if the browser doesn't support these APIs, the tool will show an error. All modern browsers (2023+) are supported.