# Runbook: get a domain indexed by Google

Use this for any domain whose DNS is on Cloudflare. It gets the root domain and every subdomain into Google Search Console, verified, with indexing requested.

It was first run on 2026-10-06 for `pravingadekar.com`, which covers the root site on Cloudflare Pages and `beagle.pravingadekar.com` on Firebase Hosting.

In the commands below, replace:

- `DOMAIN`: the registered domain, for example `pravingadekar.com`;
- `SITE`: each site URL to index, for example `https://DOMAIN/` or `https://app.DOMAIN/`.

## 1. Check the site can be indexed

Google skips a page that tells it not to index or that it can't fetch. Run these for each `SITE` before going near Search Console:

```bash
curl -sI SITE | grep -i "^HTTP\|x-robots-tag"
```

```bash
curl -s SITE | grep -io '<meta name="robots"[^>]*>\|<link rel="canonical"[^>]*>'
```

```bash
curl -s SITE/robots.txt
```

Pass when:

- the status is `200`, with no redirect loop;
- there is no `X-Robots-Tag: noindex` header and no `<meta name="robots" content="noindex">`;
- `robots.txt` doesn't `Disallow: /` (a missing `robots.txt` is fine);
- the canonical link, if any, points at the URL you want in search results. Two hostnames serving the same site (for example `*.web.app` and a custom domain) need a canonical pointing at the custom domain, or Google may index the wrong one.

Optional but useful for multi-page sites:

- a `sitemap.xml` that lists each page at its canonical URL;
- a `Sitemap: SITE/sitemap.xml` line in `robots.txt`.

A one-page site doesn't need either.

Also redirect `www` to the root domain (or the reverse) so only one of them gets indexed. In Cloudflare go to the domain, then Rules, then Templates, and use "Redirect from WWW to root". Tick "Preserve query string", then Deploy. The `www` DNS record must be proxied (orange cloud). Check it with:

```bash
curl -sI "https://www.DOMAIN/?x=1" | grep -i "^HTTP\|^location"
```

Expect `301` with `location: https://DOMAIN/?x=1`.

## 2. Add a Domain property

1. Open https://search.google.com/search-console, signed in as the Google account that should own the property.
2. Click Add property (or "Add a website" on a first visit), and choose **Domain**, not URL prefix.
3. Enter `DOMAIN`, then Continue.

A Domain property covers `http` and `https`, `www` and every subdomain, so one verification is enough for all of them. URL-prefix properties need one property per site.

## 3. Verify through a DNS TXT record

Search Console offers to verify "with a few easy steps" by authorizing Google to access your Cloudflare account. **Don't use that option.** It grants Google OAuth access to your DNS. Add the record by hand instead:

1. In the verification dialog, set "Instructions for" to **Any DNS provider**.
2. Copy the value, which looks like `google-site-verification=…`.
3. In Cloudflare, open `DOMAIN`, then DNS, then Records, then Add record, and enter:
   - Type: `TXT`
   - Name: `@`
   - Content: the copied value
   - TTL: Auto
4. Save, then check the record is published:

   ```bash
   dig +short TXT DOMAIN @1.1.1.1
   ```

5. Back in Search Console, click **Verify**. It usually passes within a minute. If it fails, wait a few minutes and retry.

**Keep the TXT record.** Deleting it later removes the verification.

## 4. Request indexing for each site's main page

For each `SITE`:

1. Paste the full URL into the "Inspect any URL" bar at the top and press Enter.
2. Expect "URL is not on Google" for a new site.
3. Click **Request indexing** and wait about 20 seconds for the live test. Expect "Indexing requested".

One request per URL is enough; repeats don't move it up the queue. Request only main pages. Google finds the rest through links and the sitemap.

## 5. Submit sitemaps

For each site that has a sitemap:

1. Go to Indexing, then Sitemaps.
2. Enter the full sitemap URL, for example `https://app.DOMAIN/sitemap.xml`, and click Submit.

The status may show **"Couldn't fetch" right after submitting**. That's common for a new property and usually clears on Google's next pass. Check the sitemap itself is fine:

```bash
curl -sI SITE/sitemap.xml | grep -i "^HTTP\|content-type"
```

```bash
curl -s SITE/sitemap.xml | python3 -c "import sys,xml.dom.minidom as m; m.parseString(sys.stdin.read()); print('xml ok')"
```

Expect `200`, `application/xml` (or `text/xml`) and `xml ok`. If it still says "Couldn't fetch" after a few days, open the sitemap entry in Search Console for the error.

## 6. Afterwards

- **A day later:** the Overview and Performance reports start showing data.
- **Days to weeks:** pages appear in results. Check with a `site:DOMAIN` search, or with URL Inspection, which should say "URL is on Google".
- **Adding a new subdomain later:** it's already covered by the Domain property. Repeat steps 1, 4 and 5 for it.
- **Making a site non-indexable again:** add `noindex` (header or meta tag), then use Indexing, then Removals for a quick temporary hide.

## Notes from the first run

- **Firebase Hosting subdomain:** a Firebase custom domain on a Cloudflare zone needs its CNAME set to **DNS only** (grey cloud) until Firebase issues the certificate. That took about 36 minutes.
- **Cloudflare Pages:** adding a custom domain to a Pages project creates proxied CNAME records automatically.
- **Browser automation:**
  - In Search Console's URL bar, type the URL. Setting the field's value directly doesn't trigger the search.
  - In Cloudflare's add-record dialog, typing failed when the 1Password inline menu was open; setting the field values directly worked.
