# SecureVault

A 100% local, zero-knowledge password manager that runs entirely from a single HTML file. No install, no server, no account, no telemetry — open it in a browser and it works.

Built by [Exploit Munchies](https://exploitmunchies.com) as both a real tool and a teaching artifact: the whole encrypt → decrypt → import → export pipeline, in code you can actually read.

## Why this exists

Before trusting any password manager — self-hosted or otherwise — with real credentials, I wanted to know exactly what happens between "user types master password" and "encrypted blob on disk." So I built the full pipeline myself: key derivation, authenticated encryption, vault serialization, multi-format CSV migration, and a generator that doesn't have modulo bias. This repo is the result.

## Try it

Download `securevault.html` and open it in any modern browser (Chrome, Edge, Firefox, Brave, Safari). That's the whole install process. It also runs fine straight off a USB stick with no network connection.

## Security architecture

| Layer | Implementation |
|---|---|
| Encryption | AES-256-GCM (authenticated — tampering or a wrong key is detected, not silently accepted) |
| Key derivation | PBKDF2-SHA256 @ **310,000 iterations** (OWASP-recommended floor) |
| Crypto provider | Native Web Crypto API (`crypto.subtle`) — zero third-party crypto libraries |
| Salt & IV | Fresh cryptographically random salt per vault, fresh IV per encryption operation |
| Model | Zero-knowledge — the master password is never stored, transmitted, or recoverable |
| Persistence | None by default. The vault lives in memory for the tab's session; you explicitly export an encrypted backup and re-import it next time |

**What "zero-knowledge" means here:** the master password only ever exists in memory. It's stretched through PBKDF2 into an AES key, used to encrypt or decrypt, and never written anywhere in cleartext. Lose the master password and the backup is unrecoverable by design — there's no reset flow, because a reset flow is a backdoor.

**On persistence:** this standalone version deliberately does not use `localStorage` or any browser storage API. Your vault exists only in the tab's memory. Closing the tab without exporting a backup means starting over. This is a tradeoff in favor of "nothing touches disk unencrypted, ever" over convenience — export a backup after any change you care about.

## Features

- **Local-only vault** — AES-256-GCM encrypted, nothing ever touches a network
- **Universal CSV import** — auto-detects source format by fingerprinting column headers: Dashlane, Bitwarden, LastPass, Chrome/Edge, 1Password, KeePass, or generic CSV with best-guess column mapping
- **Import preview** — per-row selection and duplicate detection before anything is written to the vault
- **Ranked fuzzy search** across title, username, URL, and notes, with match highlighting
- **Password generator** using `crypto.getRandomValues()` with rejection sampling (no modulo bias), configurable length and character classes, optional ambiguous-character exclusion
- **Export** as Bitwarden-format CSV (clean migration path to Bitwarden/Vaultwarden), generic CSV, or an AES-256-GCM encrypted JSON backup
- **Logins, cards, and secure notes** as distinct entry types

## Migrating from another password manager

1. Export a CSV from your current manager
2. Open SecureVault → **Import** → select the CSV
3. Confirm the detected format and review the row preview
4. Import, then immediately **Export → encrypted backup** to save your work

## Migrating *to* Vaultwarden / Bitwarden

Use **Export → Bitwarden-format CSV**, then import that file directly into Bitwarden or a self-hosted Vaultwarden instance.

## What this is not

This is an educational / personal-use project, not an audited production security product. It hasn't been through a third-party security audit. If you're choosing infrastructure for a team or storing your only copy of critical credentials, that's worth knowing going in.

## Want the Chrome extension version?

There's a companion Manifest V3 Chrome extension with browser-integrated autofill, save-on-login capture, and persistent encrypted storage via `chrome.storage.local` — plus a full written walkthrough covering every design decision in this codebase (PBKDF2 vs. Argon2, why AES-GCM, the CSV fingerprinting engine, the MV3 service-worker session model). That's available as a paid package on Gumroad: **[link coming soon]**.

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, put it in your portfolio.

---

Built by [Richard Willingham](https://github.com/exploitmunchiescoder) · [exploitmunchies.com](https://exploitmunchies.com)
