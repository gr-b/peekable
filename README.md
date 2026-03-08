# Peekable

Protect your child from online harm.

Peekable is a desktop app that parents install on their child's computer. It periodically captures screenshots, analyzes them with AI vision, and emails the parent when harmful content is detected.

## How It Works

1. **Screenshots** — Captures the screen every few seconds (configurable)
2. **AI Analysis** — Sends each screenshot to OpenAI GPT-4o vision to check against parent-configured categories
3. **Email Alerts** — When harmful content is detected, emails the parent with a description and the screenshot attached
4. **Cooldown** — Won't send repeat alerts within a configurable window (default 15 minutes)

## Alert Categories

| Category | Description |
|---|---|
| Stranger Interaction | Chatting with unknown people online |
| Adult/Sexual Content | Pornography or explicit material (strict/moderate sensitivity) |
| Violence/Gore | Graphic violent content |
| Cyberbullying | Being bullied or bullying others |
| Self-Harm/Suicide | Content related to self-harm |
| Drug/Alcohol Content | Substance use content |
| Political Content | Political media/discussions (parent describes their concern) |
| Thirst Traps | Provocative social media content |
| Looksmaxxing | Appearance obsession content |
| Gambling | Betting or gambling sites |
| Custom Rule | Parent writes their own monitoring rule |

## Setup

### Prerequisites

- Node.js 18+
- An OpenAI API key (with GPT-4o vision access)
- A Gmail account with an [App Password](https://myaccount.google.com/apppasswords)

### Install

```bash
git clone https://github.com/gr-b/peekable.git
cd peekable
npm install
```

### Configure

Create a `.env` file in the project root:

```
OPENAI_API_KEY=sk-your-key-here
GMAIL_EMAIL=your-email@gmail.com
GMAIL_APP_PASSWORD=your app password
```

### Run

```bash
npm start
```

On first launch, the onboarding wizard will guide you through:

1. **Account setup** — Set your email and a parent password
2. **Screen permission** — Grant macOS Screen Recording permission (guided)
3. **Alert categories** — Choose what to monitor
4. **Sensitivity** — Configure screenshot interval, cooldown, and confidence threshold
5. **Start monitoring** — Confirm and begin

After onboarding, the app runs in the system tray. Closing the window does not stop monitoring.

### Reconfigure

Click the tray icon > Settings, then enter your parent password to access the configuration panel.

## Testing

```bash
npm test
```

Runs 14 Playwright integration tests covering the full onboarding flow, category configuration, sensitivity settings, and password protection.

## Tech Stack

- **Electron** — Desktop app framework
- **OpenAI GPT-4o** — Vision-based screenshot analysis
- **Nodemailer** — Gmail SMTP email delivery
- **electron-store** — Encrypted local configuration
- **screenshot-desktop** — Cross-platform screen capture
- **Playwright** — Integration testing
