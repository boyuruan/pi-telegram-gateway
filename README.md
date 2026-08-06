# pi-telegram

Telegram DM bridge for pi.

Adapted for the latest version of pi (`@earendil-works/pi-coding-agent`).

## What's new vs. the original

- **Updated package dependencies** — uses `@earendil-works/pi-*` and `typebox` (the current pi package names).
- **`agent_settled` for reply delivery** — replies are sent only after pi has fully settled (no more auto-retry / auto-compact), fixing duplicate and lost replies.
- **Pairing code security** — the first user to DM the bot no longer auto-pairs. `/telegram-setup` generates a one-time code; you send `/start <code>` to pair.
- **`deliverAs: "followUp"`** — queued turns are injected with the correct delivery mode, preventing errors when pi is still settling.
- **429 rate-limit handling** — honors Telegram's `retry_after` in the poll loop.
- **Batched config writes** — `lastUpdateId` is flushed periodically instead of on every update.
- **Temp file cleanup** — downloaded attachments older than 1 hour are cleaned on session start.

## Install

From git:

```bash
pi install git:github.com/boyuruan/pi-telegram
```

Or for a single run:

```bash
pi -e git:github.com/boyuruan/pi-telegram
```

## Configure

### Telegram

1. Open [@BotFather](https://t.me/BotFather)
2. Run `/newbot`
3. Pick a name and username
4. Copy the bot token

### pi

Start pi, then run:

```bash
/telegram-setup
```

Paste the bot token when prompted. A 6-digit **pairing code** will be displayed.

The extension stores config in:

```text
~/.pi/agent/telegram.json
```

## Connect a pi session

The Telegram bridge is session-local. Connect it only in the pi session that should own the bot:

```bash
/telegram-connect
```

To stop polling in the current session:

```bash
/telegram-disconnect
```

Check status:

```bash
/telegram-status
```

## Pair your Telegram account

After `/telegram-setup` (which shows a pairing code) and `/telegram-connect`:

1. Open the DM with your bot in Telegram
2. Send `/start <pairing-code>` (e.g. `/start 123456`)

Once paired, the pairing code is cleared and only your Telegram account can interact with the bot.

## Usage

Chat with your bot in Telegram DMs.

### Send text

Send any message in the bot DM. It is forwarded into pi with a `[telegram]` prefix.

### Send images and files

Send images, albums, or files in the DM.

The extension:
- downloads them to `~/.pi/agent/tmp/telegram`
- includes local file paths in the prompt
- forwards inbound images as image inputs to pi

### Ask for files back

If you ask pi for a file or generated artifact, pi should call the `telegram_attach` tool. The extension then sends those files with the next Telegram reply.

Examples:
- `summarize this image`
- `read this README and summarize it`
- `write me a markdown file with the plan and send it back`
- `generate a shell script and attach it`

### Stop a run

In Telegram, send:

```text
stop
```

or:

```text
/stop
```

That aborts the active pi turn.

### Queue follow-ups

If you send more Telegram messages while pi is busy, they are queued and processed in order.

## Commands (in Telegram DM)

| Command     | Description                              |
| ----------- | ---------------------------------------- |
| `/start <code>` | Pair with the extension             |
| `/status`   | Show model, token usage, and context     |
| `/compact`  | Trigger context compaction               |
| `/help`     | Show available commands                  |
| `stop` / `/stop` | Abort the active pi turn            |

## Streaming

The extension streams assistant text previews back to Telegram while pi is generating.

It tries Telegram draft streaming first with `sendMessageDraft`. If that is not supported for your bot, it falls back to `sendMessage` plus `editMessageText`.

## Notes

- Only one pi session should be connected to the bot at a time
- Replies are sent as normal Telegram messages, not quote-replies
- Long replies are split below Telegram's 4096 character limit
- Outbound files are sent via `telegram_attach`

## License

MIT
