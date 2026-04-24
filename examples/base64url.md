# Base64URL

Standard Base64 uses `+`, `/`, and `=`, which all cause problems inside URLs and filenames:

- `+` is interpreted as a space in query strings.
- `/` is a path separator.
- `=` is reserved in query strings and sometimes stripped.

**Base64URL** (defined in [RFC 4648 §5](https://datatracker.ietf.org/doc/html/rfc4648#section-5)) solves this with a URL-safe alphabet and usually drops padding.

## Character differences

| Standard Base64 | Base64URL |
| --------------- | --------- |
| `+`             | `-`       |
| `/`             | `_`       |
| `=` (padding)   | often omitted |

The rest of the alphabet (`A–Z`, `a–z`, `0–9`) is identical.

## Example

Encoding the bytes of the string `subjects?`:

| Variant      | Output            |
| ------------ | ----------------- |
| Standard     | `c3ViamVjdHM/`    |
| Base64URL    | `c3ViamVjdHM_`    |
| Base64URL (no padding) | `c3ViamVjdHM_` |

Here only the last character differs because this particular input happens to produce a `/`. The same swap applies anywhere `+` or `/` would appear.

## JWTs

A JSON Web Token looks like this:

```
<header>.<payload>.<signature>
```

All three segments are Base64URL-encoded **without padding**. That's why you'll often see tokens like:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NSJ9.Sflk...
```

Decoding it with a standard Base64 decoder will either fail or produce garbage unless you first:

1. Replace `-` with `+` and `_` with `/`.
2. Add `=` padding until the length is a multiple of 4.

## JavaScript helpers

```js
function toBase64Url(str) {
  return btoa(str)
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=+$/, "");
}

function fromBase64Url(b64url) {
  let s = b64url.replace(/-/g, "+").replace(/_/g, "/");
  // Re-pad
  while (s.length % 4) s += "=";
  return atob(s);
}

toBase64Url("subjects?");      // "c3ViamVjdHM_"
fromBase64Url("c3ViamVjdHM_"); // "subjects?"
```

For Unicode input, wrap these with the `TextEncoder`/`TextDecoder` helpers from [javascript-examples.md](javascript-examples.md).

## Python helpers

```python
import base64

base64.urlsafe_b64encode(b"subjects?").decode()
# "c3ViamVjdHM_"

base64.urlsafe_b64decode("c3ViamVjdHM_").decode()
# "subjects?"

# Without padding (common for JWTs)
def b64url_encode(data: bytes) -> str:
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()

def b64url_decode(s: str) -> bytes:
    pad = "=" * (-len(s) % 4)
    return base64.urlsafe_b64decode(s + pad)
```

## When to use which

| Use case                            | Variant         |
| ----------------------------------- | --------------- |
| Email body, JSON field, config file | Standard Base64 |
| URL query parameter                 | Base64URL       |
| Filename, S3 key                    | Base64URL       |
| JWT / OAuth / WebAuthn              | Base64URL, no padding |
| HTTP Basic auth header              | Standard Base64 |

When in doubt, check the spec for the format you're targeting.
