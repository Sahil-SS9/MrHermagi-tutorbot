# Discord Bot Setup Guide

This guide walks through creating the Discord bot that Miyagi uses for lesson delivery and Q&A.

## Step 1: Create a Discord Application

1. Go to [discord.com/developers/applications](https://discord.com/developers/applications)
2. Click **New Application** → name it "Miyagi" (or your teacher's name)
3. Click **Create**
4. Go to the **Bot** tab in the left sidebar
5. Click **Add Bot** → confirm
6. Under the **Token** section, click **Reset Token** → **Copy** the token
   - ⚠️ Save this somewhere safe. You'll need it for the next step
   - You cannot see it again once you leave this page

## Step 2: Configure Bot Permissions

In the Bot tab, enable the following **Privileged Gateway Intents**:

- ✅ **MESSAGE CONTENT INTENT** (required — allows the bot to read message content)
- ✅ **SERVER MEMBERS INTENT** (recommended — allows the bot to know who's in the server)

Under **Authorization Flow**, enable:
- ✅ **Require OAuth2 Code Grant** — leave disabled

## Step 3: Invite the Bot to Your Server

1. Go to the **OAuth2** → **URL Generator** tab
2. Under **Scopes**, select:
   - ✅ `bot`
   - ✅ `applications.commands` (for slash commands if needed)
3. Under **Bot Permissions**, select:

   | Permission | Why |
   |---|---|
   | ✅ Send Messages | Deliver lessons to forum and Q&A channel |
   | ✅ Send Messages in Threads | Reply within weekly forum threads |
   | ✅ Create Public Threads | Start new weekly lesson threads |
   | ✅ Create Private Threads | Optional — for private lessons |
   | ✅ Send Attachments | Attach HTML files and audio summaries |
   | ✅ Read Message History | Read Q&A channel history for context |
   | ✅ Mention Everyone | Optional — for lesson pings |
   | ✅ Add Reactions | For Q&A emoji feedback |
   | ✅ Embed Links | For resource link previews |
   | ✅ Read Messages / View Channels | Must be able to see the channels |

4. Copy the generated URL at the bottom of the page
5. Open it in your browser
6. Select your Discord server from the dropdown
7. Click **Authorize** — the bot joins your server

## Step 4: Set Up the Bot Token in Hermes

```bash
# Open your Hermes .env file
nano ~/.hermes/.env

# Add or update the Discord bot token:
DISCORD_BOT_TOKEN=your_bot_token_here

# Save and exit
```

## Step 5: Create Discord Channels

Create **two channels** in your Discord server:

### 5a. Forum Channel (for structured curriculum)

1. Right-click your server → **Create Channel**
2. Type: **Forum**
3. Name: `ai-ml-learning` (or your subject, e.g. `pc-building`, `python-fundamentals`)
4. Permissions: ensure the bot has **Send Messages** and **Create Public Threads**

Copy the channel ID:
- Right-click the channel → **Copy Channel ID**
- (Enable Developer Mode in Discord settings if you don't see this option)

### 5b. Text Channel (for Q&A — optional but recommended)

1. Right-click your server → **Create Channel**
2. Type: **Text**
3. Name: `ask-miyagi` (or `ask-your-teacher`)

Copy the channel ID (same method as above).

## Step 6: Update Profile Config

Update `~/.hermes/profiles/mrhermagi/config.yaml` with your channel IDs:

```yaml
# Under discord:
discord:
  channel_prompts:
    "YOUR_QA_CHANNEL_ID": |
      You are Miyagi, a personal AI/ML teacher.
      # ... teacher persona prompt ...
```

## Step 7: Create the Cron Job

```bash
hermes cron create \
  --profile miyagi \
  --schedule "0 7 * * *" \
  --deliver "discord:YOUR_FORUM_CHANNEL_ID" \
  --prompt "$(cat templates/cron-prompt.md)" \
  --name "Miyagi Daily Lesson" \
  --model "kimi-k2.6" \
  --provider "ollama-cloud"
```

## Step 8: Verify It Works

1. Trigger the cron manually: `hermes cron run {job_id}`
2. Check the forum channel for the first lesson thread
3. Send a test message in the Q&A channel
4. Confirm attachments (HTML + audio) appear in the same message as the lesson text

## Permissions Troubleshooting

| Symptom | Fix |
|---|---|
| Bot can't see channel | Check channel permissions → add bot as a member |
| Bot can't send messages | Add ✅ Send Messages permission for bot role |
| "Cannot send messages in a non-text channel" | This is correct for forums — the system handles it. If you see this, the bot is trying to DM the forum channel directly. Use forum channel ID for cron delivery. |
| MEDIA files not attaching | Check file path exists. Check directories are created. Check bot has ✅ Attach Files permission. |
| No audio in message | Audio goes as a separate message after the main lesson text + HTML (Discord limitation). |

## Quick-Reference: Channel IDs

```bash
# Get channel ID (run in Discord with Developer Mode enabled)
# Right-click channel → Copy Channel ID

# Update cron delivery target
hermes cron update {job_id} \
  --deliver "discord:FORUM_CHANNEL_ID:THREAD_ID"
```

## Optional: Separate Bot Per Profile

If you want Miyagi as a standalone bot (separate from your main Kensei bot):

1. Repeat Steps 1-3 for a second Discord application
2. Use the second bot token in the miyagi profile `.env`
3. Run a separate gateway instance for the miyagi profile
4. The cron uses the profile gateway for delivery

This gives you:
- Separate online status (Miyagi shows as its own bot)
- Independent rate limits
- Clean identity separation
