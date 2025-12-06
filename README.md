# ☁️ CloudSEK CTF 2025 Writeups

**A collection of solutions for the CloudSEK Capture The Flag competition.**

This repository documents my approach to solving the Web Exploitation, Mobile Security, and Scripting challenges during the event.

---

## 🏆 Performance Overview

| **Metric** | **Stat** |
| :--- | :--- |
| **Final Rank** | **13th** / ~1600 participants (Top 1%) |
| **Status** | Active Participant (Solo Entry) |
| **Challenges Attempted** | 4 |
| **Challenges Solved** | 4 (**100% Accuracy**) |
| **Categories** | Web, Mobile, OSINT, Automation |

### 📝 Strategic Reflection
Securing a **13th place finish** in such a crowded field required a strategy of "quality over quantity." I focused exclusively on high-value targets involving complex exploit chains rather than guessing on lower-tier challenges.
- **Key Pivot:** Moving from standard payloads to protocol manipulation (XXE) in *Bad Feedback*.
- **Best Solve:** *Ticket*, which required connecting Mobile OSINT (BeVigil) to Backend Web Exploitation (JWT Forgery).

---

## 🚩 Challenge Solutions

Click on a challenge name to view the detailed write-up and exploit scripts.

| Challenge Name | Category | Difficulty | Key Technique / Vulnerability |
| :--- | :--- | :--- | :--- |
| **[Ticket 100](./Ticket)** | 🌐 Hybrid (Web/Mobile) | 🔥 Hard | **Mobile OSINT** (BeVigil) → **Hardcoded Secrets** → **JWT Signature Forgery** |
| **[Triangle 100](./Triangle)** | 🕸️ Web Exploitation | 🟠 Medium | **Backup File Disclosure** → **PHP Type Juggling** (Bypassing 2FA) |
| **[Bad Feedback 100](./Bad-Feedback)** | 🕸️ Web Exploitation | 🟠 Medium | **XXE Injection** (Content-Type Manipulation) |
| **[Nitro Automation](./Nitro%20Automation)** | 🐍 Scripting | 🟠 Medium | **Python Automation** (`requests` + Regex) vs Strict Timeout |

---

## 🔗 More CTF Writeups
This repository is part of my broader cybersecurity journey.
👉 **[View my full CTF Trophy Case & Portfolio here](https://github.com/sohankanna/ctf-trophy-case)**
