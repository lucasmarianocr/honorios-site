# Honorio Croxatto — static site

A plain HTML/CSS/JS rebuild of the Wix site, ready for GitHub Pages
(or S3) static hosting. No build step, no dependencies.

```
honorios-site/
├── index.html
├── css/style.css
├── js/main.js
├── robots.txt
└── error.html
```

**🌐 Live at:** https://lucasmarianocr.github.io/honorios-site/

## What changed vs. the Wix export

- All Wix editor/runtime JS, tracking scripts, and CSS bootstrap code were
  stripped out — that machinery only exists to run Wix's visual editor and
  won't work (and isn't needed) once the site is just static files.
- The page content (text, headings, gallery images, exhibitions, quotes,
  contact info) was carried over as-is into plain semantic HTML.
- **Images still load from Wix's media CDN** (`static.wixstatic.com`).
  That CDN is separate from Wix hosting and will keep serving those files
  even after you cancel the Wix site plan — so the migration works as-is.
  If you'd rather own the images outright (see "Hosting your own images"
  below), you'll need to get the original files from the other PC.
- The subscribe form and contact form no longer submit anywhere — Wix's
  form backend goes away with the migration. The form now just validates
  the email client-side. See "Forms on a static site" below for how to
  reconnect it.

## GitHub Pages (current hosting)

The site is already deployed to GitHub Pages at the URL above. To update:

1. Clone: `git clone https://github.com/lucasmarianocr/honorios-site.git`
2. Make your changes, commit, and push to `main` — Pages auto-deploys.

### Custom domain (optional)

To use `honoriosart.com` instead of the `github.io` URL:

1. Go to repo → **Settings** → **Pages** → enter your custom domain.
2. GitHub will verify ownership and auto-provision a TLS certificate.
3. At your domain registrar (or DNS provider), add a CNAME record
   pointing `www.honoriosart.com` → `lucasmarianocr.github.io`, or
   a flat ANAME/ALIAS record for the apex domain.
4. Pages auto-enforces HTTPS — no extra setup needed.

## Hosting your own images (optional)

Right now every `<img>` in `index.html` points at
`https://static.wixstatic.com/media/...`. To bring the images into your
own bucket instead:

1. Download each `mediaUrl` filename referenced in `index.html` (open the
   `https://static.wixstatic.com/media/<filename>` URL and save).
2. Put them in an `images/` folder next to `index.html`.
3. Find-and-replace the `https://static.wixstatic.com/media/...` prefixes
   in `index.html` with `images/<filename>`.

## Forms on a static site

S3 alone can't process form submissions — there's no server to receive
them. The two common fixes, roughly in order of effort:

- **Formspree / Getform / Basin** — point the form's `action` at their
  endpoint, no backend code needed. Fastest option.
- **A mailing-list provider's embed** (Mailchimp, Buttondown) — swap the
  subscribe form markup for their signup snippet.
- **API Gateway + Lambda** — if you want it fully on AWS: a small Lambda
  behind API Gateway that emails you or writes to DynamoDB, called via
  `fetch()` from `js/main.js`.

`js/main.js` currently just validates the email and shows a message — the
`form.addEventListener("submit", ...)` block is where you'd add the
`fetch()` call once you pick one of the above.
