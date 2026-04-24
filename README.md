# Base64 Cheatsheet

A practical, beginner-friendly reference for Base64 encoding and decoding. Covers the alphabet, padding, Base64URL, image data URIs, and real examples in the terminal, JavaScript, and Python.

## Table of contents

- [What is Base64?](#what-is-base64)
- [Quick examples](#quick-examples)
- [The Base64 alphabet](#the-base64-alphabet)
- [Padding](#padding)
- [Base64 vs Base64URL](#base64-vs-base64url)
- [Text to Base64](#text-to-base64)
- [Base64 to text](#base64-to-text)
- [Image to Base64](#image-to-base64)
- [CLI examples](#cli-examples)
- [JavaScript examples](#javascript-examples)
- [Python examples](#python-examples)
- [Common use cases](#common-use-cases)
- [Common mistakes](#common-mistakes)
- [Helpful online tools](#helpful-online-tools)
- [More examples](#more-examples)
- [License](#license)

## What is Base64?

Base64 is a way of representing binary data using only 64 printable ASCII characters. It takes three bytes of input (24 bits) and encodes them as four characters (6 bits each), so the output is always about 33% larger than the input.

Base64 is **not encryption**. It is an encoding — anyone can decode it. Its purpose is to move binary data safely through systems that were designed for text, such as email bodies, JSON payloads, URLs, HTTP headers, and configuration files.

## Quick examples

| Input                 | Base64                           |
| --------------------- | -------------------------------- |
| `Hi`                  | `SGk=`                           |
| `Hello`               | `SGVsbG8=`                       |
| `Hello World`         | `SGVsbG8gV29ybGQ=`               |
| `{"ok":true}`         | `eyJvayI6dHJ1ZX0=`               |

Reverse the table to decode Base64 back into text.

## The Base64 alphabet

Standard Base64 uses 64 characters plus `=` for padding:

```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m n o p q r s t u v w x y z
0 1 2 3 4 5 6 7 8 9 + /
```

Each character represents 6 bits of data. The `=` character is not part of the alphabet — it only appears at the end as padding.

## Padding

Base64 output length is always a multiple of 4. When the input length is not a multiple of 3 bytes, `=` characters are appended to pad the output:

| Input bytes | Output length | Padding |
| ----------- | ------------- | ------- |
| 3           | 4             | none    |
| 2           | 4             | `=`     |
| 1           | 4             | `==`    |

Example:

```
"A"   -> "QQ=="
"AB"  -> "QUI="
"ABC" -> "QUJD"
```

See [examples/base64-padding.md](examples/base64-padding.md) for more.

## Base64 vs Base64URL

Standard Base64 uses `+` and `/`, which have special meaning in URLs and filenames. Base64URL replaces them:

| Standard Base64 | Base64URL |
| --------------- | --------- |
| `+`             | `-`       |
| `/`             | `_`       |
| `=` (padding)   | often omitted |

Base64URL is used in **JWTs**, OAuth tokens, and anywhere a Base64 string must survive being placed in a URL or filename. See [examples/base64url.md](examples/base64url.md).

## Text to Base64

JavaScript:

```js
btoa("Hello World"); // "SGVsbG8gV29ybGQ="
```

Python:

```python
import base64
base64.b64encode(b"Hello World").decode()  # "SGVsbG8gV29ybGQ="
```

Terminal (macOS/Linux):

```sh
printf 'Hello World' | base64
# SGVsbG8gV29ybGQ=
```

## Base64 to text

JavaScript:

```js
atob("SGVsbG8gV29ybGQ="); // "Hello World"
```

Python:

```python
import base64
base64.b64decode("SGVsbG8gV29ybGQ=").decode()  # "Hello World"
```

Terminal (macOS/Linux):

```sh
echo 'SGVsbG8gV29ybGQ=' | base64 --decode
# Hello World
```

## Image to Base64

An image can be embedded in HTML or CSS as a **data URI**:

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA...
```

Use it like this:

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA..." alt="icon">
```

Embedding is useful for:

- Small icons or sprites (a few KB)
- Self-contained HTML emails
- Inlining a logo in a PDF report

Avoid it for large images — the file grows by ~33%, bypasses the browser cache, and bloats HTML/CSS. Keep large images as regular files. See [examples/image-to-base64.md](examples/image-to-base64.md).

## CLI examples

Encode and decode strings:

```sh
printf 'Hello World' | base64
echo 'SGVsbG8gV29ybGQ=' | base64 --decode
```

Encode and decode files:

```sh
base64 input.png > input.png.b64
base64 --decode input.png.b64 > output.png
```

On macOS, use `base64 -D` instead of `--decode`. Windows users can use PowerShell's `[Convert]::ToBase64String` and `[Convert]::FromBase64String`. See [examples/cli-examples.md](examples/cli-examples.md).

## JavaScript examples

Simple ASCII text with `btoa` / `atob`:

```js
const encoded = btoa("Hello");       // "SGVsbG8="
const decoded = atob(encoded);       // "Hello"
```

Unicode-safe encoding (because `btoa` only accepts Latin-1):

```js
function encodeUtf8(str) {
  const bytes = new TextEncoder().encode(str);
  return btoa(String.fromCharCode(...bytes));
}

function decodeUtf8(b64) {
  const binary = atob(b64);
  const bytes = Uint8Array.from(binary, c => c.charCodeAt(0));
  return new TextDecoder().decode(bytes);
}

encodeUtf8("héllo 🌍");               // "aMOpbGxvIPCfjI0="
decodeUtf8("aMOpbGxvIPCfjI0=");      // "héllo 🌍"
```

See [examples/javascript-examples.md](examples/javascript-examples.md) for `FileReader` and Base64URL helpers.

## Python examples

```python
import base64

# Text
base64.b64encode(b"Hello").decode()            # "SGVsbG8="
base64.b64decode("SGVsbG8=").decode()          # "Hello"

# Files
with open("photo.jpg", "rb") as f:
    encoded = base64.b64encode(f.read()).decode()

with open("out.jpg", "wb") as f:
    f.write(base64.b64decode(encoded))

# URL-safe
base64.urlsafe_b64encode(b"subjects?").decode()  # "c3ViamVjdHM_"
```

See [examples/python-examples.md](examples/python-examples.md).

## Common use cases

- Embedding small images or fonts in HTML, CSS, or email
- Encoding binary payloads inside JSON
- HTTP Basic authentication headers
- JWT header and payload segments (Base64URL)
- API keys and tokens
- Storing binary blobs in a text-only database column
- Data URIs in browser extensions and userscripts

## Common mistakes

- Treating Base64 as encryption — it only obscures, never protects.
- Forgetting that encoded output is ~33% larger than the input.
- Missing or extra `=` padding causing decode errors.
- Using standard Base64 where Base64URL is required (URLs, JWTs, filenames).
- Decoding binary data and treating the result as a UTF-8 string.
- Copy/paste adding line breaks or whitespace that trip strict decoders.

See [examples/common-errors.md](examples/common-errors.md) for fixes.

## Helpful online tools

- [EncodingBase64](https://encodingbase64.com/) — a free client-side Base64 encoder for text, files, images, URL-safe Base64, Base64 to Hex, Hex encoding, and URL encoding.
- [DecodingBase64](https://decodingbase64.com/) — a free client-side Base64 decoder that converts Base64 to text, images, hex output, and downloadable files.

See [tools.md](tools.md) for the full list, including built-in platform tools.

## More examples

- [Text to Base64](examples/text-to-base64.md)
- [Base64 to text](examples/base64-to-text.md)
- [Image to Base64](examples/image-to-base64.md)
- [Base64 to image](examples/base64-to-image.md)
- [Base64URL](examples/base64url.md)
- [Base64 padding](examples/base64-padding.md)
- [CLI examples](examples/cli-examples.md)
- [JavaScript examples](examples/javascript-examples.md)
- [Python examples](examples/python-examples.md)
- [Common errors](examples/common-errors.md)

## Contributing

Found a typo or want to add an example? See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE)
