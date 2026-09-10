# Deploy on Deplexo

This fork is adapted from the Railway-oriented Hermes Agent template for generic Docker/PaaS hosts such as Deplexo.

## Required Deplexo settings

- App must expose the port provided by the `PORT` environment variable.
- Mount persistent storage at `/data` if Deplexo supports it. Hermes configuration, sessions, pairing state and generated admin credentials are stored there.
- Optional but recommended: set `HERMES_DASHBOARD_PUBLIC_URL` to the exact public HTTPS URL of the app, for example `https://your-app.example.com`. This fixes OAuth callback URLs for the native Hermes dashboard.
- You can set `ADMIN_USERNAME` and `ADMIN_PASSWORD`. If `ADMIN_PASSWORD` is omitted, the app generates one once and persists it under `/data/.hermes/.admin_password`.

## First boot

1. Open the app URL.
2. Go to `/setup`.
3. Log in with the admin credentials shown in the deployment log on first boot, or the values you set in the environment.
4. Select an LLM provider and enter its API key.
5. Set `LLM_MODEL` through the setup wizard.
6. Enable Telegram and enter the BotFather token.
7. Save and start the gateway.
8. Message the bot from Telegram. Approve the pairing request in `/setup`.

## Important recovery behavior

The gateway is supervised by `server.py`. Unexpected exits are restarted with backoff. Deliberate Stop/Restart actions are serialized so two lifecycle operations cannot race each other.

The native Hermes dashboard is also supervised. If it exits unexpectedly, the proxy attempts to restart it instead of leaving the public dashboard permanently at 503 until redeploy.

Stale `gateway.pid`, `gateway.lock`, and `gateway.sock` files are removed at container boot before Hermes starts.

## Public URL

For Deplexo, prefer:

`HERMES_DASHBOARD_PUBLIC_URL=https://YOUR-DEPLEXO-APP-URL`

Do not set this to the internal `127.0.0.1:9119` address.

## Provider configuration

The setup wizard remains the normal configuration path. The template writes provider credentials to `/data/.hermes/.env` and regenerates `config.yaml` before starting the gateway.

For a custom OpenAI-compatible provider, use the Custom provider fields in `/setup` rather than inventing unrelated environment variable names.
