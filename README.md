# pravingadekar.com

A one-page personal site, styled as a two-sided visiting card. It's plain static files with no build step.

- `public/`: the site (`index.html` with inline CSS and a small flip script, plus the favicon, share image and Beagle mascot).
- Hosting: Cloudflare Pages, connected to this repo. A push to `main` deploys, and the build output directory is `public`.
- Preview locally: `python3 -m http.server -d public 8000`.

The design came from a Claude Design round. The editable source files are kept outside this repo.
