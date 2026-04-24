# JavaScript examples

The browser and Node.js handle Base64 differently. Browsers have `btoa`/`atob`; Node has `Buffer`. This page covers both.

## `btoa` and `atob` (browser)

```js
btoa("Hello");        // "SGVsbG8="
atob("SGVsbG8=");     // "Hello"
```

Both work on **Latin-1 strings**, not arbitrary Unicode. Calling `btoa("héllo")` throws `InvalidCharacterError`.

## Unicode-safe encoding and decoding

Use `TextEncoder`/`TextDecoder` to convert between strings and UTF-8 bytes:

```js
function encodeBase64Utf8(str) {
  const bytes = new TextEncoder().encode(str);
  let binary = "";
  for (const b of bytes) binary += String.fromCharCode(b);
  return btoa(binary);
}

function decodeBase64Utf8(b64) {
  const binary = atob(b64);
  const bytes = Uint8Array.from(binary, c => c.charCodeAt(0));
  return new TextDecoder().decode(bytes);
}

encodeBase64Utf8("héllo 🌍");  // "aMOpbGxvIPCfjI0="
decodeBase64Utf8("aMOpbGxvIPCfjI0="); // "héllo 🌍"
```

The intermediate `binary` string is a trick — it represents each byte as a single Latin-1 code unit so `btoa` accepts it.

## File to Base64 with `FileReader`

```js
function fileToBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result); // "data:...;base64,..."
    reader.onerror = () => reject(reader.error);
    reader.readAsDataURL(file);
  });
}

// Usage with an <input type="file">
document.querySelector("input").addEventListener("change", async (e) => {
  const dataUri = await fileToBase64(e.target.files[0]);
  console.log(dataUri);
});
```

To get only the Base64 (no `data:...;base64,` prefix):

```js
const base64 = dataUri.split(",")[1];
```

## Base64URL helpers

```js
const toBase64Url = (b64) =>
  b64.replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/, "");

const fromBase64Url = (b64url) => {
  const s = b64url.replace(/-/g, "+").replace(/_/g, "/");
  return s + "=".repeat((4 - (s.length % 4)) % 4);
};

// Encode a string to Base64URL
const b64url = toBase64Url(encodeBase64Utf8("héllo 🌍"));

// Decode Base64URL back to a string
const text = decodeBase64Utf8(fromBase64Url(b64url));
```

## Node.js (`Buffer`)

Node doesn't have `btoa`/`atob` traditionally (though they were added as globals in Node 16+). The idiomatic tool is `Buffer`:

```js
// Encode
Buffer.from("Hello").toString("base64");              // "SGVsbG8="
Buffer.from("héllo 🌍", "utf8").toString("base64");   // "aMOpbGxvIPCfjI0="

// Decode
Buffer.from("SGVsbG8=", "base64").toString("utf8");   // "Hello"

// Base64URL
Buffer.from("héllo 🌍").toString("base64url");        // "aMOpbGxvIPCfjI0"
Buffer.from("aMOpbGxvIPCfjI0", "base64url").toString("utf8");
```

`"base64url"` as an encoding name is supported in Node 16+.

## Modern browsers: `Uint8Array.toBase64()`

Recent browsers support native methods on typed arrays:

```js
new Uint8Array([72, 105]).toBase64();          // "SGk="
Uint8Array.fromBase64("SGk=");                 // Uint8Array [72, 105]

// Base64URL variant
new Uint8Array([72, 105]).toBase64({ alphabet: "base64url", omitPadding: true });
```

Check [caniuse](https://caniuse.com/?search=toBase64) before relying on these — availability is still uneven. The `btoa`/`atob` + `TextEncoder` approach works everywhere.
