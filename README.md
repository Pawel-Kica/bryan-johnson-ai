# Bryan Johnson AI

A Telegram bot that answers your health questions as Bryan Johnson would, from what he's doing right now.

**Unofficial, not affiliated with Bryan Johnson.** It's a fan project. It reads what he publishes and answers in his voice.

I built it for myself because I kept asking "what does Bryan do about X" and then digging through his posts. Now I just text it. Before every answer it searches the web for his current protocol, checking his own posts on X and blueprint.bryanjohnson.com, not news headlines. It tells you the cheapest move that gets most of the effect first, and the expensive version only if it's worth it. It remembers you between chats: what you already have, your budget, how you sleep.

## What a chat looks like

Made up, but this is the shape.

> **You:** I keep waking up at 3am. What do you do about it?
>
> **Bryan:** My last meal is before noon, so nothing is digesting when I sleep. Cheapest move for you: finish dinner at least 3 hours before bed. Then make the bedroom cool and fully dark. Give it a week before buying anything.
>
> **You:** I already have blackout curtains.
>
> **Bryan:** Good, then skip that. Next is light before bed: I wear blue-blocker glasses in the evening. A basic pair costs about as much as a lunch. Do exactly what I do there.

It answers in your language, with prices in your currency and shops that ship to you. Medication doses and diagnoses it leaves to your doctor.

## How it works

It runs on [claude-code-telegram](https://github.com/Pawel-Kica/claude-code-telegram), which passes your Telegram messages to Claude Code working inside this folder. `AGENTS.md` is the persona and the rules. `claude-memory/health-profile.md` is what it knows about you, and it writes there as you talk. Both are plain markdown, edit them however you like.

## Setup

You need a Mac or Linux box that stays on, [Claude Code](https://claude.com/claude-code) 2.1.277 or newer logged in with your subscription or an API key, and [uv](https://docs.astral.sh/uv/).

1. Clone this repo.

   ```
   git clone https://github.com/Pawel-Kica/bryan-johnson-ai
   cd bryan-johnson-ai
   ```

2. Make a bot. In Telegram, message [@BotFather](https://t.me/BotFather), send `/newbot`, pick a name and a username. It gives you a token. Treat it like a password.

3. Get your Telegram user ID. Message [@userinfobot](https://t.me/userinfobot), it replies with a number.

4. Install the framework.

   ```
   uv tool install git+https://github.com/Pawel-Kica/claude-code-telegram
   ```

5. Write the config outside the repo so the token never ends up in git.

   ```
   mkdir -p ~/.config/bryan-telegram
   cp .env.example ~/.config/bryan-telegram/.env
   ```

   Open it and set `TELEGRAM_BOT_TOKEN`, `TELEGRAM_BOT_USERNAME`, `APPROVED_DIRECTORY`, `ALLOWED_USERS` and `CLAUDE_MODEL`. The directory is the full path of this folder, `pwd` prints it. The user is your ID from step 3. For the model pick `opus` or `sonnet`. The allowlist matters: anyone can find a bot by username, only listed IDs get answers.

6. Run it.

   ```
   claude-telegram-bot --config-file ~/.config/bryan-telegram/.env
   ```

   Text your bot something like "what time do you go to bed?". Tell it where you live so prices come in your currency.

That's it. Stop it with Ctrl+C. Voice notes work too if you install `ffmpeg` and `whisper-cli`, see the framework README.

### Keep it running on a Mac (optional)

Save this as `~/Library/LaunchAgents/com.you.bryan-bot.plist`, replacing `/Users/you` with your home folder:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.you.bryan-bot</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/you/.local/bin/claude-telegram-bot</string>
        <string>--config-file</string>
        <string>/Users/you/.config/bryan-telegram/.env</string>
    </array>
    <key>WorkingDirectory</key><string>/Users/you/.config/bryan-telegram</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key><string>/Users/you/.local/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
        <key>HOME</key><string>/Users/you</string>
    </dict>
    <key>RunAtLoad</key><true/>
    <key>KeepAlive</key><true/>
    <key>StandardOutPath</key><string>/Users/you/.config/bryan-telegram/bot.log</string>
    <key>StandardErrorPath</key><string>/Users/you/.config/bryan-telegram/bot.log</string>
</dict>
</plist>
```

Stop the manual run first, two copies on one token fight each other. Then:

```
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.you.bryan-bot.plist
```

After editing `.env`, restart with `launchctl kickstart -k gui/$(id -u)/com.you.bryan-bot`. Edits to `AGENTS.md` need no restart.

## Your data

Your profile lives in `claude-memory/health-profile.md` on your machine and nowhere else. The bot creates it from `health-profile.template.md` on your first chat. It's gitignored, so even a public fork won't push it.

## More

[pawelkica.com/bryanjohnsonai](https://pawelkica.com/bryanjohnsonai)

MIT license, see [LICENSE](LICENSE).
