# Base64 to text

Decoding turns a Base64 string back into bytes. If those bytes represent text (usually UTF-8), you decode them once more to get a readable string.

## Examples

| Base64                                 | Text                          |
| -------------------------------------- | ----------------------------- |
| `SGVsbG8gV29ybGQ=`                     | `Hello World`                 |
| `eyJvayI6dHJ1ZX0=`                     | `{"ok":true}`                 |
| `aMOpbGxvIPCfjI0=`                     | `héllo 🌍`                    |

## JavaScript

```js
atob("SGVsbG8gV29ybGQ=");
// "Hello World"

// Unicode-safe
function decodeUtf8(b64) {
  const binary = atob(b64);
  const bytes = Uint8Array.from(binary, c => c.charCodeAt(0));
  return new TextDecoder().decode(bytes);
}
decodeUtf8("aMOpbGxvIPCfjI0="); // "héllo 🌍"
```

`atob` alone will not decode emoji or accented characters correctly — it returns a Latin-1 string, so run the bytes through `TextDecoder` for real UTF-8 text.

## Python

```python
import base64

base64.b64decode("SGVsbG8gV29ybGQ=").decode()
# "Hello World"

base64.b64decode("aMOpbGxvIPCfjI0=").decode()
# "héllo 🌍"
```

`b64decode` returns `bytes`. Call `.decode()` (defaults to UTF-8) to get a string.

## CLI

```sh
echo 'SGVsbG8gV29ybGQ=' | base64 --decode
# Hello World

# macOS short flag
echo 'SGVsbG8gV29ybGQ=' | base64 -D
```

If the decoded data is binary, redirect to a file rather than printing it:

```sh
echo 'iVBORw0KGgo...' | base64 --decode > image.png
```

## Things that break decoding

- **Whitespace or line breaks** in the middle of the string. Some decoders accept them, some don't. Strip them first with `.replace(/\s/g, "")` or `re.sub(r"\s", "", s)`.
- **Missing padding.** Either add `=` back until the length is a multiple of 4, or use a decoder that tolerates missing padding.
- **Base64URL input** decoded with a standard decoder. Convert `-` → `+` and `_` → `/` first.
- **Treating binary output as text.** If the Base64 encoded a PNG or a protobuf, decoding the result as UTF-8 will fail or produce garbage.

See [common-errors.md](common-errors.md) for more.
