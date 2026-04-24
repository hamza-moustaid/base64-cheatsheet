# Image to Base64

"Base64 image" usually means an image file encoded as a Base64 string, most often wrapped in a **data URI** so it can be used directly as a `src`.

## Data URI format

```
data:<mime-type>;base64,<base64-data>
```

Example:

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA
AAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO
9TXL0Y4OHwAAAABJRU5ErkJggg==
```

Common MIME types:

| Format | MIME type         |
| ------ | ----------------- |
| PNG    | `image/png`       |
| JPEG   | `image/jpeg`      |
| GIF    | `image/gif`       |
| WebP   | `image/webp`      |
| SVG    | `image/svg+xml`   |
| AVIF   | `image/avif`      |

## Using it in HTML

```html
<img
  src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA..."
  alt="pixel"
  width="16"
  height="16"
>
```

## Using it in CSS

```css
.logo {
  background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0i...");
}
```

## When to embed an image as Base64

Good fits:

- Small icons, bullets, or sprites (under ~5 KB each).
- Inline images in a single-file HTML export or email.
- Logos embedded in a generated PDF.
- Offline bundles where extra HTTP requests aren't practical.

## When not to

- **Large images.** The Base64 string is ~33% larger than the binary, and loading it blocks the HTML or CSS parser.
- **Images reused across many pages.** They can't be cached like a normal file — every page re-downloads them.
- **Content editors.** Large Base64 blobs in markdown or a CMS make diffs unreadable.

For anything bigger than a small icon, serve the image as a regular file.

## Generating a data URI

JavaScript (browser, from a file input):

```js
const [file] = document.querySelector("input[type=file]").files;
const reader = new FileReader();
reader.onload = () => {
  const dataUri = reader.result; // "data:image/png;base64,iVBORw0..."
};
reader.readAsDataURL(file);
```

Python:

```python
import base64, mimetypes

def to_data_uri(path):
    mime, _ = mimetypes.guess_type(path)
    with open(path, "rb") as f:
        b64 = base64.b64encode(f.read()).decode()
    return f"data:{mime};base64,{b64}"

print(to_data_uri("icon.png"))
```

CLI (macOS/Linux):

```sh
printf 'data:image/png;base64,%s' "$(base64 -w0 icon.png)"
```

On macOS the `base64` command does not split lines by default, so `-w0` is not needed — use just `base64 icon.png`.

## Sanity check

- The `<mime-type>` must match the actual image format. A PNG labeled as `image/jpeg` will fail to render.
- Don't include line breaks in the data URI unless you know your consumer tolerates them.
- SVGs can also be embedded as plain text (`data:image/svg+xml;utf8,...`), which is usually smaller than Base64.
