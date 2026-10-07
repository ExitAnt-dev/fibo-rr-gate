# ExitAnt Fibo R-R — Gate.io

Deploy this automated trading bot to **your own Render account** as a background
worker. The bot runs from a prebuilt Docker image; this repository contains **only
the Render Blueprint (`render.yaml`)** — no source code.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-gate)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-gate
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker** with a
   **1 GB persistent disk** mounted at `/data` (settings and positions survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for
   them on the deploy screen and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **Gate.io account registered through ExitAnt.** At every start and before every new cycle the bot checks
  your Gate.io UID with the ExitAnt server; only the UID is sent, never your keys.
- **API key** with futures trading permission (Gate.io has no API passphrase). The key must be allowed to
  read the account, trade USDT perpetual futures and — only if you turn on the cash vault — transfer between
  your own wallets.
- **Dual (long/short) position mode** in Gate.io USDT futures. The bot switches the account to it when no
  position or order is open; otherwise set it yourself in Gate.io → Futures → settings.
- If the API key has an **IP whitelist**, add your Render service's outbound IP addresses
  (Render → the service → Settings → Outbound IP addresses). Otherwise Gate.io rejects the bot's calls with
  `IP_FORBIDDEN`; on every start the bot tells you the server's current IP.
- USDT in your Gate.io **USDT futures** account.
- *Optional — only if you turn on the cash vault:* the vault moves USDT between your futures and spot accounts.
  If the transfer is refused, the bot tells you and keeps trading without the vault.

## Environment variables

| Variable | What to enter |
|---|---|
| `API_KEY` | Gate.io API key |
| `SECRET_KEY` | Gate.io API secret |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

Fibo R-R waits for a Bollinger Band breakout on the 5- or 15-minute chart, then enters in up to six steps
(levels) at the spacing you configure, takes profit per level at a Fibonacci ratio towards the opposite
band breakout, keeps one exchange stop-loss for the whole position at your USDT loss limit, and resets the
ladder on an opposite breakout, after a time limit or by hand. A margin guard trims the position when used
margin exceeds your threshold. Every parameter is set in Telegram; twelve languages are supported.

## How it works

- **Prebuilt image.** The worker pulls `docker.io/exitant/exitant-fibo-rr-gate:latest` from Docker Hub. The image ships
  a compiled binary only — no Python source is included or exposed.
- **Background worker.** There is no web page; the bot runs continuously and talks to you through Telegram.
- **Your keys stay yours.** Secrets are typed into your own Render environment and never leave it.
- **Persistent state.** Settings, positions and pending orders live in a small database on the `/data` disk.

## Important

- **Keep the instance count at 1.** Two instances cause a Telegram `getUpdates` conflict.
- Render **background workers have no free plan** — choose Starter or higher.
- **The bot trades live from the first start.** Begin with a small allocation and watch one full cycle
  (breakout → entries → take-profit/stop-loss) on your account before scaling up.
- After the deploy succeeds, send **`/start`** in Telegram to configure the bot.

---

Blueprint spec: <https://render.com/docs/blueprint-spec> · Prebuilt images: <https://render.com/docs/deploying-an-image>
