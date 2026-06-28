🔐 AES-GCM Secure Messenger 
A client-side encryption tool that lets you encrypt messages with AES-256-GCM and share them via URL. No server. No data leaves your browser.
________________________________________
Quick Start
1.	Open aes-gcm-secure-messenger.html in any modern browser (Chrome, Firefox, Edge, Safari).
2.	Set a 4–8 digit PIN on first launch.
3.	Type a message and click Encrypt & Generate Link.
4.	Copy and share the link. The recipient opens it and clicks Decrypt.
________________________________________
How It Works
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Your Browser  │────▶│  AES-256-GCM    │────▶│  Shareable URL  │
│  (Plaintext)    │     │  (Key + IV + CT)│     │  (k, i, c)      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                                               ┌─────────────────┐
                                               │ Recipient's     │
                                               │ Browser decrypts│
                                               │ locally         │
                                               └─────────────────┘
•	Key: A fresh 256-bit AES key is generated for every message.
•	IV: A random 96-bit nonce ensures uniqueness.
•	Ciphertext: The encrypted message, authenticated with GCM.
•	URL: All three values are Base64URL-encoded into query parameters (k, i, c).
________________________________________
Features
Feature	Description
🔑 256-bit AES-GCM	Industry-standard authenticated encryption
🌐 Zero-server	Everything runs in your browser; no backend
🔗 Shareable links	Recipients decrypt by opening the URL
📋 Auto-copy	Encrypted links are copied to clipboard automatically
🔢 PIN protection	4–8 digit PIN guards local access (SHA-256 hashed)
🔄 Auto-decrypt	Opening a link directly triggers decryption
________________________________________
Security Model
What this tool provides
•	Confidentiality in transit: The message is encrypted before the link is shared.
•	Integrity: AES-GCM authentication detects tampering.
•	No server exposure: Plaintext never touches a server.
What this tool does NOT provide
•	End-to-end key secrecy: The decryption key is embedded in the URL. The recipient (and anyone who intercepts the link) can decrypt.
•	Forward secrecy: Each message uses a unique key, but once shared, the link persists.
•	PIN = message encryption: The PIN only locks the tool UI locally.
Best Practices
•	Share links only over trusted channels (Signal, encrypted email, in-person).
•	Treat shared links like plaintext — anyone with the URL can read the message.
•	For highly sensitive data, share the link through a separate channel from the context that reveals what it is.
________________________________________
File Structure
.
├── aes-gcm-secure-messenger.html   # The complete app (single file)
└── README.md                        # This file
No build step. No dependencies. Just open the HTML file.
________________________________________
PIN & Local Storage
Action	Behavior
Set PIN	First launch only. 4–8 digits.
Verify PIN	Required on every return visit.
Change PIN	Available in Settings. Re-hashes with SHA-256.
Reset PIN	Clears all localStorage data and reloads the app.
Forgot PIN	Press Ctrl + Shift + R to hard-reset (irreversible).
⚠️ The PIN hash is stored in localStorage. It cannot be recovered if forgotten.
________________________________________
URL Parameter Format
https://example.com/aes-gcm-secure-messenger.html?k=<key>&i=<iv>&c=<ciphertext>
Parameter	Content	Encoding
k	256-bit AES key	Base64URL
i	96-bit IV/nonce	Base64URL
c	AES-GCM ciphertext + auth tag	Base64URL
________________________________________
Browser Compatibility
Requires a browser with the Web Crypto API:
•	Chrome 37+
•	Firefox 34+
•	Safari 7+
•	Edge 12+
________________________________________
Cryptographic Details
•	Algorithm: AES-GCM
•	Key length: 256 bits
•	IV length: 96 bits (12 bytes), randomly generated per message
•	Key generation: crypto.subtle.generateKey
•	Export format: Raw → Base64URL
•	PIN hash: SHA-256 of the digit string
________________________________________
Limitations
•	URL length: Very large messages may produce URLs that exceed browser limits (~2,000–8,000 chars depending on browser). For large files, consider splitting or using a dedicated file-sharing tool.
•	No persistence: Encrypted messages exist only in the URL. If the link is lost, the message is unrecoverable.
•	Clipboard: Auto-copy requires a secure context (https:// or localhost). On file://, manual copy may be needed.
________________________________________
License
This tool is provided as-is for secure messaging education and personal use. No warranty is provided.
