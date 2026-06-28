# 🔐 AES-GCM Secure Messenger

> Client-side AES-256-GCM encryption. Messages travel as URLs. Nothing touches a server.

---

## Contents

- [Quick Start](#quick-start)
- [How It Works](#how-it-works)
- [Features](#features)
- [Security Model](#security-model)
- [PIN & Session Protection](#pin--session-protection)
- [Passphrase Mode](#passphrase-mode)
- [Self-Destruct Links](#self-destruct-links)
- [File Encryption](#file-encryption)
- [Sharing Options](#sharing-options)
- [Settings & Customisation](#settings--customisation)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [URL Parameter Reference](#url-parameter-reference)
- [Cryptographic Details](#cryptographic-details)
- [Browser Compatibility](#browser-compatibility)
- [Limitations](#limitations)

---

## Quick Start

1. Open `aes-gcm-secure-messenger.html` in any modern browser.
2. Create a **4–8 digit PIN** on first launch.
3. Type a message (or drop a file) and click **Encrypt & Generate Link**.
4. Share the link. The recipient opens it and the message decrypts automatically.

No installation. No build step. No account. Just open the file.

---

## How It Works

```
Your browser          AES-256-GCM            Shareable URL
(plaintext)    ──▶   Key + IV + CT    ──▶   ?k=…&i=…&c=…
                                                   │
                                                   ▼
                                         Recipient's browser
                                         decrypts locally
```

Every message gets a **fresh 256-bit key** and a **random 96-bit IV**. Both are Base64URL-encoded into the link alongside the ciphertext. The GCM authentication tag is appended to the ciphertext and verified on decryption — any tampering causes an immediate failure.

When the recipient opens the link, the app reads the parameters from the URL, reconstructs the key, and decrypts locally. No network request is made.

---

## Features

| Feature | Description |
|---|---|
| 🔑 AES-256-GCM | Industry-standard authenticated encryption with integrity verification |
| 🌐 Zero-server | Everything runs in your browser; no backend, no analytics, no logs |
| 🔗 Shareable links | Recipients decrypt by opening the URL — no app install required |
| 📋 Auto-copy | Encrypted link is copied to clipboard automatically after encryption |
| 🔒 PIN protection | 4–8 digit PIN guards local access; hash stored in `localStorage` |
| 🔐 Passphrase mode | Optional PBKDF2-derived key from a shared passphrase (key excluded from URL) |
| ⏳ Self-destruct | Links can expire after 5 minutes, 1 hour, 24 hours, or 7 days |
| 📎 File encryption | Drop any file to encrypt its contents into the URL |
| 💬 Share sheet | One-click sharing via WhatsApp, SMS, or clipboard |
| 🔳 QR code | Generate a QR code for the encrypted link |
| 🔓 Auto-decrypt | Opening a link directly switches to Decrypt and unlocks the message |
| 🎨 Themes | Cyber (default), Midnight, and Light themes with system-aware transitions |
| 🔇 Sound effects | Optional audio feedback using the Web Audio API |
| 🖥️ Auto-lock | Session locks after 5 minutes of inactivity; PIN required to resume |

---

## Security Model

### What this tool provides

- **Confidentiality:** The message is encrypted before the URL is shared.
- **Integrity:** AES-GCM authentication detects any tampering with the ciphertext.
- **No server exposure:** Plaintext never leaves the browser.
- **PIN isolation:** The PIN protects local access to the tool but is separate from message encryption.

### What this tool does NOT provide

- **Key secrecy in standard mode:** The decryption key is embedded in the URL. Anyone who receives the link can decrypt the message — treat the link as the secret.
- **Forward secrecy:** Once shared, a link persists until the recipient deletes it.
- **Server-side expiry enforcement:** Self-destruct timers are checked client-side only. A recipient who saved the raw URL parameters before expiry can still decrypt.
- **PIN-based message encryption:** The PIN only controls UI access on the local device.

### Threat model

| Threat | Mitigated? | Notes |
|---|---|---|
| Server reads message | ✅ Yes | No server involved |
| Network intercept of message body | ✅ Yes | Message never sent to a server |
| Link intercepted in transit | ⚠️ Partial | Use passphrase mode + share link and passphrase separately |
| Recipient device compromised | ❌ No | Out of scope |
| Browser history / logs expose URL | ⚠️ Partial | URL contains the key in standard mode |
| Tampering with ciphertext | ✅ Yes | GCM auth tag detects modification |

### Best practices

- Share links only over trusted channels (Signal, encrypted email, in-person).
- For highly sensitive content, enable **Passphrase mode** and share the passphrase through a separate channel from the link.
- Treat the link itself as the secret in standard mode — anyone with it can read the message.

---

## PIN & Session Protection

| Action | Behaviour |
|---|---|
| First launch | Prompts to create a 4–8 digit PIN |
| Return visit | Requires PIN to unlock the tool |
| Change PIN | Available in Settings; takes effect immediately |
| Auto-lock | Locks after 5 minutes of inactivity if enabled in Settings |
| Reset | Clears all `localStorage` data and reloads — **irreversible** |
| Forgot PIN | Press `Ctrl + Shift + R` to hard-reset (all data lost) |

The PIN is hashed with SHA-256 and stored in `localStorage`. It cannot be recovered. There is no "forgot PIN" flow by design.

---

## Passphrase Mode

Enable **Use Passphrase** in the Encrypt tab before encrypting.

In passphrase mode:
- The key is derived from your passphrase using **PBKDF2** (100,000 iterations, SHA-256, 16-byte random salt).
- The derived key is **not** embedded in the URL. The link contains only the salt (`s`), IV (`i`), and ciphertext (`c`).
- Anyone who intercepts the link cannot decrypt without also knowing the passphrase.
- The recipient must enter the same passphrase in the Decrypt tab before clicking Decrypt.

This makes passphrase mode significantly stronger for sensitive content — share the link and passphrase through separate channels.

---

## Self-Destruct Links

Select an expiry window in the **Self-Destruct** dropdown before encrypting:

| Option | Expires after |
|---|---|
| Never | Link never expires |
| 5 Minutes | 5 minutes from creation |
| 1 Hour | 60 minutes from creation |
| 24 Hours | 1 day from creation |
| 7 Days | One week from creation |

The expiry timestamp (`e`) is appended to the URL. The app checks it on load and refuses to decrypt if the time has passed. Note that expiry is enforced client-side only — it is not a server-side revocation.

---

## File Encryption

Drop any file onto the upload zone in the Encrypt tab (or click to browse). The file is read as binary, Base64-encoded, and included in the encrypted payload. On decryption, the file is automatically downloaded to the recipient's device.

**Practical limits:** Very large files produce very long URLs. Most browsers handle URLs up to ~8,000 characters. Files beyond ~5 KB will start to approach this limit. For larger files, use a dedicated encrypted file-sharing service.

---

## Sharing Options

After encryption, four sharing methods are available:

| Button | Action |
|---|---|
| **WhatsApp** | Opens WhatsApp Web with the link pre-filled |
| **Copy Link** | Copies the URL to clipboard |
| **SMS** | Opens the device's SMS app with the link pre-filled |
| **QR Code** | Generates a QR code the recipient can scan |

The link is also copied to clipboard automatically when encryption completes.

---

## Settings & Customisation

Open the **Settings** tab to configure the following:

| Setting | Default | Description |
|---|---|---|
| Sound Effects | Off | Audio feedback via Web Audio API |
| Auto-Lock | On | Lock session after 5 minutes of inactivity |
| Cursor Glow | On | Ambient glow effect that follows the cursor |
| Particles | On | Animated particle background |
| Theme | Cyber | Choose between Cyber, Midnight, and Light |

Settings are persisted in `localStorage` and restored on next launch.

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + 1` | Switch to Encrypt tab |
| `Ctrl + 2` | Switch to Decrypt tab |
| `Ctrl + 3` | Switch to Settings tab |
| `Ctrl + Enter` | Encrypt or Decrypt (whichever tab is active) |
| `Ctrl + Shift + C` | Clear all fields on the active tab |
| `Ctrl + Shift + D` | Copy decrypted text to clipboard |
| `Ctrl + Shift + R` | Hard reset (clears all data and reloads) |
| `Ctrl + /` | Focus the message input |

---

## URL Parameter Reference

```
https://example.com/messenger.html?k=<key>&i=<iv>&c=<ciphertext>[&s=<salt>][&e=<expiry>]
```

| Parameter | Present when | Content | Encoding |
|---|---|---|---|
| `k` | Standard mode | 256-bit AES key | Base64URL |
| `i` | Always | 96-bit IV / nonce | Base64URL |
| `c` | Always | AES-GCM ciphertext + 128-bit auth tag | Base64URL |
| `s` | Passphrase mode | 16-byte PBKDF2 salt | Base64URL |
| `e` | Self-destruct enabled | Unix timestamp (seconds) | Integer string |

In passphrase mode, `k` is absent — the key is derived from the passphrase and salt at decryption time.

---

## Cryptographic Details

| Property | Value |
|---|---|
| Algorithm | AES-GCM |
| Key length | 256 bits |
| IV length | 96 bits (12 bytes), randomly generated per message |
| Auth tag length | 128 bits (GCM default) |
| Key derivation (passphrase mode) | PBKDF2-SHA-256, 100,000 iterations, 16-byte random salt |
| Key generation (standard mode) | `crypto.subtle.generateKey` |
| Key export format | Raw → Base64URL |
| PIN storage | SHA-256 hash in `localStorage` |
| Randomness source | `crypto.getRandomValues` for all IVs, salts, and keys |

All cryptographic operations use the browser's native **Web Crypto API** (`window.crypto.subtle`). No third-party cryptography libraries are used.

---

## Browser Compatibility

Requires the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API):

| Browser | Minimum version |
|---|---|
| Chrome / Edge | 37+ |
| Firefox | 34+ |
| Safari | 7+ |
| Opera | 24+ |

The app requires a **secure context** (`https://` or `localhost`) for clipboard write access. On `file://`, encryption works fully but auto-copy to clipboard may be blocked by the browser.

---

## Limitations

- **URL length:** Very large messages or files produce long URLs. Most browsers support URLs up to ~8,000 characters. Encrypt only short messages or small files (under ~5 KB) for reliable cross-browser compatibility.
- **No persistence:** Encrypted messages exist only as URLs. If the link is lost, the message is unrecoverable.
- **Client-side expiry only:** Self-destruct links cannot be revoked server-side — a copy of the URL taken before expiry remains valid.
- **Browser history:** In standard mode, the decryption key appears in the URL, which may be stored in browser history on the recipient's device.
- **No multi-recipient support:** Each link uses a single key. There is no concept of multiple recipients or access control.

---

## File Structure

```
.
├── aes-gcm-secure-messenger.html   # Complete single-file application
└── README.md                       # This file
```

No build step. No package manager. No server. Open the HTML file and it works.

---

## License

Provided as-is for educational and personal use. No warranty is given. Use responsibly — encrypt only content you have the right to share, and share only with intended recipients.
