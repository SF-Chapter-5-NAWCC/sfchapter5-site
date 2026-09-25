# SF Chapter 5 website

The static website for the Dr. W. Barclay Stephens Memorial SF Chapter 5 of the NAWCC. Every page is plain HTML with a single shared stylesheet. There is no build step, database, PHP or JavaScript. Edit a file and upload it.

## Layout

```
public/                  everything in here is the website
  index.html             home page (collage banner, meeting summary, Dr. Stephens, recent reports)
  meetings/index.html    schedule, location and map, mart rules, what to bring
  membership/index.html  dues, form, NAWCC requirement
  stephens/index.html    Dr. W. Barclay Stephens and the three PDFs
  archive/index.html     list of all meeting reports
  archive/<name>/        one folder per report: index.html plus its photos
  about/index.html       officers, contact email, useful links
  404.html               shown for any address that doesn't exist
  _redirects             old WordPress addresses -> new pages
  assets/css/site.css    all styling
  assets/img/            collage banner, Dr. Stephens portrait, NAWCC logo, favicon
  docs/                  membership form and Dr. Stephens PDFs
wrangler.jsonc           Cloudflare Workers settings (not part of the website)
```

Links between pages are relative (`../meetings/`), so the site works at any address. Folder links like `meetings/` need a web server to resolve to `index.html`, so preview with a local server rather than opening files directly.

## Preview locally

```sh
cd public
python3 -m http.server 8000
```

Then open http://localhost:8000/. Alternatively `npx wrangler dev` from this folder previews it the way Cloudflare will serve it, including redirects and the 404 page.

## Common edits

**Meeting time, place or schedule.** Change it in `public/index.html` (the banner text and the Meetings card) and `public/meetings/index.html`. Nothing else on the site mentions the time.

**Officers or contact email.** Edit `public/about/index.html`. The info@ address also appears in the footer of every page. To change it everywhere:

```sh
grep -rl 'info@sfchapter5.org' public | xargs sed -i '' 's/info@sfchapter5.org/new@example.org/g'
```

**Dues or the membership form.** Edit `public/membership/index.html` and the Join card on `public/index.html`. Replace `public/docs/sfchapter5-membership-form.pdf` with the new form, keeping the same name.

**Add a meeting report.**

1. Copy an existing folder, e.g. `public/archive/wood-repairs/`, to `public/archive/<short-name>/`. Delete the old photos and add the new ones.
2. In the new `index.html` change the `<title>`, the description, the `<h1>`, the date line and the article text. Each photo is `<figure><img src="photo.jpg" alt="what it shows" width="…" height="…" loading="lazy"></figure>`. Resize photos to about 1200px wide before adding them.
3. Add an entry at the top of the list in `public/archive/index.html`.
4. Optionally replace the oldest of the three "From the archive" entries on `public/index.html`.

**Add or rename a main page.** The header and footer are repeated at the top and bottom of every page. Copy the whole `<header>` block from an existing page. If you change the menu, make the same change in every `index.html` (and `404.html`). A search-and-replace across `public/` handles it. The current page's menu link carries `aria-current="page"`.

**Styling.** Colors and fonts are the variables at the top of `public/assets/css/site.css`.

## Deploy to Cloudflare Workers

The site deploys as a Worker with static assets only. There is no Worker code. Settings are in `wrangler.jsonc`:

- `assets.directory` is `./public`, so only the website is uploaded, not this README or the config.
- `html_handling: auto-trailing-slash` serves `meetings/index.html` at `/meetings/`.
- `not_found_handling: 404-page` serves `404.html` for unknown addresses.
- `public/_redirects` sends old WordPress addresses to their new pages with permanent (301) redirects. Cloudflare reads this file and does not serve it.

### First deploy

Requires Node.js and a Cloudflare account.

```sh
cd sfchapter5-claude.org
npx wrangler login        # opens a browser to authorize
npx wrangler deploy
```

The first deploy prints a `https://sfchapter5.<your-subdomain>.workers.dev` address. Check the site there before pointing the real domain at it.

### Updates

Edit files in `public/` then run `npx wrangler deploy` again. Only changed files are uploaded.

### Using the sfchapter5.org domain

1. Add `sfchapter5.org` to your Cloudflare account and switch the domain's nameservers to Cloudflare, as Cloudflare instructs. Do this after the Worker is checked, since it moves the domain away from the current WordPress host.
2. Uncomment the `routes` block in `wrangler.jsonc` and run `npx wrangler deploy`. Cloudflare creates the DNS records and certificates for the domain and `www`. Alternatively, add them in the dashboard under Workers & Pages → sfchapter5 → Settings → Domains & Routes.
3. Keep any existing email (MX) records for `info@sfchapter5.org` when moving DNS.

### Deploying from Git instead

In the Cloudflare dashboard, Workers & Pages → Create → Import a repository. Choose the repository and set the root directory to this folder. No build command is needed; the deploy command is `npx wrangler deploy`. Each push then deploys automatically.

## Old addresses

The WordPress site used addresses like `/info/`, `/directions/`, `/threaded-comments/` (Dr. Stephens) and one address per article at the top level. `public/_redirects` maps each of these to its new page. Each old page is listed with and without its trailing slash. If the site ever moves to a host that doesn't read `_redirects`, those lines would need converting to that host's format; the pages themselves work anywhere.

## Content notes

- Meeting reports are migrated from the WordPress site with their original wording. Formatting was cleaned up: numbered steps became lists, run-in subheadings became headings, "OLYMPUS DIGITAL CAMERA" captions were dropped and every photo has a description.
- Report dates come from the text of each report. The 2013 and 2015 reports only have a year (from when their photos were uploaded).
- The home page banner is the original black-and-white chapter collage (`chapter5_home_page_background_image_bw.jpg`) with its blank white margins cropped off.
- The 9:00 AM start replaces the old 8:30 early setup time. The old early-entry exceptions for out-of-state members and large dealers were removed along with it.
