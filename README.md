# ooshare-action

Create and reveal [ooshare](https://ooshare.io) one-time secrets right from your GitHub Actions workflows. Uses the official `ooshare` CLI — the same end-to-end AES-256-GCM encryption as the web, zero-knowledge, self-destructing links.

## Usage

### Create a secret and capture its URL

```yaml
steps:
  - uses: dhdtech/ooshare-action@v1
    id: share
    with:
      command: create
      text: ${{ secrets.MY_SECRET }}   # use a GitHub secret, never a literal
      ttl: 24

  - name: Use the share URL
    run: echo "Share this: ${{ steps.share.outputs.url }}"
```

The `url` output contains the decryption key — treat it as sensitive (post it to a private channel, not a public log).

### Create from a file

```yaml
- uses: dhdtech/ooshare-action@v1
  id: share
  with:
    command: create
    file: ./contract.pdf
```

### Reveal a secret

```yaml
- uses: dhdtech/ooshare-action@v1
  id: reveal
  with:
    command: view
    url: ${{ steps.share.outputs.url }}

- name: Use the secret
  run: echo "${{ steps.reveal.outputs.text }}" > secret.txt
```

The secret is deleted from the server on first view — reveal it only once.

### Reveal a secret that has a file attachment

```yaml
- uses: dhdtech/ooshare-action@v1
  id: reveal
  with:
    command: view
    url: ${{ steps.share.outputs.url }}
    output: ./attachments   # directory, exact path, or '-' for stdout

- name: Use the attachment
  run: |
    echo "attachment at: ${{ steps.reveal.outputs.attachment }}"
    cat "${{ steps.reveal.outputs.attachment }}"
```

### Create with a file + viewer language

```yaml
- uses: dhdtech/ooshare-action@v1
  with:
    command: create
    text: ${{ secrets.NOTE }}
    file: ./contract.pdf
    ttl: 48
    lang: es          # recipient sees the page in Spanish
```

## Inputs

| Name | Description | Default |
|---|---|---|
| `command` | `create` or `view` | — |
| `text` | Secret text (create) — use a GitHub secret | `''` |
| `file` | Path to a file to attach (create) | `''` |
| `ttl` | Expiry in hours (1–72) | `24` |
| `lang` | Viewer language for the link (`en`, `zh`, `es`, `hi`, `ar`, `pt`) | `en` |
| `url` | Share URL to reveal (view) | `''` |
| `output` | Where to write a decoded attachment (view): dir, path, or `-` | `''` |
| `version` | ooshare CLI release version | `v1.0.3` |
| `api-url` | API base URL | `https://api.ooshare.io` |
| `origin` | Site origin for the share URL | `https://ooshare.io` |

## Outputs

| Name | Description |
|---|---|
| `url` | The one-time share URL (create) — **sensitive** (contains the key) |
| `id` | The secret ID (create) |
| `has_attachment` | `true`/`false` — whether a file was attached (create) |
| `text` | The revealed secret (view) |
| `attachment` | Path where the decoded file was written (view) |
| `attachment-mime` | MIME type of the decoded file (view) |

## How it works

The action downloads the `ooshare` binary for your runner (Linux/macOS/Windows × amd64/arm64) from the [ooshare.io releases](https://github.com/dhdtech/ooshare.io/releases), runs `ooshare create` or `ooshare view` with JSON output, and exposes the URL / secret as outputs. No plaintext ever reaches the ooshare server — encryption happens in the binary, the master key travels only in the URL fragment.

## Security

- Never hardcode secret text in your workflow file — use `${{ secrets.* }}`.
- The `url` output embeds the decryption key. Share it over a private channel.
- Secrets are one-time: after `view`, the server deletes them.

## License

[MIT](LICENSE)
