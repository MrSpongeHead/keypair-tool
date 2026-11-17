# Key Pair Tool

Generate, convert and recreate cryptographic key pairs in your browser: SSH keys, PEM, DER, JWK and PuTTY `.ppk`.
Everything runs locally using your browser's built-in Web Crypto. Nothing is uploaded, and the page is blocked from
making network connections at all.

**Use it online:** https://mrspongehead.github.io/keypair-tool/
**Use it offline:** click *Download for offline use* on the page, or download `keypair-tool.html` from the
[latest release](https://github.com/MrSpongeHead/keypair-tool/releases/latest), and open it in your browser.

It's a single HTML file with no libraries, no build step and no tracking.

---

## Generate

Creates a new key pair and shows it in any of the output formats, or all of them at once.

| Algorithm | Approx. security | Rating |
|---|---|---|
| Ed25519 | 128-bit | Recommended |
| ECDSA P-256 | 128-bit | Good |
| ECDSA P-384 | 192-bit | Strong |
| ECDSA P-521 | 256-bit | Very strong |
| RSA 2048 | 112-bit | Minimum / legacy |
| RSA 3072 | 128-bit | Good |
| RSA 4096 | ~140-bit (estimate) | Strong |

Security levels are classical estimates based on NIST's standard equivalences. None of these algorithms would resist a
large future quantum computer. Ed25519 is switched off automatically in browsers that don't support it.

## Convert or recreate

Paste a private key, open a file, or drop one on the page. The tool works out the format and shows the algorithm,
the fingerprint (in the same `SHA256:…` form as `ssh-keygen -l`) and any comment stored in the file. It can then:

- **recreate the public key**, for example when you've lost an `id_ed25519.pub` or need the public key in another format
- **convert the whole key pair** to another format, for example OpenSSH to PuTTY `.ppk` or the other way round

It reads:

- PEM: PKCS#8 (`BEGIN PRIVATE KEY`), PKCS#1 (`BEGIN RSA PRIVATE KEY`), SEC1 (`BEGIN EC PRIVATE KEY`)
- DER (binary), as a file or pasted as Base64
- JWK, a single key or a key set
- OpenSSH (`BEGIN OPENSSH PRIVATE KEY`)
- PuTTY `.ppk` versions 2 and 3

Before showing anything, it checks that the key is intact. It signs a test message with the private key and verifies
it with the recreated public key, compares that with any public key stored in the file, and checks the integrity
code (MAC) of `.ppk` files. Damaged or edited files are refused with an explanation.

## Output formats

| Format | Files | Typical use |
|---|---|---|
| PEM (PKCS#8 / SPKI) | `private_key.pem`, `public_key.pem` | OpenSSL, web servers, most programming languages |
| PEM (PKCS#1) | `rsa_private_key.pem`, `rsa_public_key.pem` | Older tools expecting `BEGIN RSA PRIVATE KEY` (RSA only) |
| DER | `private_key.der`, `public_key.der` | Java and some embedded systems |
| JWK | `private_key.jwk.json`, `public_key.jwk.json` | Web apps, JWTs, OAuth / OpenID Connect |
| OpenSSH | `id_ed25519` / `id_ecdsa` / `id_rsa` and `.pub` | SSH (`~/.ssh`, `authorized_keys`) |
| PuTTY `.ppk` v3 | `<name>.ppk` | PuTTY and WinSCP (PuTTY 0.75 or later) |

If you enter a comment or label, it's used in the PEM, DER, JWK and PuTTY file names (for example
`github-work_ed25519_private.pem`). OpenSSH files always keep the default names, so `ssh` finds them without extra
configuration.

## Privacy and security

- **Nothing leaves your browser.** Keys are generated with the browser's Web Crypto API, using its secure random
  number generator. They are never stored or sent anywhere.
- **The page can't connect to the network.** The file includes a
  [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/Security/CSP) that the browser
  enforces. It blocks every kind of connection (fetch, XHR, WebSocket, beacons), loading scripts, styles, fonts or
  images from elsewhere, form submissions and embedded frames. Only the code inside the file can run. You can check
  this yourself: the policy is on line 8 of `index.html`, and your browser's developer tools (Network tab) will show
  no requests apart from loading the page.
- **Private keys are output without a passphrase.** Store them safely. To add a passphrase afterwards, use
  `ssh-keygen -p -f <key>` for OpenSSH keys or PuTTYgen for `.ppk` files.
- **What the policy can't protect against:** browser extensions aren't bound by a page's policy, so use a browser
  profile you trust. A tampered copy of the file could have the policy removed, which is why each release lists a
  checksum (below).

### Checking a downloaded copy

Each [release](https://github.com/MrSpongeHead/keypair-tool/releases) lists the SHA-256 checksum of
`keypair-tool.html`. To check your copy:

- **Windows (PowerShell):** `Get-FileHash keypair-tool.html -Algorithm SHA256`
- **macOS:** `shasum -a 256 keypair-tool.html`
- **Linux:** `sha256sum keypair-tool.html`

The result should match the checksum in the release notes.

## Browser support

Built for current versions of Chrome, Edge, Firefox and Safari, and tested in Chrome. Web Crypto only works on secure pages, so use the
`https://` site, `http://localhost`, or a downloaded copy opened from your computer. If the browser blocks Web Crypto,
the page says so.

## Not included (deliberately)

- **Passphrase-protected keys**, as input or output. Encrypted PuTTY keys need Argon2 and encrypted OpenSSH keys need
  bcrypt, neither of which browsers provide. Remove the passphrase from a copy first; the page tells you how.
- **Other algorithms:** Ed448, X25519, secp256k1, Brainpool curves, DSA and RSA-PSS keys.
- **Certificates, CSRs and hardware (FIDO / smart card) keys.**

## How it was tested

- Keys made by `ssh-keygen` (OpenSSH 9.6), `openssl` (3.0) and `puttygen` (0.81) in every supported input format
  were recreated and converted. Each recreated public key and fingerprint matched what the original tool reports, and
  every converted private key loaded in all three tools and produced signatures that verified against the original
  public key.
- Passphrase-protected, public-only, unsupported, damaged and edited files were all refused with clear messages.
- Generated keys in every algorithm and format were checked with `ssh-keygen -y`, `openssl pkey -check` and `puttygen`,
  including edge cases such as Ed25519 keys ending in zero bytes.
- The page was tested in Chrome, both from a web server and opened from disk, including the network block: every
  attempt to send data out was refused.

## Licence

[MIT](LICENSE). Not affiliated with OpenSSH, OpenSSL or PuTTY.
