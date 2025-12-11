# QUIETLINK - Minimalistic Private Messaging App — MVP Specification

## 1. Overview

Minimal, privacy-first one-to-one messaging mobile app with a black/white visual language (inspired by Linear). Goals for the MVP: secure end‑to‑end encryption, zero‑knowledge relay infrastructure (no message content stored), simple onboarding, Expo-based cross‑platform frontend.

---

## 2. High-level Principles

* **Privacy by default**: users’ messages and identifying metadata are not retained by servers.
* **Strong cryptography**: use proven, audited protocols (X3DH / (P)QXDH for initial key agreement; Double Ratchet for message confidentiality and forward secrecy).
* **Minimal UI**: monochrome, distraction‑free UX focused on chats and composing messages.
* **Stateless relay**: backend only routes encrypted payloads and holds minimal routing information required for delivery.

---

## 3. Technical Targets (MVP)

* Expo-managed frontend (React Native / Expo) with TypeScript.
* One‑to‑one encrypted messaging (text only) using Signal‑class protocols (X3DH + Double Ratchet).
* Device‑local identity (keypair) generation and local encrypted message store (device‑encrypted SQLite or secure storage).
* Stateless relay + PreKey publish service for asynchronous delivery.
* Push notifications carrying no message content (opaque delivery tokens).

---

## 4. Encryption & Security Model (Detailed)

### 4.1 Protocol Stack (recommended)

* **Initial key agreement**: X3DH / PQXDH-style prekey exchange to establish initial shared secret for asynchronous delivery. This allows one party to initiate a conversation while the other is offline.
* **Message ratcheting**: Double Ratchet algorithm (symmetric-key ratchet + DH ratchet) for per-message keys, forward secrecy, and post‑compromise recovery. Implement Double Ratchet exactly as in the specification to preserve known security properties.
* **Optional PQ (post‑quantum)**: PQXDH extensions exist in literature; include only after careful review and when supported by chosen libraries.

### 4.2 Cryptographic Primitives

* **Curve / KEX**: Prefer X25519 (Curve25519) for Diffie‑Hellman operations (compact, widely supported). Use Ed25519 for signatures if needed.
* **AEAD**: Use a modern authenticated cipher (XChaCha20-Poly1305 or AES-GCM depending on platform support). XChaCha20 is preferable for simpler cross-platform usage and nonce misuse resistance.
* **KDF / HKDF**: Use HKDF-SHA256 or equivalent for key derivation inside ratchets.

### 4.3 Key Management

* **Device identity**: Each installation generates a long‑term identity keypair (stored encrypted on-device). Public identity key is shared via user identifier or QR code.
* **PreKeys**: Clients publish a bundle of signed prekeys (one signed prekey + multiple one‑time prekeys) to the PreKey service so asynchronous senders can perform X3DH when the recipient is offline. The PreKey service stores only public keys and minimal routing tokens — no private material.
* **Session keys**: Derived per conversation and stored only in memory and encrypted local storage; rotated by the Double Ratchet.
* **Key backup**: Provide explicit, user‑initiated encrypted export (passphrase‑protected) or client‑side encrypted cloud backup option (end‑to‑end encrypted with user passphrase). Avoid automatic server‑side key escrow.

### 4.4 Transport & Network Security

* **Transport layer**: TLS 1.3 for all server communications (relay, PreKey upload, discovery). Enforce strict TLS configuration and certificate pinning in the mobile client where feasible.
* **Message delivery**: Use WebSockets for real‑time delivery; fall back to long‑polling if necessary. Relay only forwards opaque, fully encrypted payloads.
* **Push notifications**: Use APNs / FCM to wake clients; notifications must contain no message content, only an opaque routing token/notification that prompts the client to fetch messages.

### 4.5 Metadata Minimization & Zero‑Knowledge

* **No message storage**: Relay nodes discard message payloads immediately after successful delivery acknowledgement; if recipient is offline, server can queue encrypted bundles in volatile memory or ephemeral encrypted queue with strict TTL and automatic purge.
* **Minimal identifiers**: Use cryptographic identifiers derived from public keys (e.g., base58 of public key hash) instead of phone numbers or email addresses.
* **No analytics with PII**: Avoid event telemetry that contains user identifiers or conversation metadata. If telemetry is needed for diagnostics, aggregate and anonymize on-device before upload.

### 4.6 Threat Model Summary

* **Assumes**: attacker can observe network traffic and operate the relay server but cannot extract private keys from user devices.
* **Provides**: message confidentiality, forward secrecy, and cryptographic deniability similar to Signal when properly implemented with audited primitives.
* **Limitations**: No server-side abuse detection; account loss if device and backups are lost; push providers see metadata required by APNs/FCM.

---

## 5. Libraries & Implementation Options (Expo-specific notes)

* **Signal / libsignal**: Official Signal stacks are migrating toward `libsignal-client` (Rust-based with TypeScript bindings). This is the most feature‑complete and actively maintained reference implementation for Signal‑class protocols. Integration into Expo may require native modules or a custom dev/client build.
* **libsignal‑protocol‑typescript / community implementations**: TypeScript ports exist and may be used for a pure‑JS approach that runs in Expo’s managed JS runtime, but they vary in maintenance and may require careful review and testing.
* **Libsodium / low‑level primitives**: `react-native-libsodium` (or `libsodium.js`) can provide primitives (X25519, XChaCha20‑Poly1305, HKDF) and has Expo plugin support via the dev‑client. For production, prefer audited protocol libraries rather than assembling protocol primitives yourself.

**MVP Frontend libraries to consider:**

1. Prototype with a TypeScript Signal‑protocol implementation that runs in pure JS (e.g., community `libsignal-protocol-typescript`) to validate UX and flows inside Expo managed workflow.
2. For production, move to `@signalapp/libsignal-client` (or a maintained native binding) integrated via an Expo dev‑client or prebuild to expose native libs for full compatibility and performance. Expect native build work and CI changes.

---

## 6. System Architecture (Detailed)

### 6.1 Components

* **Clients (Expo mobile apps)**

  * Key manager (identity keys, prekey generation)
  * Session manager (X3DH handshake, Double Ratchet state)
  * Local encrypted DB (encrypted SQLite or SecureStore for metadata + encrypted blobs)
  * UI (monochrome chat list, single chat view, QR/ID scanner)

* **PreKey Service** (public)

  * Stores only published user public prekeys and a small routing token (no PII).
  * Used by senders to perform X3DH when recipient offline.
  * Allows periodic cleanup of stale prekeys.

* **Stateless Relay / Router**

  * Accepts encrypted payloads; routes to recipient connection (WebSocket) or places into an ephemeral queue if offline.
  * Acknowledges delivery and immediately purges payload.
  * Horizontally scalable; no durable storage of message content.

* **Push Notification Gateway**

  * Sends silent push (APNs/FCM) with only an opaque identifier so the client can open a WebSocket and fetch messages.

* **Optional Admin / Management**

  * Minimal operator interface (health checks, metrics that do not include user identifiers).

### 6.2 Delivery Flow (simplified)

1. Sender obtains recipient PreKey bundle from PreKey Service.
2. Sender performs X3DH and derives initial session keys; sends encrypted message payload to Relay with recipient routing token.
3. Relay delivers to recipient’s WebSocket (if online). If offline, Relay places encrypted bundle in ephemeral in-memory queue and triggers push.
4. Recipient fetches bundle, performs Double Ratchet setup/continued ratchet, decrypts locally, and confirms delivery (optional encrypted ACK).
5. Relay deletes the bundle immediately after delivery/ACK or on TTL expiry.

---

## 7. Data Storage & Local Persistence

* **On‑device**: Encrypted SQLite for chat entries (ciphertext only), local storage of identity keys encrypted under a device key (Secure Enclave / Keychain recommended).
* **Server**: No persistent message store. PreKey Service stores *only* public keys. Relay may temporarily hold encrypted bundles in memory/ephemeral queue for asynchronous delivery.

---

## 8. UX / Onboarding

* On first run, app generates identity keypair and shows a 2–3 step minimal flow: (1) optional display alias (local only), (2) show shareable public‑key‑derived ID and QR code, (3) allow scanning another user’s QR or paste ID to start.
* In chat, messages show send/receive, timestamp, and optional ephemeral timer toggle per message (local deletion only).

---

## 9. Operational Considerations

* **CI / builds**: For production using `@signalapp/libsignal-client` or native libs, use Expo prebuild or EAS Build to include native bindings.
* **Security reviews**: Mandatory third‑party cryptographic audit before public launch.
* **Monitoring**: Operate relay nodes with privacy‑preserving metrics (aggregate counts, health checks). Avoid logs containing identifiers.

---

## 10. Risks & Mitigations

* **Key loss**: Implement user‑exportable encrypted backups; warn users of risk when no backup exists.
* **Push provider metadata**: Push services (APNs/FCM) reveal device tokens to providers; document this in privacy statement and minimize data in notifications.
* **Library maintenance**: Some JS Signal ports are community projects—plan for migration to upstream `libsignal-client` or vetted native bindings for long‑term support.

---

## 11. Implementation Roadmap (MVP -> v1)

**MVP (0–3 months)**

* Expo prototype with pure‑JS Signal protocol implementation for core flows. citeturn1search0
* Stateless relay + PreKey service (minimal API), WebSocket delivery, push integration.
* Local encrypted storage and identity key generation.
* Security review checklist and internal threat modelling.

**v1 (3–6 months)**

* Migrate to native `libsignal-client` bindings (prebuild/EAS) for production cryptography.
* Add encrypted backups, ephemeral messages, and UX polish.
* Independent cryptographic audit and penetration test.

---

## 12. References (select)

* Double Ratchet specification (Signal). citeturn0search19
* `libsignal-protocol-javascript` archival notice and migration guidance.
* `@signalapp/libsignal-client` NPM (active libsignal client package).
* `react-native-libsodium` Expo plugin notes. citeturn0search2
* Community TypeScript Signal protocol implementations / demos.

---
