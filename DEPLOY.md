# thorudcustombuilders — deployment notes

Live: https://imaginative-cendol-8a600c.netlify.app
Repo: DarkerParker/thorud-custom-builders (public — required, see "Contributor errors" below)
Editor: /admin, auth via DecapBridge (site id 4d0b0356-1046-4359-b494-aa1187527464)

## Files in the repo

    index.html        the site
    404.html          shown for any URL that isn't /, /projects or /contact
    support.js        the runtime index.html loads — MUST sit next to index.html
    content.json      projects, reviews, Instagram link (the editor writes this)
    admin/index.html  loads the editor
    admin/config.yml  editor config + DecapBridge auth
    netlify.toml      routing, cache headers, security headers
    robots.txt        crawl rules + sitemap pointer
    sitemap.xml       the three pages, for search engines
    site.webmanifest  name, colours and icons for "add to home screen"
    favicon.ico       tab icon (also favicon.svg, apple-touch-icon.png)
    fonts/            the five webfont files the site uses
    vendor/           React 18.3.1, the two files support.js needs
    images/brand/     logo-derived social image and app icons
    images/uploads/   photos Andrew uploads land here

No build step, no environment variables, no secrets. Netlify copies the files as-is.
Build command blank, publish directory `.`

`fonts/` and `vendor/` are third-party files served from our own domain rather than from
Google Fonts and unpkg. That removes two external connections from every page load. They
are byte-for-byte the published releases (the React files match the checksums support.js
pins), and support.js still falls back to unpkg if `vendor/` ever goes missing.

## Everything already done

- Repo connected to Netlify, deploying from `main`
- Repo made public (fixes the contributor error, see below)
- DecapBridge site created, PKCE auth wired into admin/config.yml
- Netlify form block pasted into index.html (see "The contact form")

## Remaining setup

1. **Form notifications.** Netlify → Forms → `estimate` → Settings → form notifications →
   add `thorudandrew@gmail.com`. Then submit the form from a phone and confirm it arrives.
2. **Invite Andrew** in DecapBridge, send him the /admin URL and HOW-TO-UPDATE.md.
3. **Instagram feed** — see below.
4. **Domain** — see below, and read "Changing the domain" before you do it.
5. **His photos.** The five project images are stock. He replaces each one in the editor at
   /admin → Projects → Photo. Last thing standing between this and a finished site.
6. **Submit the sitemap.** Google Search Console → add the property → submit
   `https://<your domain>/sitemap.xml`. Do this after the domain is set, not before.

## The contact form

`index.html` has a hidden form right after `<body>` plus one line of script:

```html
<form name="estimate" netlify hidden>
  <input type="text" name="name" />
  <input type="tel" name="phone" />
  <input type="email" name="email" />
  <textarea name="message"></textarea>
</form>
<script>window.THORUD_FORM_ENDPOINT = "/";</script>
```

Netlify detects forms by scanning the served HTML at deploy time. The site renders its
markup with JavaScript, so this hidden copy is what Netlify registers; the script line
tells the real form to POST there instead of opening the visitor's mail app.

**If I ever send you a regenerated `index.html`, this block has to be pasted back in.**
It is not in the source file. Without it the form silently falls back to `mailto:`, which
fails on phones with no mail app configured.

The same goes for everything in `<head>` — the title, description, canonical link, the
Open Graph and Twitter tags, the icons, the `@font-face` rules and the JSON-LD business
listing. Those deliberately sit in the real document head rather than in the template's
helmet block, because Facebook, LinkedIn and iMessage read the raw HTML and never run the
JavaScript that would move a helmet into place. A regenerated file will put them back
inside the helmet block, where crawlers can't see them.

## Instagram feed

The strip near the bottom of the home page shows a fixed set of photos until you give it a
feed.

1. Free account at **behold.so**, connect `@thorudcustombuilders`. Behold can email him an
   authorization link if you don't have his login.
2. Add a **JSON** feed, copy the feed URL.
3. Paste it at /admin → Instagram → Feed link → Publish.

Free tier: 6 posts, refreshed once a day — exactly what the strip shows. No access tokens
sit in the page. If the URL is blank or the service is down, the site quietly falls back to
the fixed photos.

## Domain

Netlify → Domain management → Add a domain. Bought through Netlify it wires itself up;
owned elsewhere, Netlify gives you two nameservers for the registrar. HTTPS is automatic.

### Changing the domain

The site tells search engines its own address, so five places hold the URL and all five
have to change together. Search the repo for `imaginative-cendol-8a600c.netlify.app`:

1. `index.html` — `<head>`: the canonical link, `og:url`, `og:image`, `twitter:image`
   and the URLs inside the JSON-LD block
2. `index.html` — `SITE_URL` at the top of the script at the bottom of the file
3. `robots.txt` — the `Sitemap:` line
4. `sitemap.xml` — all three `<loc>` entries
5. `admin/config.yml` — `site_url`, which powers the "view site" link in the editor

Get one wrong and search engines are told the canonical version of the page lives at the
old address, which quietly keeps the new domain out of results. After the change, resubmit
the sitemap in Google Search Console.

## Pages and URLs

There are three: `/`, `/projects` and `/contact`. They are one HTML file — `netlify.toml`
rewrites the two extra paths to it, and the page swaps its own title, description and
canonical link as you navigate, so each one indexes separately and shares as itself.

Anything else 404s to `404.html` on purpose. Answering every URL with the home page would
have search engines index a pile of duplicates.

## Contributor errors

On Netlify's free plan, private repos allow only one verified contributor, and DecapBridge
commits under Andrew's identity — so on a private repo every publish he makes gets blocked.
The repo is public for this reason. Keep it that way, or move to Netlify Pro.

Nothing in the repo is sensitive: it is all served publicly at the URL anyway, and the
Behold feed URL is designed to be public.

## Costs

- Domain: $12–20/year
- Netlify free plan: 300 credits/month (~15GB bandwidth, ~20 publishes). When credits run
  out the site stops serving rather than auto-billing you.
- Netlify Personal ($9/mo, 1,000 credits) is the sane upgrade if Andrew publishes often.
- Forms are free on all plans. Behold free tier is enough. DecapBridge free.

Realistic total: under $130/year.

## Photo sizes

The editor still uploads photos at full size with no resizing, and phone photos are 3–5MB,
but visitors no longer download them that way. Uploaded photos are requested through
Netlify's Image CDN, which resizes each one to the width actually needed and re-encodes it
as WebP; the page offers five widths and the browser picks one. A phone loads roughly a
400px-wide copy of a card image instead of a 4MB original.

That lives in `optimized()` near the top of the script at the bottom of `index.html`. The
stock Unsplash photos are resized by Unsplash instead, so they don't spend Netlify
bandwidth credits. If a transform ever fails, the page falls back to the original file
rather than showing a gap.

Andrew doesn't have to do anything differently — but one good wide shot per job is still
the right instinct.

## Notes

- Git Gateway is deprecated by Netlify and unavailable on new sites. That's why auth runs
  through DecapBridge. Do not enable Netlify Identity; the site no longer loads it.
- `content.json` is the live source for projects and reviews. The defaults compiled into
  index.html are only a fallback if that file fails to load.
