# Déploiement Cloudflare Pages — NEFERTEM Website

Site statique Vite + React (SPA). Aucune variable d'environnement requise pour le build.

**État actuel (11 nov. 2026)** : projet Pages `nefertem` en **Direct Upload** (CLI),
repo source `UlkhadPharaon/NEFERTEM-Website` → live https://nefertem.pages.dev.

## Déploiement (Direct Upload — méthode active)

```bash
npm run build          # ou wrapper one-shot en session agent (guard watch-process)
npx wrangler pages deploy dist --project-name nefertem --branch main --commit-dirty true
```

- `--branch main` **obligatoire** : sans ce flag le build part sur une branche de
  preview et le domaine canonique `nefertem.pages.dev` sert l'ancien build.
- L'API Cloud refuse de poser `source` sur un projet Direct Upload
  (« cannot update the `source` object in a Direct Uploads project ») — la
  connexion Git passe donc par le dashboard.

## Auto-deploy Git (connexion dashboard, ~2 min)

Dashboard → Workers & Pages → `nefertem` → Settings → Build & deployments →
**Connect to Git repository** :

| Réglage | Valeur |
|---|---|
| Repo | `UlkhadPharaon/NEFERTEM-Website` |
| Branche de production | `main` |
| Framework preset | Vite |
| Build command | `npm run build` |
| Output directory | `dist` |
| Env build | `PUPPETEER_SKIP_DOWNLOAD=true` |
| Node version | 20 (fichier `.nvmrc`) |

Après connexion : chaque push sur `main` redéploie, les PR ont une URL de preview.

## Déjà prêt dans le repo

- `public/_redirects` : `/* /index.html 200` → les routes `/contact`, `/portfolio`, `/securite` fonctionnent en SPA (sinon 404 au refresh).
- `public/_headers` : cache long sur `/assets/*` (fichiers hashés Vite), cache 24h sur `*.splinecode`, en-têtes de sécurité de base.
- `public/logo/nefertem-mark.png` + `nefertem-full.png` : favicon (`index.html`) et image SEO (`Home.tsx`) résolvaient vers `./logo/...` qui n'existait pas dans `dist` (seul `public/` est copié). Corrigé en dupliquant les fichiers dans `public/logo/`.
- `.nvmrc` : `20` (Vite 6 exige Node 20+).
- `vercel.json` conservé : sans effet sur Cloudflare.

## Domaine personnalisé

Pages → **Custom domains** → ajouter le domaine → Cloudflare crée le DNS automatiquement si la zone est chez Cloudflare.

Le SEO (`sitemap.xml`, canonical `VITE_SITE_URL`) référence déjà `https://nefertem.africa` :
rattacher ce domaine au projet Pages quand la zone DNS est prête.

## Dépannage

| Symptôme | Cause probable | Fix |
|---|---|---|
| 404 sur `/contact` au refresh | `_redirects` manquant | Vérifier `public/_redirects` déployé à la racine du `dist` |
| Favicon / image SEO cassés | assets hors `public/` | Remis dans `public/logo/` — ne pas référencer `logo/` racine en URL directe |
| Build error `esbuild` / Node | Mauvaise version Node | Forcer Node `20` dans les réglages Pages |
| Build lent / timeout Chrome | `puppeteer` postinstall | Env `PUPPETEER_SKIP_DOWNLOAD=true` |
| Page blanche | JS bloqué / cache | Rebuild + purge cache Pages, vérifier la console navigateur |
