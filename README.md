# bg-rpc-watchdog

Health monitor for the BuidlGuidl RPC stack. Every five minutes it GETs each service’s `/watchdog` endpoint, expects JSON `{ "ok": true }`, and sends a Telegram message when something goes down or comes back.

This process does not serve RPC traffic. The proxy, pool, and web server each expose `/watchdog`; this service is the client that watches them.

## What it checks

| Name | URL |
| --- | --- |
| Public RPC (pre-proxy) | `https://mainnet.rpc.buidlguidl.com/watchdog` |
| RPC Proxy | `https://$RPC_HOST:48544/watchdog` |
| RPC Pool | `https://$RPC_HOST:48546/watchdog` |
| RPC Web Server | `https://$RPC_HOST:48547/watchdog` |

A staging public URL is in `watchdog.js` but commented out. Each probe times out after 10 seconds. Self-signed certs are accepted (`rejectUnauthorized: false`).

## Alerts

Status is stored per endpoint so Telegram is not spammed:

- First failure after a success (or on the first check if it is already down) → 🔴 down, with URL, error, and timestamp
- First success after a failure → 🟢 recovery

Healthy checks only log to stdout.

## Config

Copy `.env.example` to `.env`:

| Variable | Purpose |
| --- | --- |
| `RPC_HOST` | Host for the proxy / pool / web `/watchdog` URLs |
| `TELEGRAM_BOT_TOKEN` | Bot token |
| `TELEGRAM_CHAT_IDS` | Comma-separated chat IDs |

## Layout

```
watchdog.js              Probe loop, status map, Telegram send
utils/telegramUtils.js   Shared Telegram helper (not used by watchdog.js)
.env.example             Required environment variables
```

## Run

```bash
yarn install
node watchdog.js
```

On this host it runs as the PM2 process `watchdog`.

## License

MIT
