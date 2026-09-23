# Backup & Recovery Suite (BRS)

A web-based backup and recovery management platform with **real cryptographic hashing, real gzip compression, and real differential backup logic** — running entirely in the browser.

**[View live demo →](https://samuel1tadele-hash.github.io/backup-recovery-suite/)**

---

## What it does

BRS simulates a production-grade backup management system for a set of data sources:

- **Overview** — system health, storage breakdown, unbacked changes, recent activity
- **Sources** — three mock data systems (Customers, Orders, Inventory) with the ability to mutate live data
- **Backups** — create and manage full and differential backups with real checksums
- **Restore** — visual recovery timeline; restore any source to any point in history
- **Verify** — re-hash each backup and detect corruption via SHA-256 comparison
- **Schedule** — automated backup jobs with configurable intervals and retention
- **Activity** — full audit log with severity filtering
- **Role-based access** — admin, operator, and viewer roles with distinct permissions

---

## Why these technology choices?

Most "backup tool" side projects just copy JSON into `localStorage` and call it a day. That doesn't demonstrate any real engineering.

This project uses **browser-native cryptographic and compression APIs** so the backup logic is genuinely functional:

| Feature | Real implementation | Why it matters |
|---|---|---|
| **Integrity verification** | `crypto.subtle.digest('SHA-256', ...)` via Web Crypto API | The exact same hashing algorithm used by git, Docker, and TLS |
| **Compression** | `CompressionStream('gzip')` — real DEFLATE, not a fake size number | Actual byte-level compression with meaningful ratios |
| **Decompression** | `DecompressionStream('gzip')` | Proves the restore path actually reads the archived bytes back |
| **Differential backups** | Row-level diff against the last full backup (added / removed / modified sets) | Demonstrates understanding of backup strategy, not just file copying |
| **Persistence** | `localStorage` with quota-aware error handling | Real storage limits, real failure modes |

The whole app runs client-side. No backend, no server costs, no deployment complexity — just a single HTML file.

---

## Tech stack

| Layer | Technology |
|---|---|
| Markup + Styling | HTML5, Tailwind CSS |
| Logic | Vanilla JavaScript (ES2020+) |
| Crypto | Web Crypto API (SHA-256) |
| Compression | CompressionStream / DecompressionStream (gzip) |
| Auth | SHA-256 hashed passwords via Web Crypto |
| Persistence | localStorage (JSON state) |

No build step. No npm. No framework. No dependencies beyond Tailwind and Google Fonts from a CDN.

---

## Backup architecture

### Full vs differential backups

BRS implements a real two-tier backup strategy:

**Full backup** — captures the entire source dataset as a JSON snapshot, compresses it with gzip, and stores the SHA-256 checksum of the *uncompressed* content.

**Differential backup** — computes the delta between the current source state and the last full backup for that source:
