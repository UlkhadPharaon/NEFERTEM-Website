# NEFERTEM — Site vitrine « AI Growth Systems »

[![Live](https://img.shields.io/badge/live-nefertem.pages.dev-0a0a0a?logo=cloudflare&logoColor=f6821f)](https://nefertem.pages.dev)
![Stack](https://img.shields.io/badge/stack-Vite%206%20%2B%20React%2019%20%2B%20Tailwind%204-blue)
![Deploy](https://img.shields.io/badge/deploy-Cloudflare%20Pages-orange)

Site vitrine NEFERTEM (Ouagadougou) — la croissance devient un système.

- **Production** : https://nefertem.pages.dev (Cloudflare Pages, projet `nefertem`, branche `main`)
- **Repo GitHub** : https://github.com/UlkhadPharaon/NEFERTEM-Website
- **SPA** : `public/_redirects` (`/* /index.html 200`, `/contact` → `/demarrer`)

## Développement local

**Prérequis :** Node.js 20+.


1. Install dependencies:
   `npm install`
2. Set the `GEMINI_API_KEY` in [.env.local](.env.local) to your Gemini API key
3. Run the app:
   `npm run dev`

## Environment variables

Copy `.env.example` to `.env.local` and fill what you use. Every variable
is optional — the site works without any of them.

| Variable | Usage |
|---|---|
| `VITE_SITE_URL` | Canonical site URL for SEO (default `https://nefertem.africa`) |
| `VITE_ANALYTICS_PROVIDER` | `none` (default) \| `plausible` \| `ga4` |
| `VITE_ANALYTICS_DOMAIN` | Plausible domain (default: current hostname) |
| `VITE_GA4_ID` | GA4 measurement ID (`G-…`), required when provider is `ga4` |
| `LEAD_WEBHOOK_URL` | (server) destination webhook for `POST /api/lead` — Google Apps Script, Airtable, Supabase or n8n |
| `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` | (server) instant lead notification on Telegram |

## Lead API (`POST /api/lead`)

Vercel serverless function (`api/lead.ts`). It validates the payload with
zod, rejects bots via honeypot + a basic per-instance rate limit
(10 req/min/IP), and forwards to `LEAD_WEBHOOK_URL` and/or Telegram.
The client (`/demarrer` wizard) always calls it with a 1.5 s timeout and
opens WhatsApp regardless of the result — a lead is never lost because
the API failed. Failures are reported with the `lead_api_failed` event.

Lead schema: `{id, createdAt, step, business, firstName, whatsapp, city,
sector, pains[], goal, networks[], pageUrl, plan, period, price, src,
utm{source,medium,campaign}, recommendedPlan, completed}`. A partial lead
(`completed:false`) is saved after step 1 and updated at the end.

## Déploiement

Méthode actuelle : **Cloudflare Pages en Direct Upload** (CLI wrangler), repo Git séparé.

1. Build : `npm run build` (sortie `dist/`) — en session agent, wrapper one-shot
   type `build-runner.mjs` pour éviter le guard watch-process.
2. Déploiement production :
   ```bash
   npx wrangler pages deploy dist --project-name nefertem --branch main --commit-dirty true
   ```
   (`--branch main` obligatoire : sans ça le build atterrit sur une branche preview
   et `nefertem.pages.dev` continue de servir l'ancien.)
3. Vérifier le domaine canonique :
   ```bash
   curl -sS -o /dev/null -w '%{http_code}\n' https://nefertem.pages.dev/   # 200
   ```

Connexion Git (auto-deploy sur push) : dashboard Cloudflare → Workers & Pages →
`nefertem` → Settings → Build & deployments → **Connect to Git repository** →
`UlkhadPharaon/NEFERTEM-Website`, branche `main`, build `npm run build`, sortie `dist`,
env `PUPPETEER_SKIP_DOWNLOAD=true`. (L'API refuse `source` sur un projet Direct
Upload — la connexion doit passer par le dashboard.)

L'ancien repo (`Voix-d-or-marketing-offers-2`) reste conservé en remote `old`
(`git remote -v`). Détails et dépannage : `CLOUDFLARE.md`.
