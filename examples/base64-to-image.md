# Base64 to image

Decoding a Base64 image string means turning the text back into binary bytes and writing those bytes to a file (or a `Blob` in the browser).

## Two common input shapes

**Raw Base64:**

```
iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI...
```

**Data URI:**

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACN...
```

Before decoding, strip the `data:<mime>;base64,` prefix if present. The MIME type tells you the right file extension.

| MIME type       | Extension |
| --------------- | --------- |
| `image/png`     | `.png`    |
| `image/jpeg`    | `.jpg`    |
| `image/gif`     | `.gif`    |
| `image/webp`    | `.webp`   |
| `image/svg+xml` | `.svg`    |
| `image/avif`    | `.avif`   |

## JavaScript (browser)

Convert a data URI to a `Blob` and trigger a download:

```js
function dataUriToBlob(uri) {
  const [meta, data] = uri.split(",");
  const mime = meta.match(/data:(.*?);base64/)[1];
  const binary = atob(data);
  const bytes = Uint8Array.from(binary, c => c.charCodeAt(0));
  return new Blob([bytes], { type: mime });
}

const blob = dataUriToBlob("data:image/png;base64,iVBORw0KGgo...");
const url = URL.createObjectURL(blob);

const a = document.createElement("a");
a.href = url;
a.download = "image.png";
a.click();
URL.revokeObjectURL(url);
```

## Node.js

```js
import { writeFile } from "node:fs/promises";

const dataUri = "data:image/png;base64,iVBORw0KGgo...";
const base64 = dataUri.split(",")[1] ?? dataUri;

await writeFile("image.png", Buffer.from(base64, "base64"));
```

## Python

```python
import base64, re

def save_image(data_uri_or_b64, out_path):
    m = re.match(r"data:(?P<mime>[^;]+);base64,(?P<data>.+)", data_uri_or_b64)
    data = m.group("data") if m else data_uri_or_b64
    with open(out_path, "wb") as f:
        f.write(base64.b64decode(data))

save_image("data:image/png;base64,iVBORw0KGgo...", "image.png")
```

## CLI (macOS/Linux)

If the string is raw Base64:

```sh
echo 'iVBORw0KGgo...' | base64 --decode > image.png
```

If it's a data URI, strip the prefix first:

```sh
echo 'data:image/png;base64,iVBORw0KGgo...' \
  | sed 's/^data:[^;]*;base64,//' \
  | base64 --decode > image.png
```

## Verifying the result

After decoding, check the first few bytes ("magic number") to confirm the format:

| Format | First bytes (hex)      |
| ------ | ---------------------- |
| PNG    | `89 50 4E 47 0D 0A 1A 0A` |
| JPEG   | `FF D8 FF`             |
| GIF    | `47 49 46 38` (`GIF8`) |
| WebP   | `52 49 46 46 .. .. .. .. 57 45 42 50` |

```sh
xxd image.png | head -1
```

If the magic bytes don't match the extension, either the input wasn't valid Base64, the MIME type was wrong, or the string was corrupted in transit (often by URL encoding or missing padding — see [base64-padding.md](base64-padding.md)).
