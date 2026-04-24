# CLI examples

macOS and most Linux distributions ship a `base64` command. It handles both text and binary files through standard input, standard output, and pipes.

## Encoding text

```sh
printf 'Hello World' | base64
# SGVsbG8gV29ybGQ=
```

Prefer `printf` over `echo`. `echo` appends a newline, so `echo 'Hi' | base64` encodes `Hi\n`, not `Hi`.

If you really want to use `echo`, pass `-n`:

```sh
echo -n 'Hello World' | base64
```

## Decoding text

Linux (GNU coreutils):

```sh
echo 'SGVsbG8gV29ybGQ=' | base64 --decode
# Hello World
```

macOS (BSD):

```sh
echo 'SGVsbG8gV29ybGQ=' | base64 -D
```

The short form `-d` works on both.

## Encoding a file

```sh
# Encode to stdout
base64 photo.jpg

# Encode to a file
base64 photo.jpg > photo.jpg.b64
```

GNU `base64` wraps output at 76 columns by default. Disable wrapping with `-w 0`:

```sh
base64 -w 0 photo.jpg > photo.jpg.b64
```

macOS `base64` does not wrap by default, and does not support `-w`.

## Decoding to a file

```sh
base64 --decode photo.jpg.b64 > photo.jpg
```

Be careful: redirecting decoded **binary** to the terminal will print garbage and can mess up your shell. Always redirect to a file.

## One-liners

Encode a file to a data URI:

```sh
printf 'data:image/png;base64,%s' "$(base64 -w0 icon.png)"
# macOS: printf 'data:image/png;base64,%s' "$(base64 icon.png)"
```

HTTP Basic auth header:

```sh
printf 'user:pass' | base64
# dXNlcjpwYXNz
curl -H "Authorization: Basic $(printf 'user:pass' | base64)" https://example.com
```

Inspect a JWT payload (Base64URL, no padding — re-pad first):

```sh
jwt='eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NSJ9.xxx'
payload=$(echo "$jwt" | cut -d. -f2 | tr '_-' '/+')
pad=$(( (4 - ${#payload} % 4) % 4 ))
printf '%s%s' "$payload" "$(printf '%*s' $pad '' | tr ' ' '=')" | base64 --decode
# {"sub":"12345"}
```

## Windows PowerShell

PowerShell doesn't ship a `base64` command; use `[Convert]`:

```powershell
# Encode
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("Hello World"))
# SGVsbG8gV29ybGQ=

# Decode
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("SGVsbG8gV29ybGQ="))
# Hello World

# Encode a file
[Convert]::ToBase64String([IO.File]::ReadAllBytes("photo.jpg")) | Out-File photo.jpg.b64

# Decode a file
[IO.File]::WriteAllBytes("photo.jpg", [Convert]::FromBase64String((Get-Content photo.jpg.b64)))
```

## Windows cmd / `certutil`

The legacy `certutil` tool can also do it:

```cmd
certutil -encode input.txt output.b64
certutil -decode output.b64 input.txt
```

`certutil` wraps output in `-----BEGIN CERTIFICATE-----` markers by default. Strip those if you need just the Base64 payload.

## Troubleshooting

- **"invalid input" on decode.** Usually padding or whitespace. Strip whitespace and re-pad:
  ```sh
  tr -d '[:space:]' < in.b64 | base64 --decode
  ```
- **Unexpected trailing newline in encoded output.** You used `echo` without `-n`, or your file ends with a newline. Use `printf` or a file without trailing newline.
- **`+` or `/` breaking in a URL.** Use Base64URL instead. See [base64url.md](base64url.md).
