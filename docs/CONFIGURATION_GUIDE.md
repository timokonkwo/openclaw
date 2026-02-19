# Configuration Guide for Mance (OpenClaw)

This guide will help you set up the specific tools and environment variables needed to empower Mance with email, location, and social media capabilities.

## 1. Environment Variables (.env)

Duplicate the `.env.example` file to `.env` and configure the following:

```bash
cp .env.example .env
```

### Essential Keys
- **TELEGRAM_BOT_TOKEN**: Required for Mance's primary communication channel.
  - *How to get:* Message `@BotFather` on Telegram, create a new bot (`/newbot`), and paste the token here.
- **ANTHROPIC_API_KEY** (or OPENAI_API_KEY): The brain. Anthropic (Claude 3.5 Sonnet/Opus) is recommended for Mance's persona.
- **OPENCLAW_GATEWAY_TOKEN**: Generate a random string (e.g., `openssl rand -hex 32`) for security.

## 2. Setting Up Skills

### Email (Himalaya)
Mance uses `himalaya` to manage emails.
1. **Install CLI:**
   ```bash
   brew install himalaya
   ```
2. **Configure Account:**
   Run the wizard to link Luxen Labs email (`info@luxenlabs.com` / `luxenlabs@gmail.com`):
   ```bash
   himalaya account configure
   ```
   ```
   *Follow the prompts to set up IMAP/SMTP credentials.*

   **Best Practice: Email Identity**
   - **Recommended:** Use a subdomain or alias like `mance@sales.luxenlabs.com` or `mance@luxenlabs.com`.
   - **Why:** This separates the bot's automated outreach from your main domain (`info@luxenlabs.com`), protecting your primary domain's reputation from spam filters during cold outreach.


### Location & Places (GoPlaces)
Allows Mance to find places (restaurants, client offices).
1. **Install CLI:**
   ```bash
   brew install steipete/tap/goplaces
   ```
2. **Configure API Key:**
   Add `GOOGLE_PLACES_API_KEY` to your `.env` file.

### Social Media (X/Twitter) & Research
Mance uses the **Browser** tool for tasks without direct APIs (like posting to X or deep research).
1. **Enable Browser:**
   Ensure `browser` is enabled in your `openclaw.json` (usually enabled by default in the `browser` block).
   ```json
   {
     "browser": {
       "enabled": true
     }
   }
   ```
2. **Login:**
   You will need to log in to `x.com` on the browser instance OpenClaw uses (run `openclaw browser launch` or similar if available, or use the debug UI to log in once).

### Apple Ecosystem (Reminders & Notes)
Since you are on macOS:
- **Apple Reminders:** Mance can natively add value to your "Reminders" app. No extra setup needed, just permissions.
- **Apple Notes:** Mance can read/write notes.

## 3. Knowledge Base
Ensure `knowledge/LUXEN_LABS.md` is kept up to date. Mance treats this as his primary source of truth.

## 4. Automation & Cron Jobs
To make Mance proactive (reminders, standups), you need to configure `cron` jobs in your `openclaw.json` or valid config file.

### Setting up Telegram Topics (Groups & Subgroups)
Mance uses **Topic IDs** to know exactly where to post.

1.  **Add Mance to the Group:**
    - Add `@YourBotName` to your Telegram group.
    - **Promote to Admin:** This is highly recommended so Mance can see messages in all topics and tag people.
    - **Disable Privacy Mode (if not admin):** Talk to `@BotFather` -> `/mybots` -> Select Bot -> Bot Settings -> Group Privacy -> **Turn off**.

2.  **Get Chat ID and Topic IDs:**
    You need two numbers:
    - `CHAT_ID`: The ID of the main group (starts with `-100`).
    - `TOPIC_ID`: The ID of the specific topic (e.g., General, Standup).
    
    **Method A: The Easy Way (@getidsbot)**
    1.  Go to the specific topic (e.g., "Daily Standup").
    2.  Forward any message from that topic to the bot **@getidsbot**.
    3.  It will reply with:
        - `Origin chat`: This is your `CHAT_ID` (e.g., `-100123456789`).
        - `Message thread id`: This is your `TOPIC_ID` (e.g., `42`).

    **Method B: The Logs Way**
    1.  Run OpenClaw locally: `openclaw agent --name Mance`.
    2.  Send a message in the topic: "Mance, are you there?"
    3.  Check your terminal logs. You will see an incoming event with `chat: { id: -100... }` and `message_thread_id: ...`.

3.  **Update `openclaw.json`:**
    Replace the placeholders in the `delivery.to` fields:
    - Format: `CHAT_ID:topic:TOPIC_ID`
    - Example: `-100123456789:topic:42`

**Example Configuration:**
```json
{
  "cron": {
    "jobs": [
      {
        "name": "Daily Standup",
        "schedule": "0 9 * * 1-5", 
        "timezone": "Africa/Lagos", 
        "task": "Daily Standup Routine",
        "message": "It is 9am WAT. Initiate the daily standup in the 'Daily Standup' topic. Tag everyone. Ask for updates."
      },
      {
        "name": "Weekly Meeting Reminder",
        "schedule": "0 20 * * 4",
        "timezone": "Africa/Lagos",
        "task": "Weekly Meeting Check",
        "message": "It is 8pm WAT on Thursday. Remind everyone about the Weekly Meeting on Google Meet. Share the link."
      }
    ]
  }
}
```
*Note: Ensure the timezone matches WAT (Africa/Lagos is GMT+1).*

## 5. Running Mance
**Through CLI (Local/Dev):**
```bash
openclaw agent --name "Mance"
```

**Production (AWS/Docker):**
Refer to `docs/DEPLOY_AWS.md` for server deployment.
