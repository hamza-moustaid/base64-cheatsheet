# Python examples

Python's standard-library `base64` module covers encoding, decoding, URL-safe variants, and more. No third-party package needed.

All functions in the module work with `bytes`, not `str`. Encode to bytes first, decode back to text at the end.

## Encoding strings

```python
import base64

base64.b64encode(b"Hello World").decode()
# "SGVsbG8gV29ybGQ="

# Non-ASCII: encode the string to UTF-8 bytes first
base64.b64encode("héllo 🌍".encode("utf-8")).decode()
# "aMOpbGxvIPCfjI0="
```

## Decoding strings

```python
base64.b64decode("SGVsbG8gV29ybGQ=").decode()
# "Hello World"

base64.b64decode("aMOpbGxvIPCfjI0=").decode("utf-8")
# "héllo 🌍"
```

If the source isn't valid UTF-8, `.decode()` raises `UnicodeDecodeError`. That's a clue you're actually holding binary data, not text.

## Encoding a file

```python
import base64

with open("photo.jpg", "rb") as f:
    encoded = base64.b64encode(f.read()).decode()

with open("photo.jpg.b64", "w") as f:
    f.write(encoded)
```

For very large files, stream the encoding so you don't load everything into memory:

```python
import base64

with open("big.bin", "rb") as src, open("big.bin.b64", "wb") as dst:
    base64.encode(src, dst)  # reads and writes in chunks
```

`base64.encode` writes line-wrapped output (76-char lines), which is what RFC 2045 expects. Use `b64encode` on the full bytes if you need a single unwrapped string.

## Decoding a file

```python
import base64

with open("photo.jpg.b64", "r") as f:
    data = base64.b64decode(f.read())

with open("photo.jpg", "wb") as f:
    f.write(data)
```

Streaming variant:

```python
import base64

with open("big.bin.b64", "rb") as src, open("big.bin", "wb") as dst:
    base64.decode(src, dst)
```

## URL-safe Base64

```python
import base64

base64.urlsafe_b64encode(b"subjects?").decode()
# "c3ViamVjdHM_"

base64.urlsafe_b64decode("c3ViamVjdHM_").decode()
# "subjects?"
```

For JWTs and other formats that omit padding:

```python
def b64url_encode(data: bytes) -> str:
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()

def b64url_decode(s: str) -> bytes:
    pad = "=" * (-len(s) % 4)
    return base64.urlsafe_b64decode(s + pad)

b64url_encode(b'{"sub":"12345"}')
# "eyJzdWIiOiIxMjM0NSJ9"

b64url_decode("eyJzdWIiOiIxMjM0NSJ9")
# b'{"sub":"12345"}'
```

## HTTP Basic auth

```python
import base64

user, password = "alice", "s3cret"
token = base64.b64encode(f"{user}:{password}".encode()).decode()

headers = {"Authorization": f"Basic {token}"}
```

## Building a data URI

```python
import base64, mimetypes

def to_data_uri(path: str) -> str:
    mime, _ = mimetypes.guess_type(path)
    with open(path, "rb") as f:
        b64 = base64.b64encode(f.read()).decode()
    return f"data:{mime};base64,{b64}"

print(to_data_uri("icon.png"))
```

## Strict decoding

By default, `b64decode` silently accepts non-alphabet characters. Pass `validate=True` to raise on unexpected input — useful when you want to reject corrupted tokens:

```python
base64.b64decode("SGVs bG8=", validate=False)  # ok, spaces silently ignored
base64.b64decode("SGVs bG8=", validate=True)   # binascii.Error: Invalid base64-encoded string
```
