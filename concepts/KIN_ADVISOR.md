# KinAdvisor — MVP Specification (Markdown)

## 1. Overview

KinAdvisor is a personal relationship-support tool designed to help users stay meaningfully connected with family members. It records conversations (voice), produces structured summaries, stores them per‑contact, and proactively reminds users to reach out when too much time has passed. It also suggests conversation topics based on recent interaction summaries.

The product focuses on lightweight personal relationship management, private note‑taking, and automated conversation insights.

---

## 2. Product Principles

- **Assistive, not intrusive:** Provide helpful reminders and suggestions without overwhelming the user.
- **Privacy-first:** All summaries and recordings processed on-device where possible. Cloud syncing optional and end‑to‑end encrypted.
- **Clarity:** Simple visual structure and minimalistic layout similar to modern productivity apps.
- **Actionable insights:** Provide practical nudges—contact reminders and topic suggestions.

---

## 3. Core Objectives (MVP)

- Capture voice notes or short conversation recordings.
- Generate concise AI-generated summaries stored per contact.
- Maintain contact list with interaction timelines.
- Surface reminders when the user hasn’t interacted with someone for a configurable interval.
- Suggest future conversation topics derived from previous summaries.

---

## 4. User Flows

### 4.1 Onboarding

1. User grants microphone permission.
2. User optionally imports contacts or creates private contact entries manually.
3. Onboarding explains how summaries and reminders work.

### 4.2 Recording a Conversation

1. User selects a contact.
2. User records a voice note or logs a conversation.
3. Audio is transcribed.
4. Summary is generated locally or through encrypted processing.
5. Summary stored in the contact’s history.

### 4.3 Reminder Flow

1. App evaluates last-interaction timestamps.
2. When threshold is exceeded, user receives a notification.
3. Notification includes:

   - A reminder to reach out.
   - Suggested discussion topic based on recent summaries.

### 4.4 Viewing a Contact Profile

- Recent summaries
- Interaction timeline
- Suggestions for next call
- Personal notes

---

## 5. Functional Requirements

### 5.1 Mandatory

- Voice recording
- Automatic transcription
- Summary generation
- Local contact list
- Interaction timeline
- Notification engine with inactivity thresholds
- Topic-suggestion engine based on summary embeddings

### 5.2 Optional (Stretch)

- End-to-end encrypted cloud backup
- Multi-device sync
- Rich media note attachments
- Mood detection from voice
- Calendar integration

---

## 6. Non-Functional Requirements

- **Privacy:** All processing local by default; encrypted backups optional.
- **Accuracy:** Transcription accuracy must support multiple languages.
- **Reliability:** Summaries deterministic and traceable.
- **Performance:** Summaries generated within seconds.

---

## 7. System Architecture

### 7.1 Client Components

- Recorder module
- On-device transcription model
- Summarization model (lightweight or hybrid local/cloud)
- Contact manager with local encrypted database
- Reminder scheduler
- Topic-suggestion model using vector similarity across summaries

### 7.2 Backend Components (Optional for MVP)

- Encrypted sync service (if enabled)
- User profile for encrypted backup keys

### 7.3 Data Model

**Contact**

- Name
- Last interaction timestamp
- Interaction frequency target
- Metadata (local only)

**Conversation Entry**

- Date/time
- Transcript text
- Summary
- Suggested topics
- Audio reference (encrypted local file)

---

## 8. Security & Privacy Model

- All contact data, transcripts, audio, and summaries stored in encrypted local storage.
- Cloud sync (if enabled) uses end-to-end encryption with user-held encryption keys.
- Models run locally; if cloud inference used, transcripts transmitted encrypted in transit and deleted immediately after processing.
- No analytics on content; only aggregate technical metrics.

---

## 9. UX / UI Approach

- Clean, monochrome or desaturated palette.
- Contact list as primary navigation.
- Compact timeline with summaries per interaction.
- Clear call-to-action for recording.
- Notification center showing pending reminders.

---

## 10. Technical Stack

**Frontend**

- Expo (React Native)
- TypeScript
- Local embedded database (SQLite with encryption)

**AI Processing**

- On-device transcription via small ASR model (e.g., RN Whisper or similar optimized model)
- Summarization via local LLM or encrypted remote LLM
- Embedding model for topic generation

**Backend (Optional)**

- Encrypted storage buckets
- Minimal backup API

---

## 11. Risks

- On-device models may increase app size.
- Long recordings challenge local processing.
- Cloud processing introduces privacy concerns.
- Reminder fatigue possible without careful UX tuning.

---

## 12. Success Criteria

- Users can store and summarize conversations reliably.
- Reminder engine operates accurately.
- Summaries remain helpful and consistent.
- Topic suggestions relevant to past conversations.

---

## 13. Monetization

### Revenue Models

1. **Freemium / Subscription Model**

   - Free tier: limited contacts, summary generation, and reminders.
   - Premium tier:

     - Unlimited contacts & history
     - Cloud backup (encrypted)
     - Advanced AI summarization (longer conversation, multi-language support)
     - Topic suggestions powered by larger AI models

2. **In-App Purchases (Non-Intrusive Add-ons)**

   - Add-on packs for specialized conversation topics (e.g., mental health, family history prompts).
   - Customizable reminder patterns or personalized AI coaching tips.

3. **Enterprise / Family Plan**

   - Multi-user family subscriptions with shared insights (still encrypted per user) or family office solutions.

4. **Optional Partnerships / Integration**

   - Integration with smart devices (calendar, voice assistants) for premium tier.
   - Partnerships with coaching or relationship content providers — only if privacy-preserving and opt-in.

**Unsuitable:** Ads or third-party content tracking.

---
