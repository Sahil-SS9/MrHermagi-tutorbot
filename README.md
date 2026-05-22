# MrHermagi-tutorbot 🧘

**A Discord AI tutor bot — daily lessons, curriculum YAML, HTML + audio delivery.**

MrHermagi turns your Discord server into a personal learning academy. Pick any subject, define a curriculum in YAML, and get daily lessons delivered to a forum with full HTML deep-dives and TTS audio summaries for commute listening.

Named after Mr. Miyagi: the student thinks they're learning one thing, but they're learning something deeper underneath. Wax on, wax off.

## What you get

- **Daily lessons** — delivered to a Discord forum at 07:00 UK. One thread per week, lessons as replies
- **Multi-format content** — scannable Discord summary + full dark-mode HTML lesson + TTS audio for commute
- **Interactive Q&A** — dedicated channel where you can ask questions, share links, have conversations
- **Curriculum YAML** — define any subject in a config file. Swap topics without touching infrastructure
- **Teacher persona** — patient, analogy-driven, Socratic. Teaches from first principles, no assumed knowledge

## Quick start

```bash
# 1. Set up your Discord bot (see discord-bot-setup.md)
# 2. Copy the profile
cp -r mrhermagi-tutorbot/profiles/mrhermagi ~/.hermes/profiles/

# 3. Edit your details
nano ~/.hermes/profiles/mrhermagi/USER.md        # Who you are, what you know
nano ~/.hermes/profiles/mrhermagi/curriculum.yaml # Your subject (or use the AI/ML one)

# 4. Set up the Q&A channel prompt in config.yaml
#    Add under discord.channel_prompts:
#    "YOUR_CHANNEL_ID": "You are MrHermagi, ..."

# 5. Create the cron
hermes cron create \
  --profile mrhermagi \
  --schedule "0 7 * * *" \
  --deliver "discord:YOUR_FORUM_CHANNEL_ID" \
  --prompt "$(cat templates/cron-prompt.md)" \
  --name "MrHermagi Daily Lesson"

# 6. Apply the scheduler patch (fixes HTML+audio in one message)
patch ~/.hermes/hermes-agent/cron/scheduler.py < scheduler.diff
# Restart gateway after applying
```

## Structure

```
mrhermagi-tutorbot/
├── README.md                    # This file
├── discord-bot-setup.md         # Step-by-step Discord bot creation guide
├── scheduler.diff               # Patch for combined text+attachment delivery
├── profiles/
│   └── mrhermagi/
│       ├── SOUL.md              # Teacher persona (edit if you want)
│       ├── USER.md              # Your context — who you are, what you know
│       └── config.yaml          # Profile config (model, toolsets, channels)
├── curriculum/
│   ├── ai-ml-learning.yaml      # Full 28-lesson AI/ML curriculum (example)
│   └── template.yaml            # Blank template for your own subject
└── templates/
    ├── cron-prompt.md           # Cron prompt template
    └── lesson-template.html     # Dark-mode HTML lesson template
```

## How to switch subjects

1. Write a new `curriculum.yaml` for the new subject (use `template.yaml`)
2. Move the old one to `curriculum-old.yaml` for reference
3. MrHermagi reads the new YAML and delivers the first pending lesson
4. That's it. No profile changes, no cron rewriting, no gateway restarts

### Example: Switch from AI/ML to AI-Computing (Build Your Own PC)

```yaml
# curriculum.yaml
meta:
  title: "AI-Computing: Build Your Own PC"
  description: "From zero hardware knowledge to understanding every component"
  total_days: 28

weeks:
  - week: 1
    theme: "CPU Architecture"
    days:
      - day: 1
        title: "What Is a CPU?"
        topic: "Fetch-decode-execute cycle, cores, threads, clock speed"
      ...
```

Swap the file. Tomorrow's lesson picks up from Day 1.

## Example curriculum included

The `curriculum/ai-ml-learning.yaml` covers a complete 4-week AI/ML foundations sprint:

| Week | Theme | Lessons |
|---|---|---|
| 1 | Foundations: Text Models | Tokens, parameters, inference, model families |
| 2 | All Modalities | Image, voice, audio, video, multimodal |
| 3 | Running & Comparing | Quantisation, benchmarks, token-maxxing |
| 4 | Advanced | Agents, fine-tuning, production, Hermes architecture |

## Prerequisites

- Hermes Agent running with Discord gateway
- Discord server with:
  - One forum channel (for curriculum delivery)
  - One text channel (for Q&A — optional but recommended)
- Bot has Send Messages, Create Threads, and Attach Files permissions

## Built with

- [Hermes Agent](https://hermes-agent.nousresearch.com) — multi-agent orchestration
- Discord.py — Discord gateway integration
- edge-tts — audio summary generation
- No external API costs beyond your existing Hermes provider

## License

MIT. Use it, share it, fork it, sell it. If you improve it, send a PR back.
