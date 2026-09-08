# thorudcustombuilders — deployment notes

Live: https://imaginative-cendol-8a600c.netlify.app
Repo: DarkerParker/thorud-custom-builders (public — required, see "Contributor errors" below)
Editor: /admin, auth via DecapBridge (site id 4d0b0356-1046-4359-b494-aa1187527464)

## Files in the repo

    index.html        the site
    support.js        the runtime index.html loads — MUST sit next to index.html
    content.json      projects, reviews, Instagram link (the editor writes this)
    admin/index.html  loads the editor
    admin/config.yml  editor config + DecapBridge auth
    netlify.toml      stops content.json being cached
    images/uploads/   photos Andrew uploads land here

No build step, no environment variables, no secrets. Netlify copies the files as-is.
Build command blank, publish directory `.`

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
4. **Domain** — see below.
5. **His photos.** The five project images are stock. He replaces each one in the editor at
   /admin → Projects → Photo. Last thing standing between this and a finished site.

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

Afterwards set `site_url` in `admin/config.yml` to the real address — it powers the
"view site" link inside the editor.

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

The editor uploads photos at full size with no resizing, and phone photos are 3–5MB. One
good wide shot per job, two or three at most. If the count climbs, point the media library
at Cloudinary, which resizes and serves optimized versions automatically.

## Notes

- Git Gateway is deprecated by Netlify and unavailable on new sites. That's why auth runs
  through DecapBridge. Do not enable Netlify Identity; the site no longer loads it.
- `content.json` is the live source for projects and reviews. The defaults compiled into
  index.html are only a fallback if that file fails to load.
