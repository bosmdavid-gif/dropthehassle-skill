# Deploy answers and what to do

The CLI and MCP server print the server's message, plus `Next step:` and an instruction for the
agent when there is one. Follow those first. They are written for this.

| Message (start) | Code | What to do |
| --- | --- | --- |
| "This looks like source code, not a built site…" | `source_project` | Install and build (`npm install && npm run build`), then deploy the output folder it names (`dist/`, `build/`, `out/`, `.output/public/`). Next.js needs `output: 'export'` first. |
| "We need a folder with an index.html file…" | `no_index_html` | Upload the folder whose top level has `index.html`. Rename the main page to `index.html` if the site is a single HTML file with another name. |
| "We found more than one site (…)" | `multiple_sites_found` | Several folders each look like a site. Deploy only the one that is the finished site, from the `Folders:` list. |
| "This upload has N files and the limit is …" | `too_many_files` | Usually `node_modules` or `.git` got in. Deploy only the build folder. |
| "…too large…" | `too_large` | Make the build smaller (compress images, move video to a video host). Before a claim the cap is 25 MB, signed in it's 100 MB. |
| "We cannot host executable or archive files (…)" | `forbidden_files` | Remove `.exe .msi .apk .dmg .scr .bat .zip .rar .7z .iso` files from the site folder. |
| "This is an app that needs a server, and we do not run servers…" | `server_app` | The project is a server app (for example `server.js`, `app.py`). Host the server elsewhere (Railway, Render, Fly). A static front end can stay on DTH, and `set_backend` (token) proxies `/api/*` to the server. Don't rewrite the app unless the user asks. |
| "This app runs on a server as it is…" / "This is your app's source code…" / "This is a server build of your app…" | `server_app_detected` | TanStack Start. See below. |
| "You can put 5 new sites online per hour from this network…" / "…20 new sites online per day…" | HTTP 429 | Stop creating new sites. Re-deploy from the linked folder (CLI) or pass `site` with a token instead of making another anonymous site. |
| "No DropTheHassle token set…" | (MCP) | The tool needs an account token. On npm `dropthehassle-mcp` 0.4.2 this includes `search_domain` and `whoami`. Use the hosted connector or the HTTP search in [domains.md](domains.md) for search, or ask the human for a token for account tools. Deploy still works without one. |
| "…that token is not a DropTheHassle token. This site is anonymous…" | (MCP) | `DTH_TOKEN` is a placeholder or a key from another service. The site went live anonymously. Give the human the claim link, and fix or remove the env value. |
| "The saved link is no longer valid (token revoked). Delete .dropthehassle.json and deploy again." | (CLI) | Do what it says. Note: deploying again creates a new site. To update a claimed site, use a dashboard code (`--code`) or the MCP server with a token. |
| "That code did not work…" | (CLI) | The dashboard code is single-use and lasts 15 minutes. Ask the human for a fresh one. |
| "This folder is not linked yet…" | (CLI status/rollback) | `status` and `rollback` need a folder deployed with `--login` or `--code`. After an anonymous deploy they don't work. |

## Updating a site later

- Same machine, same folder: run `npx -y dropthehassle deploy <folder>` again. `.dropthehassle.json`
  makes it update the same site.
- New machine or a cloud session (no link file): a plain deploy makes a **new** site. To update the
  claimed one, either
  - the human opens that site in the dashboard, picks **Update**, and gives you the code. Then run
    `npx -y dropthehassle deploy <folder> --code XXXX-XXXX` (valid 15 minutes, fills that site), or
  - use the MCP server with the human's token: `list_sites`, then `deploy_site` with that site
    (`site` address, or `site_id` on npm 0.4.2).
  - Or the human drops the new folder on the site in the dashboard.
- Without a token, the hosted connector always starts a new site.

## TanStack Start and new Lovable apps

Lovable apps made from 13 May 2026 onward are TanStack Start apps. As they are, they need a
server. DTH can host them once they are built in **SPA mode**, but server functions and `/api`
routes won't work then. Get the code from GitHub first (every Lovable plan can connect to GitHub).
The server's own instruction for the agent is:

> Build this TanStack Start app as static files, without changing how it runs on Lovable. Leave
> vite.config.ts as it is. Create vite.config.dth.ts as a copy of vite.config.ts, then: add
> spa: { enabled: true } to the tanstackStart options (add tanstackStart: { spa: { enabled: true } }
> if there is none), and add nitro: false. Run npm install, then npx vite build --config
> vite.config.dth.ts. Check that dist/client/_shell.html exists. Then list every server function
> (createServerFn, files ending in .functions.ts) and every /api route, because those won't work on
> a static host, and tell me which pages use them. Tell me when dist/client is ready to upload.

Then deploy `dist/client`. If the app relies on server functions, tell the user to keep it on
Lovable or use a host that runs server code. Guides:
https://dropthehassle.com/guides/lovable-custom-domain and
https://dropthehassle.com/guides/move-off-lovable

## Broken pages after a successful deploy

- **Blank page, 404s for `/src/main.tsx`**: the dev `index.html` was published. Build and deploy the
  output folder.
- **CSS/JS 404s**: the build used a sub-path base (for example Vite `base: '/repo-name/'` left over
  from GitHub Pages). The site is served at the root of its host, so set the base to `/` (or `./`)
  and rebuild.
- **Links work from the home page but a refreshed deep link 404s**: a client-side router. Check
  the URL on the live site before changing router code, and tell the user what you saw.
- **The page calls `localhost` or a dev API**: point it at the production API URL and rebuild.
