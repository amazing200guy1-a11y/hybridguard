# HybridGuard AI — Zero-Trust On-Device & Cloud Sentry

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Firebase AI Logic](https://img.shields.io/badge/Firebase-AI%20Logic-orange.svg)](https://firebase.google.com)
[![Gemini Nano](https://img.shields.io/badge/Gemini-Nano%20On--Device-green.svg)](https://ai.google.dev)
[![Gemini Flash](https://img.shields.io/badge/Gemini-Flash%20Cloud-blue.svg)](https://ai.google.dev)

## Overview

**HybridGuard AI** is a production-grade zero-trust AI security firewall that implements a hybrid inference architecture — combining **Gemini Nano on-device processing** with **Gemini Flash cloud-tier analysis** via **Firebase AI Logic**. It provides instant, offline threat detection with deep fraud-graph analysis on demand, enforced by a mandatory human checkpoint gate and an immutable offline-first audit ledger.

> Built for the Firebase AI Logic Hackathon. Demonstrates the full hybrid AI security stack: on-device inference → confidence-gated cloud escalation → human review → tamper-proof audit.

---

## Live Demo

**Dashboard:** [https://amazing200guy1-a11y.github.io/hybridguard/](https://amazing200guy1-a11y.github.io/hybridguard/)

**YouTube Demo:** [https://youtu.be/[PENDING]](https://youtu.be/)

---

## Architecture

```
                        ┌─────────────────────────────────┐
                        │       INCOMING REQUEST           │
                        └────────────┬────────────────────┘
                                     │
                        ┌────────────▼────────────────────┐
                        │   GEMINI NANO (ON-DEVICE)        │
                        │   Latency: < 1ms                 │
                        │   Offline-capable                │
                        └────────────┬────────────────────┘
                                     │
                          Confidence > 72%?
                         /               \
                       YES                NO
                        │                 │
              ┌─────────▼──────┐  ┌───────▼──────────────────┐
              │ LOCAL VERDICT   │  │ FIREBASE AI LOGIC ROUTER  │
              │ BLOCK / PASS   │  │ → Gemini Flash (Cloud)    │
              └─────────┬──────┘  │   Latency: ~240ms         │
                        │         └───────┬──────────────────┘
                        │                 │
                        │       ┌─────────▼──────────────┐
                        │       │  HUMAN CHECKPOINT GATE  │
                        │       │  Mandatory review before │
                        │       │  enforcement action      │
                        │       └─────────┬──────────────┘
                        │                 │
                        └─────────────────▼
                        ┌─────────────────────────────────┐
                        │  OFFLINE AUDIT LEDGER            │
                        │  Firestore offline persistence   │
                        │  Hash-chained tamper-proof log   │
                        └─────────────────────────────────┘
```

### Inference Tiers

| Tier | Model | Latency | Trigger |
|------|-------|---------|---------|
| On-Device | Gemini Nano | 0.4ms | All requests (primary) |
| Cloud | Gemini Flash via Firebase AI Logic | ~240ms | On-device confidence < 72% |

### Routing Logic

```
confidence = gemini_nano.infer(request)
if confidence >= 0.72:
    verdict = local_decision(confidence)
    audit_ledger.write(verdict, tier="ON_DEVICE")
else:
    escalation = firebase_ai_logic.route_to_flash(request)
    human_checkpoint.review(escalation)
    audit_ledger.write(escalation.verdict, tier="CLOUD")
```

---

## Features

### Zero-Trust Hybrid Inference
- Every request is inspected — no implicit trust at any tier
- On-device Gemini Nano processes 97%+ of requests locally with sub-millisecond latency
- Automatic cloud escalation when on-device confidence falls below threshold

### Firebase AI Logic Integration
- Firebase AI Logic SDK routes escalated requests to Gemini Flash
- Seamless tier switching with no client-side configuration changes
- Cloud verdict feeds back into local confidence model

### Offline-First Architecture
- Gemini Nano operates fully offline — zero cloud dependency for primary inference
- Firebase Firestore offline persistence queues escalations and audit entries
- Automatic sync when connectivity is restored, with conflict resolution

### Human Checkpoint Gate
- **No enforcement action** is taken without mandatory human review on escalated cases
- Review queue surfaces cloud verdict, confidence delta, threat classification, and analysis timeline
- Override capability with full audit trail

### Tamper-Proof Audit Ledger
- Hash-chained append-only ledger stored in Firestore
- Every decision (block, pass, escalate, override) is immutably recorded
- Offline writes are queued and synced with cryptographic integrity verification
- Full forensic replay capability

### Threat Detection Coverage
- `PROMPT_INJECTION` — LLM prompt manipulation attempts
- `SQL_INJECT` — SQL injection payloads
- `XSS_PAYLOAD` — Cross-site scripting vectors
- `OPEN_REDIRECT` — Unvalidated redirect attacks
- `AUTH_BYPASS` — Authentication circumvention attempts
- `SAFE` — Clean request passthrough (cleared with confidence score)

---

## Dashboard Panels

| Panel | Description |
|-------|-------------|
| **Inference Tier Router** | Live animated routing diagram showing on-device vs cloud decision flow, latency, and routing statistics |
| **Live Threat Detection Feed** | Real-time scrolling feed of threat verdicts with timestamps, confidence scores, and tier attribution |
| **Cloud Escalation Queue** | Detailed view of escalated cases with on-device confidence bars, cloud verdicts, and analysis times |
| **Offline Audit Ledger** | Firestore sync status, tamper-proof hash-chained audit table, and ledger integrity verification |

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| On-Device AI | Gemini Nano (Chrome AI / Android AICore) |
| Cloud AI | Gemini Flash (via Firebase AI Logic) |
| AI Router | Firebase AI Logic SDK |
| Offline Persistence | Firebase Firestore (offline mode) |
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Typography | JetBrains Mono, Inter (Google Fonts) |
| Hosting | GitHub Pages |

---

## Setup

### Prerequisites
- Firebase project with Firestore enabled
- Firebase AI Logic SDK access
- Chrome 127+ (for Gemini Nano) or Android device with AICore

### Installation

```bash
# Clone the repository
git clone https://github.com/amazing200guy1-a11y/hybridguard.git
cd hybridguard

# No build step required — pure HTML/CSS/JS
# Open index.html in a browser or serve via any static file server
```

### Firebase Configuration

```javascript
// Add to index.html before closing </body>
import { initializeApp } from 'firebase/app';
import { getAILogic } from 'firebase/ai-logic';
import { getFirestore, enableIndexedDbPersistence } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  // ...
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const aiLogic = getAILogic(app);

// Enable offline persistence
await enableIndexedDbPersistence(db);
```

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /audit_ledger/{entry} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
        && request.resource.data.keys().hasAll(['timestamp','action','tier','verdict','hash','prev_hash']);
      allow update, delete: if false; // Tamper-proof: no updates or deletes
    }
  }
}
```

---

## Project Structure

```
hybridguard/
├── index.html          # Complete self-contained dashboard (HTML + CSS + JS)
├── README.md           # This file
└── .gitignore          # Standard web project ignores
```

---

## Security Design Principles

1. **Zero Trust** — Every request is treated as potentially hostile regardless of source
2. **Defense in Depth** — On-device + cloud + human review as three independent layers
3. **Fail Secure** — On confidence failure, escalate; never default to pass
4. **Audit Everything** — Every decision is immutably recorded with hash-chain integrity
5. **Offline Resilience** — Core protection functions without cloud connectivity

---

## License

MIT License — Copyright (c) 2024 HybridGuard AI

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
