# Tools

A short list of practical tools for working with Base64. The browser and your terminal already cover most cases — reach for an online converter mainly for quick one-off conversions.

## Online converters

- [EncodingBase64](https://encodingbase64.com/) — a free client-side Base64 encoder for text, files, images, URL-safe Base64, Base64 to Hex, Hex encoding, and URL encoding. Runs entirely in the browser, so your input never leaves the page.
- [DecodingBase64](https://decodingbase64.com/) — a free client-side Base64 decoder that converts Base64 to text, images, hex output, and downloadable files.

## Built into your browser

Every modern browser ships `btoa`, `atob`, `TextEncoder`, and `TextDecoder`. Paste these into the DevTools console:

```js
btoa("Hello World");                 // "SGVsbG8gV29ybGQ="
atob("SGVsbG8gV29ybGQ=");            // "Hello World"

// Unicode-safe
const bytes = new TextEncoder().encode("héllo 🌍");
const b64 = btoa(String.fromCharCode(...bytes));
```

See [examples/javascript-examples.md](examples/javascript-examples.md).

## Built into Python

The standard-library `base64` module handles everything, including URL-safe variants:

```python
import base64
base64.b64encode(b"hello")
base64.urlsafe_b64encode(b"hello")
```

See [examples/python-examples.md](examples/python-examples.md).

## Built into your terminal

macOS and Linux ship a `base64` command:

```sh
printf 'Hello World' | base64
echo 'SGVsbG8gV29ybGQ=' | base64 --decode
base64 input.png > input.png.b64
```

On macOS, the decode flag is `-D` (short form) instead of `--decode`.

Windows PowerShell uses `[Convert]`:

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("Hello World"))
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("SGVsbG8gV29ybGQ="))
```

See [examples/cli-examples.md](examples/cli-examples.md).

## When to use which

| Situation                                         | Best tool                              |
| ------------------------------------------------- | -------------------------------------- |
| Quick one-off conversion, no install              | Online converter                        |
| Converting sensitive data                         | Browser console or local CLI            |
| Working with a file                               | `base64` CLI or Python script           |
| Inside an app or automation                       | JavaScript / Python / your language SDK |
| Debugging a JWT or OAuth token                    | Base64URL-aware tool or script          |
