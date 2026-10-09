# pravingadekar.com

A one-page personal site over an animated skeleton watch movement. It's plain static files with no build step.

- `public/`: the site (`index.html` with inline CSS and a small script that draws the movement SVG, plus the favicon, share image and Beagle mascot).
- Hosting: Cloudflare Pages, connected to this repo. A push to `main` deploys, and the build output directory is `public`.
- Preview locally: `python3 -m http.server -d public 8000`.
- Runbooks: `docs/runbooks/` (not deployed). [Google search indexing](docs/runbooks/google-search-indexing.md) covers any Cloudflare-hosted domain.

The design came from a Claude Design round. The editable source files are kept outside this repo.
