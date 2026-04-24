# Common errors

A walkthrough of mistakes that bite teams using Base64, and how to fix each one.

## 1. Treating Base64 as encryption

```
"admin:password" → "YWRtaW46cGFzc3dvcmQ="
```

Anyone can paste that into a decoder and get the original string back. Base64 is an **encoding**, not a cipher. It does not hide data, authenticate senders, or protect against tampering.

If you need secrecy, use actual cryptography: TLS for transport, a KMS-managed key for storage, AES-GCM or libsodium's `crypto_secretbox` for payload encryption.

## 2. Missing padding

Standard decoders insist on a length that's a multiple of 4:

```python
base64.b64decode("SGVsbG8")        # binascii.Error: Incorrect padding
base64.b64decode("SGVsbG8=")       # b"Hello"
```

Fix by re-padding before decoding:

```python
def decode_loose(s: str) -> bytes:
    return base64.b64decode(s + "=" * (-len(s) % 4))
```

Same idea in JavaScript:

```js
const padded = s + "=".repeat((4 - (s.length % 4)) % 4);
atob(padded);
```

## 3. Wrong character set (Base64 vs Base64URL)

A token like `eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NSJ9.xxx` contains `-` and `_` characters in real payloads. Passing it to a **standard** Base64 decoder either errors out or produces wrong bytes.

Convert first:

```js
const standard = urlSafe.replace(/-/g, "+").replace(/_/g, "/");
```

Or use the URL-safe function directly (`base64.urlsafe_b64decode` in Python, `"base64url"` in Node).

## 4. Mixing up Base64 and Base64URL

Telltale signs:

- You see `-` or `_` in the string → it's Base64URL.
- You see `+` or `/` in the string → it's standard Base64.
- No padding and length not a multiple of 4 → almost certainly Base64URL.

Don't guess; check the spec for the format you're consuming. See [base64url.md](base64url.md).

## 5. Decoding binary as text

```python
data = base64.b64decode(encoded_image)
data.decode("utf-8")  # UnicodeDecodeError
```

If the Base64 was encoding a PNG, protobuf, zip, or any other binary format, `.decode()` will fail or return garbled output. Write it to a file or keep it as `bytes`:

```python
with open("out.png", "wb") as f:
    f.write(data)
```

In JavaScript, use `Uint8Array` or `Blob` instead of a string.

## 6. Whitespace and line breaks in input

Base64 strings copied from email, PEM files, or pretty-printed logs often contain newlines or stray spaces. Some decoders tolerate them; others don't.

Strip whitespace before decoding:

```python
import re
clean = re.sub(r"\s+", "", encoded)
base64.b64decode(clean)
```

```js
const clean = encoded.replace(/\s+/g, "");
atob(clean);
```

## 7. `btoa` throwing on non-ASCII input

```js
btoa("héllo"); // InvalidCharacterError
```

`btoa` only accepts Latin-1 characters. Use the Unicode-safe helper from [javascript-examples.md](javascript-examples.md) instead.

## 8. `echo` adding a trailing newline

```sh
echo 'Hello' | base64
# SGVsbG8K  ← note the trailing "K" — that's an encoded "\n"
```

Use `printf` or `echo -n`:

```sh
printf 'Hello' | base64
# SGVsbG8=
```

## 9. Very large Base64 strings

Base64 output is ~33% larger than the input. Embedding a 5 MB image as a data URI means:

- Your HTML or CSS grows by ~6.7 MB.
- Browsers cannot cache it separately.
- Parsing and rendering stall on the main thread.
- JSON responses become huge, and many APIs apply payload limits.

For anything bigger than a small icon, serve the file normally and reference it by URL.

## 10. Wrong MIME type in a data URI

```
data:image/jpeg;base64,iVBORw0KGgo...
```

That payload starts with `iVBORw0KGgo`, which decodes to the PNG magic bytes `89 50 4E 47`. Browsers may refuse to render it, or guess the real type and log a warning.

Always match the MIME to the actual bytes. Programmatically, derive it from the source file:

```python
import mimetypes
mime, _ = mimetypes.guess_type(path)
```

## 11. Copy/paste mangling in URLs

Pasting a standard Base64 string into a URL turns `+` into a space, sometimes silently. If a Base64 string will ever travel through a URL, **use Base64URL** from the start. Retrofitting is error-prone.

## 12. Double-encoding

```python
token = base64.b64encode(b"Hello").decode()
# "SGVsbG8="

# Then later, by accident:
wrapped = base64.b64encode(token.encode()).decode()
# "U0dWc2JHOD0="   ← this is Base64 of "SGVsbG8="
```

The result is valid Base64 but represents the Base64 **text**, not the original bytes. Decoding it once just gives you back `"SGVsbG8="` and a confused debugger. Trace your pipeline and encode exactly once.
