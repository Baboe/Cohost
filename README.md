# CoHost Dordogne — cohostdordogne.com

A static, multilingual, deploy-agnostic marketing site for CoHost Dordogne, a
solo remote co-hosting service for absentee owners of holiday homes around
Bergerac, Sarlat, Eymet, Issigeac and Périgueux.

Plain HTML/CSS. No framework, no build step, no external JS libraries, no
hosting-specific features. Open `index.html` directly in a browser and the
whole site works, including navigation between pages.

## Structure

```
/                                    English homepage
/fr/                                 French homepage ("conciergerie Airbnb Bergerac")
/nl/                                 Dutch homepage ("beheer vakantiehuis Dordogne")
/airbnb-management-bergerac/         English local landing page
/reglementation-meubles-tourisme-2025/   French registration guide (informational, not a sales page)
/mentions-legales/                   French legal notice + privacy section (required by French law)
/css/style.css                       Shared stylesheet (all pages)
/assets/images/                      SVG placeholder images + favicon
/robots.txt
/sitemap.xml
/404.html                            Custom not-found page (noindex)
/_redirects                          Netlify: canonical-host redirects
/vercel.json                         Vercel: canonical-host redirects
/.htaccess                           Apache: canonical-host redirects + 404 wiring
```

Every page is a self-contained `index.html` in its own folder, so URLs work
without a `.html` extension on any static host that serves `folder/index.html`
for `folder/` (this is the default behaviour of virtually every static host
and web server).

## Placeholders to replace before launch

Every placeholder in the source is wrapped in `[PLACEHOLDER: ...]` (or
`[PLACEHOLDER PHOTO]` / `[PLACEHOLDER FOTO]` for images) so a find-and-replace
across all files will catch every one. Search the whole project for
`PLACEHOLDER` to find them all; here's what each group means:

| Placeholder | Appears in | What to do |
|---|---|---|
| `[PLACEHOLDER: FORM ENDPOINT URL]` | The `<form action="...">` on every page with a contact form | See **Swapping the contact form endpoint** below |
| `[PLACEHOLDER: STRIPE PAYMENT LINK URL]` | `/sterrendossier/` only — the two "Bestel je Sterrendossier" buttons | See **Swapping the Stripe payment link** below |
| `[PLACEHOLDER PHOTO]` / `[PLACEHOLDER FOTO]` | Hero property photo (SVG placeholder, `assets/images/placeholder-hero.svg`), used on every homepage/landing page and as the default social-share image for every page | Replace the `<img src="...">` with a real photo, update the `alt` text to describe the real image, and remove the `[PLACEHOLDER]` caption text underneath |
| `[PLACEHOLDER: nom de l'hébergeur]` / `[PLACEHOLDER: adresse de l'hébergeur]` / `[PLACEHOLDER: site web ou contact de l'hébergeur]` | `/mentions-legales/` only | Fill in with your actual hosting provider's name, registered address and contact once you've chosen where to deploy — required by French law |
| `[PLACEHOLDER: nom et coordonnées du médiateur de la consommation...]` | `/mentions-legales/` only | French consumer-mediation clause. If you're not required to designate one (check with an accountant/lawyer), you can remove this paragraph instead of filling it in |

**Do not leave any of these visible to a real visitor.** They exist so
nothing false ships by accident — the honesty constraint for this project
was that a brand-new business shouldn't display invented testimonials,
client counts, or contact details, so placeholders stand in until the real
values exist.

Note: the business's legal identity (publisher name, SIRET, registered
address, auto-entrepreneur status), contact email (marloesmotta@gmail.com),
WhatsApp number (+33 7 62 67 89 04) and the founder photo
(`assets/images/founder.jpg`, used on the EN/FR/NL homepages and referenced
in `LocalBusiness` JSON-LD everywhere) are **already filled in** — those are
not placeholders. The IBAN provided during the build was deliberately
**left out of the site entirely**, since a bank account number isn't
required for `mentions légales` and only adds fraud/phishing surface.

Only the **hero property photo** still needs a real image — see the table
above.

## Swapping the Stripe payment link

The Sterrendossier page (`/sterrendossier/`) sells a €149 report. It uses a
plain Stripe **Payment Link** — no Stripe SDK, no JavaScript, no server-side
code, so the site stays a static site.

To set it up: in the Stripe Dashboard go to **Payment links → + New**, create
a link for a €149 one-off product, and copy the resulting URL. Then search
`/sterrendossier/index.html` for `STRIPE PAYMENT LINK` and replace the `href`
value in **both** places (the hero button and the pricing-block button — they
should point at the same link):

```html
<a class="btn btn-primary" href="[PLACEHOLDER: STRIPE PAYMENT LINK URL]">Bestel je Sterrendossier — €149</a>
```

Stripe hosts the checkout page itself, so nothing else needs configuring. Set
the post-payment confirmation/redirect and the receipt email inside the
Stripe Payment Link settings.

Two things worth doing before taking real money: enable a **refund policy**
in Stripe that matches the "Niet tevreden? Geld terug, geen vragen." promise
on the page, and add Stripe to the "Formulaire de contact et service tiers"
and "Cookies et traceurs" sections of `/mentions-legales/` — Stripe's
checkout is a third-party processor handling customer data, so the privacy
section should name it.

## Swapping the contact form endpoint

Each page's contact form has exactly one line to change. Search for
`FORM ENDPOINT URL` and replace the `action` attribute's value:

```html
<form class="contact-form" action="[PLACEHOLDER: FORM ENDPOINT URL]" method="POST">
```

Point it at whatever form backend you choose (Formspree, Basin, Getform, a
custom serverless function, etc.) — most of these services just want that
one URL, and some also want a `name="_replyto"` style hidden field per their
own docs, which you can add alongside the existing `name`, `email`,
`location` and `message` fields.

**No-JS fallback:** every contact section also shows a plain email address
and a WhatsApp link next to the form, so the site keeps working for visitors
with JavaScript disabled or if the form backend is ever down — update those
in the same place you update the email/WhatsApp placeholders above.

If you swap in a third-party form service, add its name and a link to its
privacy policy to the "Formulaire de contact et service tiers" section of
`/mentions-legales/` (already flagged with a placeholder there) and update
the "Cookies et traceurs" section if that service sets cookies.

## Deploying to a generic static host

There is no build step. Deploy the repository contents as-is to any static
host — Netlify, Vercel, GitHub Pages, Cloudflare Pages, S3+CloudFront, or a
plain Apache/Nginx server. In every case:

1. Set the site root to the repository root (the folder containing this
   README and `index.html`).
2. Make sure the host serves `folder/index.html` when a visitor requests
   `folder/` (this is the default for every host listed above — nothing to
   configure).
3. Point the domain `cohostdordogne.com` at the host once DNS is ready.
4. If your host doesn't auto-detect `robots.txt` and `sitemap.xml` at the
   root, no action is needed — they're already plain files at the root and
   will be served like any other static file.
5. **Enforce the canonical host — see below.** This step is not optional:
   every page's `<link rel="canonical">` and JSON-LD point at
   `https://cohostdordogne.com` (HTTPS, no `www`), so `http://` and `www.`
   must redirect there or Google will index them as separate, competing
   pages (this already happened once — see "Canonical host" below).

No environment variables, no `.env` file, no server-side code, no database.

## Canonical host (important — read before going live)

Every page declares `https://cohostdordogne.com` (HTTPS, apex/no-`www`) as
canonical via `<link rel="canonical">`, and `sitemap.xml` and every
`hreflang`/JSON-LD `url` field agree. But **canonical tags are only a
hint** — they don't stop Google from crawling and indexing `http://`,
`www.`, or any other host variant that actually returns 200 OK. The only
reliable fix is a **301 redirect at the hosting/DNS layer** that sends
every variant to the canonical URL before the page is ever served.

This repo ships redirect config for the most common static hosts so the
right one is picked up automatically, with no other action needed on
Netlify, Vercel, or plain Apache:

- **Netlify** — `_redirects` (redirects `www.` and `http://` to
  `https://cohostdordogne.com`; Netlify's own HTTPS enforcement handles
  the rest).
- **Vercel** — `vercel.json` (redirects `www.` to the apex; Vercel
  enforces HTTPS on every domain automatically).
- **Apache** (shared/VPS hosting) — `.htaccess` (forces HTTPS and
  redirects `www.` to the apex via `mod_rewrite`; also wires up the
  custom `404.html`).

Hosts that don't read files for this and need dashboard configuration
instead:

- **GitHub Pages** — add a `CNAME` file containing `cohostdordogne.com`
  (apex, no `www`), then in the repo's **Settings → Pages** tick
  **Enforce HTTPS**. If DNS also points `www` at GitHub Pages, GitHub
  redirects it to the apex domain in `CNAME` automatically.
- **Cloudflare Pages** — set `cohostdordogne.com` as the primary custom
  domain in the project's **Custom domains** settings, then add a
  **Bulk Redirect** (or a Redirect Rule) sending `www.cohostdordogne.com/*`
  and `http://cohostdordogne.com/*` to `https://cohostdordogne.com/$1`.
  Cloudflare enforces HTTPS via "Always Use HTTPS" under SSL/TLS settings.

**After changing hosts or DNS, verify the redirect actually fires** —
e.g. `curl -I http://www.cohostdordogne.com/` should return a `301`
pointing at `https://cohostdordogne.com/` — and re-check Google Search
Console's **Pages** report after a few weeks to confirm only the apex
HTTPS URLs are indexed.

## QA checklist

Confirmed before this handoff:

- [x] All internal links resolve — every `<a href>` between the six pages
      points to a folder that exists in this repository (verified with a
      link-crawl script against the local file tree)
- [x] hreflang is reciprocal — `/`, `/fr/` and `/nl/` each list all three
      language alternates plus `x-default`, and `sitemap.xml` mirrors the
      same three-way set
- [x] No invented facts — no testimonials, client counts, review scores, or
      "years of experience" claims anywhere; the "Why owners choose to work
      with me" section explicitly states this is a new business instead
- [x] No placeholder text left where a real visitor would mistake it for
      finished content — every placeholder uses the `[PLACEHOLDER: ...]`
      convention so it's unmistakable in the source and needs a deliberate
      find-and-replace pass (see table above) before launch
- [x] All three languages complete — homepage content (not machine-translated)
      exists natively in English, French and Dutch, each with its own
      `<title>`, meta description, JSON-LD and ~800+ words of body copy
- [x] Single `<h1>` per page, descriptive `alt` text on every image
      placeholder, visible focus states, keyboard-operable navigation
      (including the mobile menu, which uses `<details>`/`<summary>` — no
      JavaScript required anywhere on the site)
- [x] Every color pairing introduced during design iteration was checked
      against WCAG AA contrast ratios (documented in the build conversation)

Still outstanding — these require the real business details above, not
something a QA pass can catch on its own:

- [ ] Replace every `[PLACEHOLDER: ...]` per the table above
- [ ] Add a real hero property photo and update its `alt` text (founder
      photo is already in place)
- [ ] Point the contact form at a real endpoint and test a live submission
- [ ] Fill in the hosting-provider block in `/mentions-legales/` once a host
      is chosen
- [ ] Decide on the consumer-mediation clause in `/mentions-legales/` with
      an accountant or lawyer, and either fill it in or remove it
