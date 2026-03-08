Peekable log


❯ Peekable: Protect your child from online harm                                                                                                 
                                                                                                                                                
You are asked with building Peekable, which is an AI service which parents install on their child’s laptop.                                     
Peekable takes screenshots every 5 seconds and uses OpenAI gpt 5 in a prompt to determine whether the child is currently doing things the       
parent configured them to be alerted on.                                                                                                        
When the AI determines the child is doing something harmful, it emails the parent.                                                              
                                                                                                                                                
For email: I will give you an app password from gmail that you can use to the send the email.                                                   
                                                                                                                                                
This will all run in an electron app on the child’s computer.                                                                                   
The parent will download and setup the application on the child’s computer.                                                                     
During onboarding, the parent will have to enter their email and configure which types of harmful content they want to be notified of.          
Examples:                                                                                                                                       
- Child interacting with strangers (rather than friend, for example on the internet)                                                            
- Political content that disagrees with the parent’ political opinion                                                                           
- Adult themes                                                                                                                                  
- “Thirst traps”                                                                                                                                
- Looksmaxxing                                                                                                                                  
                                                                                                                                                
Thinkt the onboarding flow and how the parent would want to configure when they would be notified.                                              
We need to remember to not send another email if we’ve already triggered in the past 15 minutes.                                                
                                                                                                                                                
The parent will configure a password.                                                                                                           
                                                                                                                                                
How can we make this not detectable by the child, but that the parent can go reconfigure it later?                                              
                                                                                                                                                
                                                                                                                                                
                                                                                                                                                
                                                                                                                                                
Don't start yet, help me figure out how to make the app not easily detectable to children but configurable by parents                           

⏺ Here are the key design considerations for stealth vs. configurability:

  Stealth Mechanisms                                                                                                                            
  
  Process & App Hiding                                                                                                                          
  - Name the process something innocuous (e.g., SystemHealthService, DisplayAdapter) — not "Peekable"
  - No dock icon, no menu bar icon by default
  - Run as a Launch Daemon/Agent so it starts on boot without a visible app
  - Don't appear in the Applications folder — install to /Library/Application Support/ or similar system path

  Tray/UI Hiding
  - No visible window or tray icon during normal operation
  - The app has zero visual footprint when running

  Parent Access (Secret Entry Point)

  A few options, ranked by how kid-proof they are:

  1. Global hotkey combo — e.g., Ctrl+Shift+Option+P opens a password-protected config window. Simple but a tech-savvy kid could discover it via
   key monitoring.
  2. CLI command — parent opens Terminal and runs a specific command like peekable-config which prompts for password. Kids are less likely to
  stumble on this.
  3. Secret URL in browser — parent navigates to http://localhost:<obscure-port>/admin, enters password. Blends in as a generic local service.
  4. Companion app / remote config — parent configures everything from their own phone or computer via a web dashboard. The child's machine has
  zero config UI. This is the strongest option since there's literally nothing to find on the child's machine.

  My Recommendation: Hybrid Approach

  - Option 4 (remote config) as primary — Parent uses a web portal or their phone to change settings. The child's app just polls for config
  updates.
  - Option 2 (CLI) as fallback — For initial setup on the child's machine, and for cases where the parent is physically at the child's computer.

  Anti-Detection Considerations

  ┌────────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────────┐
  │                Concern                 │                                     Mitigation                                     │
  ├────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Activity Monitor shows process         │ Use a generic system-sounding process name                                         │
  ├────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Child searches for "monitoring" apps   │ Don't use obvious names in any file paths                                          │
  ├────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Network traffic to OpenAI is visible   │ Route through your own backend proxy so the domain looks like a generic API        │
  ├────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Notifications/emails tip off the child │ Emails only go to parent; no local notifications                                   │
  ├────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Child checks Login Items               │ Register as a LaunchDaemon (requires admin) so it doesn't show in user Login Items │
  └────────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────────┘

  Setup Flow Sketch

  1. Parent downloads installer, runs it with admin password
  2. Installer places the service in a system directory, registers LaunchDaemon
  3. Onboarding wizard (only shown once at install) collects: parent email, password, alert categories, OpenAI key / app credentials
  4. Config is saved encrypted on disk, wizard closes, app becomes invisible
  5. Future config changes happen via localhost secret URL or remote dashboard

  Key Question for You

  Do you want to build a remote dashboard (parent configures from their own device — more work but much stealthier), or keep it local-only
  (secret hotkey/CLI/localhost — simpler to build)?

  That decision shapes the entire architecture.

❯ Actually let's make it Peekable and run in system tray. Just make it not quittable without force quit. Quitting just closes the render        
window, not the actual process.                                                                                                                 
                                                                                                                                                
GReat now let's create a more extensive plan, thinking more deeply about the parental onboarding.                                               
                                                                                                                                                
Tell me what api keys and stuff like that you need                                                                                              
                                                                                                                                                
⏺ Peekable — Design Plan                                                                                                                        
                                                      
  Architecture
                                                                                                                                                
  Electron app running two layers:
  - Background process (main process) — always running, takes screenshots, calls AI, sends emails. Cannot be quit from the UI. Closing the      
  window just hides it. Tray icon stays.                          
  - Renderer window — only shows during onboarding or when parent authenticates with password to reconfigure.

  What I Need From You

  1. OpenAI API key — for GPT-5 vision calls to analyze screenshots
  2. Gmail address — the "from" address for alert emails
  3. Gmail App Password — 16-character app password (not your regular password). Generate at https://myaccount.google.com/apppasswords

  Onboarding Flow (First Launch)

  Step 1: Parent Account Setup
  - Set a parent password (used to reopen the config window later)
  - Enter parent's email address (where alerts go)

  Step 2: Alert Categories
  Parent toggles on/off and optionally customizes each category:

  ┌─────────────────────┬────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────┐
  │      Category       │            Description             │                             Customizable Detail                              │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Stranger            │ Chatting with unknown people       │ —                                                                            │
  │ interaction         │ online                             │                                                                              │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Adult/sexual        │ Pornography, explicit material     │ Sensitivity slider (strict/moderate)                                         │
  │ content             │                                    │                                                                              │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Violence/gore       │ Graphic violent content            │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Cyberbullying       │ Being bullied or bullying others   │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Self-harm/suicide   │ Content related to self-harm       │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Drug/alcohol        │ Substance use content              │ —                                                                            │
  │ content             │                                    │                                                                              │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Political content   │ Political media/discussions        │ Parent specifies their concern (e.g., "extremist content" or a specific      │
  │                     │                                    │ stance)                                                                      │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Thirst traps        │ Provocative social media content   │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Looksmaxxing        │ Appearance obsession content       │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Gambling            │ Betting or gambling sites          │ —                                                                            │
  ├─────────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────┤
  │ Custom              │ Free-text description              │ Parent writes their own rule                                                 │
  └─────────────────────┴────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────┘

  Step 3: Sensitivity & Frequency
  - Screenshot interval (default 5 seconds)
  - Alert cooldown (default 15 minutes — no repeat emails within this window)
  - Confidence threshold — how certain should the AI be before alerting? (Low/Medium/High)

  Step 4: Confirmation
  - Summary screen of all settings
  - "Start Monitoring" button
  - Window closes, app goes to tray, monitoring begins

  Reopening Config

  - Click tray icon → "Settings" → password prompt → full config UI reopens
  - Parent can also change password from within settings

  Email Alert Content

  Subject: Peekable Alert: [Category] detected

  Body:
  - Timestamp
  - Category triggered
  - AI's brief description of what it saw
  - The screenshot attached (or a link if we store it)
  - "This alert was sent because [category] monitoring is enabled"

  Key Behaviors

  - Window close = hide, app stays in tray
  - Cmd+Q / quit = hide, main process keeps running
  - Tray icon always visible (small, unobtrusive)
  - App launches on system boot (Login Item)
  - Config stored encrypted on disk, keyed to parent password
  - 15-minute cooldown per category (not global — can alert for violence and adult content within the same window)

  Tech Stack

  - Electron (main + renderer)
  - screenshot-desktop for captures
  - OpenAI Node SDK (gpt-5 vision)
  - Nodemailer with Gmail SMTP
  - Electron Store (encrypted) for config
  - electron-builder for packaging

  ---
  Does this plan look right? Once you confirm and provide the three credentials (OpenAI key, Gmail address, Gmail app password), I'll start
  building.

❯ Let's also think about permissions pop up -- how can we guide the user to grant us screen permissions at teh appropriate time in the          
onboarding workflow?                                                                                                                            
                                                                                                                                                
⏺ Screen Recording Permission — macOS                                                                                                           
                                                                                                                                                
  On macOS, screenshot-desktop (or any screen capture) requires Screen Recording permission. The OS shows a system dialog the first time we try 
  to capture, but it's not a simple "Allow" tap — the user has to go to System Settings manually. This is the most friction-prone part of       
  onboarding.                                                                                                                                   
                                                                  
  The Problem

  - macOS doesn't grant permission inline — it just says "App would like to record" and sends you to System Settings
  - The app must be restarted after granting permission for it to take effect
  - If the parent misses this step, monitoring silently fails

  Proposed Flow

  Insert between Step 1 (account setup) and Step 2 (alert categories):

  Step 1.5: Screen Permission Setup

  Screen shows:
  Peekable needs Screen Recording permission to monitor your child's activity.

  1. Click "Open Settings" below
  2. In the System Settings window, find "Peekable" in the list
  3. Toggle it ON
  4. Come back to this window

  [Open System Settings]

  When parent clicks the button, we:
  1. Trigger a test screenshot (this makes macOS show Peekable in the Screen Recording list if it isn't there yet)
  2. Open System Settings directly to the right pane: x-apple.systempreferences:com.apple.preference.security?Privacy_ScreenCapture
  3. Show a "Checking..." spinner that polls every 2 seconds, attempting a test screenshot
  4. Once a screenshot succeeds, auto-advance to Step 2 with a green checkmark: "Permission granted!"

  If the parent proceeds without granting:
  - Show a blocking warning: "Monitoring won't work without this permission. Grant it now or Peekable can't protect your child."
  - Don't allow advancing past this step

  Post-Permission Restart

  Some macOS versions require an app restart after granting Screen Recording. So after detecting the permission grant:

  Permission granted! Peekable needs to restart to activate monitoring.
  Restarting in 3 seconds...

  Auto-relaunch the app via app.relaunch() + app.quit(), then resume onboarding at Step 2 (persist onboarding progress in the store so it picks
  back up).

  Summary of Updated Onboarding

  1. Parent account — password + email
  2. Screen permission — guided grant with polling + auto-restart
  3. Alert categories — toggle and customize
  4. Sensitivity & frequency — intervals, cooldown, confidence
  5. Confirmation — summary + start monitoring