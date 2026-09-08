# Deploying thorudcustombuilders.com

Everything in this folder is the site. It is static files — no server, no build step.

## What's here

    index.html      the site
    content.json    the projects, reviews and Instagram link (this is what the editor writes to)
    admin/          the editor Andrew logs into
    netlify.toml    tells Netlify not to cache content.json
    images/uploads/ where photos he uploads land

## 1. Put it on GitHub

The editor writes changes back to a Git repo, so the site has to live in one.

1. Create a new **private** repo, e.g. `thorud-site`.
2. Upload the contents of this folder to the repo root (not nested in a `deploy/` folder).
3. Make sure the default branch is named `main`. If it's `master`, change `branch: main` in `admin/config.yml`.

## 2. Connect Netlify

1. app.netlify.com → **Add new site** → **Import an existing project** → pick the repo.
2. Leave the build command blank. Publish directory: `.`
3. Deploy. You'll get a `something-random.netlify.app` URL — the site is live at that point.

## 3. Turn on the editor

In the Netlify site dashboard:

1. **Integrations → Identity → Enable Identity.**
2. Identity → **Registration** → set to **Invite only**. (Skip this and anyone can sign up and edit the site.)
3. Identity → **Services → Git Gateway → Enable**.
4. Identity → **Invite users** → enter Andrew's email.

He gets an email, clicks the link, sets a password. From then on he goes to
`yoursite.com/admin`, logs in, and edits. Nothing else to install.

## 4. Make the contact form real

Out of the box the form opens the visitor's email app with the message pre-filled
(a `mailto:`). That works, but it fails silently if they have no email app configured —
common on phones. Netlify Forms posts straight to his inbox instead, free up to 100
submissions/month.

Netlify detects forms by scanning the served HTML at deploy time. This site renders its
markup with JavaScript, so there is no form in the HTML for Netlify to find — you give it
one to detect, and point the live form at it.

Open `index.html` and paste this immediately after the `<body>` tag. Do not edit anything
further down the file; the rest is machine-generated.

```html
<form name="estimate" netlify hidden>
  <input type="text" name="name" />
  <input type="tel" name="phone" />
  <input type="email" name="email" />
  <textarea name="message"></textarea>
</form>
<script>window.THORUD_FORM_ENDPOINT = "/";</script>
```

The hidden form is what Netlify registers. The one line of script tells the real form to
POST there instead of opening a mail app. Then:

Netlify dashboard → **Forms** → **estimate** → **Settings → Form notifications** → add an
email notification to `thorudandrew@gmail.com`.

Test it once from your phone after deploying. If the POST ever fails, the form falls back
to the old `mailto:` behaviour rather than losing the lead.

## 5. Domain

Netlify → **Domain management → Add a domain**. If he buys the domain through Netlify it
wires itself up. If he already owns one elsewhere, Netlify gives you two nameservers to
paste into the registrar. HTTPS turns itself on either way.

Once the domain is live, open `admin/config.yml` and set `site_url` to the real address.
It is currently a placeholder, and it powers the "view site" link inside the editor.

## Instagram feed

The strip near the bottom of the home page shows a fixed set of photos until you give it a
feed. To make it live:

1. Sign up free at **behold.so**, connect the `@thorudcustombuilders` account.
   (If you don't have his login, Behold can email him an authorization link.)
2. Add a **JSON** feed, copy the feed URL.
3. Paste it into the editor at `/admin` → Instagram → Feed link → Publish.

Free tier gives 6 posts, refreshed once a day — exactly what the strip shows. No access
tokens live in the page, so nothing to rotate or leak. If the feed URL is blank or the
service is down, the site quietly falls back to the fixed photos.

## Still placeholder

The five project photos are stock. Andrew replaces each one at `/admin` → Projects → Photo.
That's the last thing standing between this and a finished site.
