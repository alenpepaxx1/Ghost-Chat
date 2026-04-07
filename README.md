# 👻 Ghost Chat — Secure Ephemeral Messaging

<div align="center">

![Ghost Chat Banner](<https://img.shields.io/badge/Ghost_Chat-v2.0-7c3aed?style=for-the-badge&logo=ghost&logoColor=white>)
![License](<https://img.shields.io/badge/License-Proprietary-red?style=for-the-badge>)
![Encryption](<https://img.shields.io/badge/Encryption-AES--256--GCM-10b981?style=for-the-badge&logo=letsencrypt&logoColor=white>)
![Firebase](<https://img.shields.io/badge/Backend-Firebase-f59e0b?style=for-the-badge&logo=firebase&logoColor=white>)
![Status](<https://img.shields.io/badge/Status-Live-10b981?style=for-the-badge>)

**Real-time end-to-end encrypted messaging where messages vanish after being read.**

No accounts. No history. No traces. Just pure privacy.

[🚀 Live Demo](<https://yourusername.github.io/ghost-chat>) · [🐛 Report Bug](<https://github.com/yourusername/ghost-chat/issues>) · [💡 Request Feature](<https://github.com/yourusername/ghost-chat/issues>)

</div>

---

## 📋 Table of Contents

- [About](#-about)
- [Features](#-features)
- [How It Works](#-how-it-works)
- [Security Architecture](#-security-architecture)
- [Getting Started](#-getting-started)
- [Usage Guide](#-usage-guide)
- [Technical Stack](#-technical-stack)
- [Project Structure](#-project-structure)
- [Security Considerations](#-security-considerations)
- [FAQ](#-faq)
- [License](#-license)
- [Author](#-author)

---

## 🔍 About

Ghost Chat is a **real-time, peer-to-peer encrypted messaging platform** built for privacy-first communication. Two people can connect to the same room using a shared secret code, and all messages are encrypted locally in the browser before being transmitted. The server (Firebase) **never sees plaintext** — it only stores encrypted ciphertext that is automatically deleted after being read.

Unlike traditional chat apps, Ghost Chat:
- Requires **no sign-up or account**
- Stores **nothing permanently** — messages auto-delete
- Encrypts **everything client-side** — zero-knowledge architecture
- Runs entirely as a **static website** — hostable on GitHub Pages for free

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔒 **End-to-End Encryption** | AES-256-GCM with PBKDF2 key derivation (150,000 iterations) |
| 👁️ **View Once Messages** | Incoming messages are blurred — tap to reveal, then they vanish |
| 👻 **Auto-Vanish** | Messages disappear with a ghost animation after being read |
| ⚡ **Real-Time** | Instant message delivery via Firebase Realtime Database |
| 👥 **Presence Detection** | See how many people are in the room and when they join/leave |
| ⌨️ **Typing Indicators** | Real-time typing detection for connected peers |
| 🔑 **Room Code Generator** | Generate cryptographically strong room codes (format: `XXXX-XXXX-XXXX`) |
| 🔗 **Easy Sharing** | Copy room code with one click to share with your partner |
| ⏱️ **Vanish Toggle** | Switch between vanishing and persistent message modes |
| 🧹 **Auto-Cleanup** | Firebase automatically purges messages older than 30 seconds |
| 📱 **Fully Responsive** | Works perfectly on desktop, tablet, and mobile |
| 🌐 **Static Hosting** | No backend server needed — runs on GitHub Pages |
| 🚫 **Zero Knowledge** | Firebase stores only encrypted data — cannot read any messages |

---

## ⚙️ How It Works

### Communication Flow

┌──────────┐                  ┌──────────────┐                  ┌──────────┐
│ Person A │                  │   Firebase   │                  │ Person B │
│ (Browser)│                  │  (Relay Only)│                  │ (Browser)│
└────┬─────┘                  └──────┬───────┘                  └────┬─────┘
│                               │                               │
│  1. Generate AES-256 key      │                               │
│     from shared room code     │      Generate same key from   │
│     (PBKDF2, 150k iter)       │      same room code           │
│                               │                               │
│  2. Encrypt message           │                               │
│     (AES-256-GCM + IV + AAD)  │                               │
│                               │                               │
│  3. Send ciphertext ─────────►│                               │
│                               │  4. Push ciphertext ─────────►│
│                               │                               │
│                               │      5. Decrypt with same key │
│                               │         Read plaintext        │
│                               │                               │
│                               │◄──── 6. Mark as read ─────────│
│                               │                               │
│                        7. Auto-delete                         │
│                           ciphertext                          │
│                               │                               │
│  8. Vanish animation          │         Vanish animation      │
│     on both sides             │         on both sides         │

### Encryption Pipeline

Room Code: "WXYZ-1234-ABCD"
│
▼
┌──────────────────────────────┐
│         PBKDF2               │
│  Salt: "ghost-chat-alen-     │
│         pepa-2026"           │
│  Iterations: 150,000         │
│  Hash: SHA-256               │
└──────────────┬───────────────┘
│
▼
┌──────────────────────────────┐
│     AES-256-GCM Key          │
│  (256-bit symmetric key)     │
└──────────────┬───────────────┘
│
Message: "Hello!"
│
▼
┌──────────────────────────────┐
│     AES-256-GCM Encrypt      │
│  + Random 96-bit IV          │
│  + Authenticated Data (AAD)  │
│  + 128-bit Auth Tag          │
└──────────────┬───────────────┘
│
▼
Ciphertext: "x8fK2p9..."
(sent to Firebase)

### Room Hashing
Room Code: "WXYZ-1234-ABCD"
│
▼
┌──────────────────────────────┐
│       SHA-256 Hash           │
│  (first 24 hex characters)   │
└──────────────┬───────────────┘
│
▼
Firebase Path: /rooms/a3f8b2c1d4e5f6a7b8c9d0e1/
(Room code is NEVER stored on the server)

---

## 🔐 Security Architecture
┌─────────────────────────────────────────────────────┐
│                    CLIENT SIDE                       │
│                                                     │
│  ┌─────────────┐    ┌──────────────────────────┐   │
│  │  Room Code  │───►│  PBKDF2 Key Derivation   │   │
│  │  (secret)   │    │  150,000 iterations       │   │
│  └─────────────┘    └────────────┬─────────────┘   │
│                                  │                   │
│                          AES-256 Key                 │
│                                  │                   │
│  ┌─────────────┐    ┌───────────┴──────────────┐   │
│  │  Plaintext  │───►│  AES-256-GCM Encryption  │   │
│  │  Message    │    │  + Random IV + AAD        │   │
│  └─────────────┘    └────────────┬─────────────┘   │
│                                  │                   │
│                           Ciphertext                 │
│                                  │                   │
└──────────────────────────────────┼───────────────────┘
│
─────────┼─────────
│
┌──────────────────────────────────┼───────────────────┐
│                    FIREBASE      │                    │
│                   (Zero Knowledge)                    │
│                                  │                    │
│  ┌───────────────────────────────┴──────────────┐    │
│  │  Stores ONLY:                                │    │
│  │  - Encrypted ciphertext (unreadable)         │    │
│  │  - Hashed room ID (irreversible)             │    │
│  │  - Timestamp (for auto-cleanup)              │    │
│  │  - User display name                         │    │
│  │                                              │    │
│  │  CANNOT access:                              │    │
│  │  - Message content                           │    │
│  │  - Room code / encryption key                │    │
│  │  - Any plaintext data                        │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  Auto-deletes all data after 15-30 seconds           │
└──────────────────────────────────────────────────────┘

---

## 🚀 Getting Started

### Prerequisites

- A [GitHub](<https://github.com>) account (free)
- A [Firebase](<https://firebase.google.com>) account (free — Google account)
- A modern web browser (Chrome, Firefox, Safari, Edge)

---

### Step 1: Create Firebase Project

1. Go to **[Firebase Console](<https://console.firebase.google.com>)**
2. Click **"Add Project"**
3. Name it `ghost-chat` (or anything you prefer)
4. **Disable** Google Analytics (not needed) → Click **"Create Project"**
5. Wait for creation → Click **"Continue"**

---

### Step 2: Create Realtime Database

1. In the left sidebar, click **Build → Realtime Database**
2. Click **"Create Database"**
3. Choose a location closest to you (e.g., `United States (us-central1)`)
4. Select **"Start in test mode"** → Click **"Enable"**
5. Go to the **"Rules"** tab and replace with:

```json
{
  "rules": {
    "rooms": {
      "$roomId": {
        ".read": true,
        ".write": true,
        "messages": {
          "$msgId": {
            ".write": true,
            ".read": true
          }
        },
        "presence": {
          "$uid": {
            ".write": true,
            ".read": true
          }
        },
        "typing": {
          "$uid": {
            ".write": true,
            ".read": true
          }
        }
      }
    }
  }
}


Click "Publish"
Step 3: Register Web App & Get Config
Go to Project Settings (⚙️ gear icon, top left)
Scroll down to "Your apps" section
Click the Web icon (</>)
Enter app nickname: ghost-chat
Don't check Firebase Hosting → Click "Register app"
You'll see a config object like this:

const firebaseConfig = {
    apiKey: "AIzaSyBxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    authDomain: "ghost-chat-xxxxx.firebaseapp.com",
    databaseURL: "<https://ghost-chat-xxxxx-default-rtdb.firebaseio.com>",
    projectId: "ghost-chat-xxxxx",
    storageBucket: "ghost-chat-xxxxx.appspot.com",
    messagingSenderId: "123456789",
    appId: "1:123456789:web:abcdef123456"
};


Copy these values
Step 4: Add Firebase Config to Code
Open index.html and find this section (around line 480):

const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
    projectId: "YOUR_PROJECT",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
};

Replace each YOUR_... value with your actual Firebase config values from Step 3.
Step 5: Deploy to GitHub Pages
Create a new repository on GitHub named ghost-chat
Upload your files:


Go to Settings → Pages
Under "Source", select Branch: main → Folder: / (root)
Click "Save"
Wait 1-2 minutes — your site will be live at:

📖 Usage Guide
Starting a Conversation

Person A                              Person B
────────                              ────────
1. Open the website
2. Enter your name
3. Click "Generate" for room code
4. Click "Enter Secure Room"
5. Click 🔗 to copy room code
6. Send room code to Person B
   (via any channel)
                                      7. Open the same website
                                      8. Enter their name
                                      9. Paste the room code
                                      10. Click "Enter Secure Room"

✅ Both are now connected with end-to-end encryption!

Message Lifecycle
Message sent ──► Encrypted (AES-256) ──► Stored on Firebase (ciphertext)
                                                    │
                                             Peer receives
                                                    │
                                              Peer decrypts
                                                    │
                                         Blurred overlay shown
                                                    │
                                          Peer taps to reveal
                                                    │
                                     ┌──── 4 second timer ────┐
                                     │                         │
                                     ▼                         ▼
                              Ghost animation          Deleted from Firebase
                                     │
                                     ▼
                               Message gone forever

🔒 Security Considerations

✅ What IS Secure
Messages are encrypted client-side before leaving the browser
Firebase never sees plaintext message content
Room codes are hashed with SHA-256 before use as database paths
Each message uses a unique random IV (Initialization Vector)
Authenticated encryption (GCM mode) prevents message tampering
Messages auto-delete from Firebase within seconds
PBKDF2 with 150,000 iterations makes brute-force impractical

⚠️ What to Be Aware Of
Room code security depends on how you share it — use a secure channel
Display names are sent unencrypted (only names, not message content)
Firebase rules are open for demo purposes — add authentication for production
Browser extensions could potentially read the DOM after decryption
This is a portfolio/demo project, not professionally audited for production use

🛡️ Recommended for Production
Add Firebase Authentication
Implement rate limiting rules
Add message size limits on Firebase rules
Use Firebase Security Rules with auth validation
Consider adding forward secrecy (key rotation per session)
Add Content Security Policy headers

👤 Author
Alen Pepa
