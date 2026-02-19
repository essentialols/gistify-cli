# Gistify CLI

CLI reimplementation of the [Gistify](https://chromewebstore.google.com/detail/gistify/hlpmoeanajniaaofddpeffghklanpmpe) Chrome extension. Summarizes web pages and PDFs using the Gistify serverless backend.

## How it works

1. **Text extraction**: Uses Playwright (stealth mode) to load web pages and extract `document.body.innerText`. For arXiv URLs, it rewrites to the PDF URL and downloads directly. Local PDFs are extracted via pdfminer.six.
2. **API call**: Sends extracted text to the Gistify Netlify function (`POST .netlify/functions/summarize` with `{"text": "..."}`). No API key required.
3. **Output**: Saves a Markdown file with title, source URL, summary, and date.

## Setup

```bash
pip install -r requirements.txt
python -m playwright install chromium
```

No API key needed — the Gistify backend is a public Netlify function.

## Usage

```bash
python summarize.py <url_or_pdf>
python summarize.py <url_or_pdf> --output summary.md
python summarize.py --debug                           # dump raw API response
python summarize.py --proxy                           # force proxy on
python summarize.py --no-proxy                        # force proxy off
```

## Current status: API broken

As of 2026-02-19, the Gistify backend API returns a 500 error:

```
{"error": "https://api-inference.huggingface.co is no longer supported. Please use https://router.huggingface.co instead."}
```

The Netlify serverless function proxies to the Hugging Face Inference API, which deprecated its old endpoint (`api-inference.huggingface.co`). This is a server-side issue — the Chrome extension has the same problem. The CLI code is correct and will work if/when the extension author updates the backend.

**Text extraction still works** — you can verify PDF/URL extraction with `--debug`.

## Known limitations

- **Backend broken**: See above. The Gistify server needs to update its Hugging Face endpoint.
- **No auth/model control**: The AI model is hidden behind the Netlify function. No way to choose model, adjust temperature, or customize the prompt.
- **Rate limiting**: Local rate limiter enforces 10 requests/hour and 5s minimum interval.

## Proxy support

Reads proxy configuration from `~/.scholar-proxies.json`:

```json
{
  "enabled": true,
  "proxies": ["http://proxy1:8080", "http://proxy2:8080"]
}
```

Override with `--proxy` or `--no-proxy` flags.
