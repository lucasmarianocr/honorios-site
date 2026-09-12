# Honorio Croxatto — static site

A plain HTML/CSS/JS rebuild of the Wix site, ready for S3 static website
hosting. No build step, no dependencies — just three files plus assets.

```
honorios-site/
├── index.html
├── css/style.css
├── js/main.js
├── robots.txt
└── error.html
```

## What changed vs. the Wix export

- All Wix editor/runtime JS, tracking scripts, and CSS bootstrap code were
  stripped out — that machinery only exists to run Wix's visual editor and
  won't work (and isn't needed) once the site is just static files.
- The page content (text, headings, gallery images, exhibitions, quotes,
  contact info) was carried over as-is into plain semantic HTML.
- **Images still load from Wix's media CDN** (`static.wixstatic.com`).
  That CDN is separate from Wix hosting and will keep serving those files
  even after you cancel the Wix site plan — so the migration works as-is.
  If you'd rather own the images outright (recommended long-term, in case
  Wix ever changes that policy), see "Hosting your own images" below.
- The subscribe form and contact form no longer submit anywhere — Wix's
  form backend goes away with the migration. The form now just validates
  the email client-side. See "Forms on a static site" below for how to
  reconnect it.

## Deploy to S3 (console)

1. **Create the bucket.** S3 console → *Create bucket* → name it exactly
   your domain if you're using a custom domain (e.g. `honoriosart.com`),
   since that's required for a domain-mapped S3 website. Uncheck
   "Block all public access" (static website buckets must be public-readable).
2. **Enable static website hosting.** Bucket → *Properties* → *Static
   website hosting* → Enable. Index document: `index.html`. Error
   document: `error.html`.
3. **Add a bucket policy** so anyone can read the objects (Bucket →
   *Permissions* → *Bucket policy*):

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
       }
     ]
   }
   ```

4. **Upload the files**, keeping the folder structure (`css/`, `js/` as
   subfolders — drag the whole `honorios-site` folder contents in, or use
   "Add folder").
5. **Test it** at the *Bucket website endpoint* URL shown on the Static
   website hosting page (looks like
   `http://YOUR-BUCKET-NAME.s3-website-us-east-1.amazonaws.com`).

## Deploy via CLI (faster for re-uploads)

```bash
aws s3 mb s3://YOUR-BUCKET-NAME
aws s3 website s3://YOUR-BUCKET-NAME/ --index-document index.html --error-document error.html
aws s3 sync . s3://YOUR-BUCKET-NAME/ --exclude "README.md" --exclude ".git/*"
aws s3api put-bucket-policy --bucket YOUR-BUCKET-NAME --policy file://bucket-policy.json
```

## Custom domain + HTTPS (recommended)

Plain S3 website endpoints are HTTP-only. For `https://honoriosart.com`
with the domain your dad already owns:

1. Request a certificate for the domain in **AWS Certificate Manager**
   (must be in `us-east-1` for CloudFront), validate via DNS.
2. Create a **CloudFront distribution** with the S3 bucket's website
   endpoint (not the bucket ARN) as the origin, attach the certificate,
   add the domain as an alternate domain name (CNAME).
3. Point the domain's DNS at CloudFront — either an ALIAS/A record in
   **Route 53** if you move DNS there, or a CNAME at the current
   registrar if it's not the apex domain.

This also gets you CDN caching and compression for free, which the Wix
site had built in.

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
