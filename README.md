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
| `[PLACEHOLDER: hello@cohostdordogne.com]` | Every page (footer, contact section, JSON-LD, mentions légales) | Replace with the real contact email, in both the visible text **and** the `mailto:` link |
| `[PLACEHOLDER: +33 6 XX XX XX XX]` | JSON-LD `telephone` field on every page | Replace with a real phone number, or delete the `telephone` line entirely if you don't want one listed |
| `[PLACEHOLDER: +33 6 00 00 00 00]` / `[PLACEHOLDER: 33600000000]` | WhatsApp link in the contact section and footer of every page | Replace both — the visible number and the digits-only version in the `wa.me/` URL (no `+`, no spaces, no leading `0` after the country code) |
| `[PLACEHOLDER: FORM ENDPOINT URL]` | The `<form action="...">` on every page with a contact form | See **Swapping the contact form endpoint** below |
| `[PLACEHOLDER PHOTO]` / `[PLACEHOLDER FOTO]` | Hero and founder image placeholders (SVG files in `assets/images/`) | Replace the `<img src="...">` with a real photo, update the `alt` text to describe the real image, and remove the `[PLACEHOLDER]` caption text underneath |
| `[PLACEHOLDER: nom de l'hébergeur]` / `[PLACEHOLDER: adresse de l'hébergeur]` / `[PLACEHOLDER: site web ou contact de l'hébergeur]` | `/mentions-legales/` only | Fill in with your actual hosting provider's name, registered address and contact once you've chosen where to deploy — required by French law |
| `[PLACEHOLDER: nom et coordonnées du médiateur de la consommation...]` | `/mentions-legales/` only | French consumer-mediation clause. If you're not required to designate one (check with an accountant/lawyer), you can remove this paragraph instead of filling it in |

**Do not leave any of these visible to a real visitor.** They exist so
nothing false ships by accident — the honesty constraint for this project
was that a brand-new business shouldn't display invented testimonials,
client counts, or contact details, so placeholders stand in until the real
values exist.

Note: the business's legal identity (publisher name, SIRET, registered
address, auto-entrepreneur status) is **already filled in** on
`/mentions-legales/` and in the `LocalBusiness` JSON-LD on every page — those
are not placeholders. The IBAN provided during the build was deliberately
**left out of the site entirely**, since a bank account number isn't
required for `mentions légales` and only adds fraud/phishing surface.

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

No environment variables, no `.env` file, no server-side code, no database.

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
- [ ] Add real photos and update their `alt` text
- [ ] Point the contact form at a real endpoint and test a live submission
- [ ] Fill in the hosting-provider block in `/mentions-legales/` once a host
      is chosen
- [ ] Decide on the consumer-mediation clause in `/mentions-legales/` with
      an accountant or lawyer, and either fill it in or remove it
