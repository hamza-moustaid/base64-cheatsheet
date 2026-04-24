# Text to Base64

Converting plain text to Base64 is the most common Base64 task. The text is first turned into bytes (usually UTF-8), and those bytes are then encoded using the Base64 alphabet.

## Examples

| Text                         | Base64                                 |
| ---------------------------- | -------------------------------------- |
| `Hello World`                | `SGVsbG8gV29ybGQ=`                     |
| `{"user":"ada","admin":true}`| `eyJ1c2VyIjoiYWRhIiwiYWRtaW4iOnRydWV9` |
| `sk_test_4eC39HqLy...`       | `c2tfdGVzdF80ZUMzOUhxTHkuLi4=`         |

The third row is a dummy API-key-shaped string. Note that Base64-encoding a token does **not** protect it — see [common-errors.md](common-errors.md).

## JavaScript

```js
// ASCII / Latin-1 text
btoa("Hello World");
// "SGVsbG8gV29ybGQ="

// JSON
const body = JSON.stringify({ user: "ada", admin: true });
btoa(body);
// "eyJ1c2VyIjoiYWRhIiwiYWRtaW4iOnRydWV9"

// Unicode-safe (emoji, accents, non-Latin scripts)
function encodeUtf8(str) {
  const bytes = new TextEncoder().encode(str);
  return btoa(String.fromCharCode(...bytes));
}
encodeUtf8("héllo 🌍"); // "aMOpbGxvIPCfjI0="
```

`btoa` will throw on characters outside Latin-1, which is why the Unicode-safe helper is needed for anything beyond ASCII.

## Python

```python
import base64

# Text
base64.b64encode("Hello World".encode()).decode()
# "SGVsbG8gV29ybGQ="

# JSON
import json
body = json.dumps({"user": "ada", "admin": True}, separators=(",", ":"))
base64.b64encode(body.encode()).decode()
# "eyJ1c2VyIjoiYWRhIiwiYWRtaW4iOnRydWV9"

# API token
token = "sk_test_4eC39HqLy..."
base64.b64encode(token.encode()).decode()
```

## CLI

```sh
printf 'Hello World' | base64
# SGVsbG8gV29ybGQ=

printf '{"user":"ada","admin":true}' | base64
# eyJ1c2VyIjoiYWRhIiwiYWRtaW4iOnRydWV9
```

Use `printf` instead of `echo`; `echo` appends a trailing newline that changes the encoded output.

If the input is multi-line, either quote it or pipe from a file:

```sh
base64 <<< 'line 1
line 2'

base64 notes.txt
```

## Notes

- Base64 output is always a multiple of 4 characters — trailing `=` characters are padding, not part of the data.
- The encoded string is ~33% larger than the input.
- Different encodings of the same text (UTF-8 vs UTF-16) produce different Base64 output. Stick to UTF-8 unless you have a specific reason not to.
