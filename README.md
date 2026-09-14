# kalix-help — the Worker behind api.kalix.org

One route, `POST /chat`. The browser sends the conversation so far; the Worker
prepends a system prompt containing the published docs (fetched from
`https://kalix.org/llm/corpus.txt` and edge-cached for an hour), calls the
Anthropic Messages API with the corpus in a one-hour prompt cache, and streams
the reply back. It stores nothing.

Limits, all enforced here regardless of what the model does: 6 requests per
IP per minute, 10 turns per conversation, 2,000 characters per message,
800 output tokens per reply.

## Setup, once

1. **Anthropic.** At https://console.anthropic.com create an API key named
   `kalix-help` and set a monthly spend limit (Settings → Limits). When the
   limit is reached the API refuses requests and the widget shows the
   out-of-budget message.

2. **Cloudflare.** Add kalix.org to a Cloudflare account and move its
   nameservers there (this leaves the site on GitHub Pages; only DNS moves).
   The `routes` entry in `wrangler.toml` then binds the Worker to
   api.kalix.org on deploy.

3. **Deploy.**

       npm install
       npx wrangler login
       npx wrangler secret put ANTHROPIC_API_KEY
       npm run deploy

4. **Smoke test.**

       curl -N https://api.kalix.org/chat \
         -H 'Origin: https://kalix.org' -H 'Content-Type: application/json' \
         -d '{"messages":[{"role":"user","content":"How do I define a storage node?"}]}'

## Changing things

- Model: `MODEL` in `wrangler.toml`, then `npm run deploy`.
- Instructions: `INSTRUCTIONS` in `src/index.ts`.
- Docs content: nothing to do here. The site build regenerates the corpus;
  the Worker picks it up within an hour.

## Where this lives

https://github.com/kalix-org/kalix-help — a small repo of its own. It changes
on a different cadence from the engine and needs no engine permissions.
