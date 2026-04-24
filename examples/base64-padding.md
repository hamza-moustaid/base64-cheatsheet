# Base64 padding

Base64 output length is always a multiple of 4 characters. When the input byte count isn't a multiple of 3, the encoder appends `=` characters so the output still lines up.

## Why it exists

Base64 encodes **3 input bytes → 4 output characters** (24 bits → 4 × 6 bits). If the input has 1 or 2 leftover bytes, the last group is incomplete and is padded:

| Input bytes | Full groups | Leftover | Padding | Example        |
| ----------- | ----------- | -------- | ------- | -------------- |
| 3           | 1           | 0        | none    | `ABC` → `QUJD` |
| 2           | 0           | 2        | `=`     | `AB`  → `QUI=` |
| 1           | 0           | 1        | `==`    | `A`   → `QQ==` |

You will **never** see `===`. Three equals would imply zero useful bits in the final group, which never happens.

## Valid padding

| Base64            | Valid? | Notes                                 |
| ----------------- | ------ | ------------------------------------- |
| `QUJD`            | ✅     | 4 chars, no padding needed            |
| `QUI=`            | ✅     | 1 pad char                            |
| `QQ==`            | ✅     | 2 pad chars                           |
| `SGVsbG8=`        | ✅     | decodes to `Hello`                    |

## Invalid padding

| Base64        | Why it's wrong                                        |
| ------------- | ----------------------------------------------------- |
| `QQ=`         | Length 3 — must be multiple of 4                      |
| `QQ===`       | Length 5 — must be multiple of 4                      |
| `=QUJD`       | `=` may only appear at the end                        |
| `QUJ=D`       | Padding can't sit in the middle                        |
| `SGVsbG8`     | Missing final `=` for strict decoders                 |

## "Padding missing" errors

Many real-world systems (JWTs, URL parameters) deliberately omit padding. Standard decoders then complain with errors like:

- Python: `binascii.Error: Incorrect padding`
- Java: `IllegalArgumentException: Input byte array has wrong 4-byte ending unit`
- Node: silently returns wrong bytes

Fix it by re-padding before decoding:

**Python:**

```python
import base64

def decode_loose(s: str) -> bytes:
    s += "=" * (-len(s) % 4)
    return base64.b64decode(s)
```

**JavaScript:**

```js
function decodeLoose(s) {
  while (s.length % 4) s += "=";
  return atob(s);
}
```

Both add just enough `=` characters to reach the next multiple of 4.

## Should I strip padding?

- **Standard Base64** — keep padding. It's required by strict decoders.
- **Base64URL in JWTs, WebAuthn, OAuth** — strip it. The spec omits it by convention.
- **Base64URL in a filename or URL** — usually strip it, but check the target system's rules.

When you strip padding, make sure the consumer on the other end re-pads before decoding, or uses a tolerant decoder.

## Quick check

You can tell from the length alone how much padding a string should have:

```
length % 4 == 0 → no padding needed
length % 4 == 2 → add "=="
length % 4 == 3 → add "="
length % 4 == 1 → invalid, not a Base64 string
```
