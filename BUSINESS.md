# Boo & Co. / Happy Hive — the operating plan

Built 2026-08-09. Everything below is either already running or is a decision
made from verified 2026 pricing, with the source named.

**Live storefronts:** https://suhasc-hue.github.io/boostudio-live/
(`/boo/` and `/hive/` — GitHub Pages, free, no server to run or patch.)

---

## 1. Credit discipline — what this cost

The hard ceiling was 5-7% of available AI/cloud credits. Actual spend on
generating this phase: **zero cloud credits, zero paid API calls.**

| Thing we needed | Expensive way | What we did instead |
|---|---|---|
| ~110 book illustrations | generate new art | **mined from 260+ reels we already rendered** on our own GPUs (`engine/mine_frames.py`) |
| Colouring pages | generate line art | **OpenCV region segmentation** of the same frames (`engine/linework.py`) |
| Book layout + PDF | paid design tool / API | **HTML + CSS printed by headless Chrome** we already had (`engine/book.py`) |
| Product photography | photo shoot / renders | **screenshots of the real interior pages** (`engine/preview.py`) |
| Fonts | licensed type | Fredoka / Baloo 2 / Quicksand (open source) |
| Hosting | paid host | GitHub Pages, free |
| Cart, checkout, file delivery | custom backend | Payhip embeds, free plan |
| Analytics | paid SaaS | privacy-friendly free tier (see §7) |

The only ongoing AI cost is new *video* generation, which runs on our own
machines and costs electricity, not credits.

---

## 2. What exists right now

**Products built and sellable (PDF, print-ready A4):**

| Brand | Product | Pages | Price |
|---|---|---|---|
| Boo | The Garden That Woke Up (storybook) | 13 | ₹349 |
| Boo | Colour Boo's Garden | 10 | ₹199 |
| Boo | The Boo Bundle | 23 + poster | ₹499 |
| Hive | Ten Minutes of Quiet (storybook) | 13 | ₹349 |
| Hive | Colour the Chaos | 12 | ₹199 |
| Hive | New Parent Survival Kit | 25 + prints | ₹499 |

Plus free lead magnets on both sites: a 4-page story sampler, a 2-page
colouring sampler and a phone wallpaper.

**The one-asset-many-products engine works.** One story JSON
(`stories/*.json`) plus the shared frame pool produces the storybook, the
colouring book, the samplers, the covers, the preview images and the poster.
Adding a story is one JSON file and one command.

---

## 3. Payments — decision

**Launch stack: Payhip (free) + Razorpay (domestic) + PayPal (international).**

Why this and not the obvious alternatives:

- **Payhip is the only platform that carries digital AND physical products**,
  has built-in secure file delivery with PDF stamping, and embeds as a buy
  button on a static site. Free plan is 5% and it is not a merchant of record,
  so **money lands in our own Razorpay account in INR at T+1** — we keep the
  merchant relationship. It still remits EU/UK VAT and US/CA sales tax, which
  is the only foreign tax exposure that matters at our size.
- **Razorpay** gives UPI-native Indian checkout at 2% + GST, and CKYC
  activation now completes in minutes.
- **PayPal Business** covers international buyers from day one with no
  approval wait, and issues free weekly FIRA for EDPMS/GST evidence.
- **Rejected:** Stripe (India is invite-only, effectively unavailable),
  Lemon Squeezy (its stated migration path is Stripe Managed Payments, which
  India cannot use), Paddle (bans physical goods; sub-$10 needs custom
  pricing), Instamojo (10% + ₹3 for digital delivery, charges international
  customers in INR), PhonePe (Mastercard-only, non-DCC internationally).

Cost on a ₹499 digital sale, domestic: 5% Payhip + 2% Razorpay + GST on the
2% ≈ **7.4%**, so we net about ₹462.

---

## 4. Print on demand — routing decision

There is no single provider. The split, from verified 2026 pricing:

| Order | Provider | Landed cost | Why |
|---|---|---|---|
| **Book, India, catalogue title** | **Pothi.com** | ₹292 for 24pp A4 colour | 71% margin at ₹999. No API — one manual order-entry step per sale. |
| **Book, India, personalised** | **Lulu Print API** | ₹1,431 | The only verified fully-automated one-off personalised book printer **with a plant in India**. Needs ₹2,499 retail for 43% margin. |
| **Book, US/UK/EU** | **Lulu Print API** | $15.97 | Prints in the destination country. 47% margin at $29.99. |
| **Colouring book, India** | **Pothi.com** | ₹160 | 84% margin at ₹999. |
| **Posters, India** | **Gelato** | ~₹2,282 A3 delivered | **Gelato prints paper in Maharashtra** — 2-4 day delivery, zero customs, INR billing with a GST invoice we can claim as ITC. |
| **Merch, India** | **Qikink** | kids tee ₹292 | ₹0 platform fee, free COD, cheapest per line, sandbox API. |
| **Merch, US/EU/UK** | **Printify Premium** (US) / **Gelato** (UK/EU) | tee $14.25 landed US | Printify is the only one supporting real neck labels; Gelato prints locally in 14 apparel countries. |

**Hard finding: never ship international POD into India.** No de minimis, duty
is 30.98% of CIF minimum (41.60% if mis-flagged as a gift), no provider offers
DDP to India, and the customer gets a surprise bill at the door. A Printful
kids tee lands at ~₹2,865 in India versus ₹292 from Qikink.

**Also decided:** hardcover is not the hero SKU. Lulu case-wrap 24pp lands at
$22.58, which is negative margin at $19.99. Saddle-stitch and perfect-bound
softcover is where the money is.

---

## 5. Unit economics

Digital, ₹349 storybook, sold domestically:

```
price                        349
Payhip 5%                    -17
Razorpay 2% + 18% GST        -8
generation cost               0   (assets already owned)
----------------------------------
contribution                 324   (93%)
```

Printed 24pp A4 colour book via Pothi at ₹999:

```
price                        999
payment fees ~7.4%           -74
print                       -232
shipping                     -60
packaging                    -15
----------------------------------
contribution                 618   (62%)
```

Personalised book via Lulu India at ₹2,499:

```
price                       2499
payment fees ~7.4%          -185
Lulu print                  -908
Lulu fulfilment + ship      -523
forex markup ~3%            -43
----------------------------------
contribution                 840   (34%)
```

The personalised book is the weakest margin and the highest operational load.
It is still worth running because it is the product people screenshot and
share, but **it must be priced at ₹2,499, not ₹999** — at ₹999 it loses money.

---

## 6. What is NOT built yet, and why

- **Live payment buttons.** Payhip needs an account and Razorpay needs KYC —
  both require the owner. Every product page has the button markup ready; the
  `buy` field in `shop/products.json` just needs the Payhip URL pasting in.
- **Personalised book automation.** The web form, preview and pricing are
  built. The generation path (photo → character → story → PDF → Lulu API) is
  designed but not wired, because Lulu needs an account and the character step
  needs GPU time we are currently spending on the reel channels.
- **Email automation.** Signup captures locally. Brevo free tier (300/day) is
  the pick; needs an account.

---

## 7. What the owner has to do (nobody else can)

Short list, in order:

1. **Payhip account** (free, 5 minutes) → create the 6 products → paste each
   product URL into `shop/products.json` → rerun `python sites/build_site.py`.
   The shop goes live the same day.
2. **Razorpay account + KYC.** Business PAN, current account in the *exact*
   same name, address proof, and one of GSTIN / Udyam / Shop & Establishment.
   CKYC often activates in minutes.
3. **File the LUT (GST RFD-11)** before the first export invoice, and renew
   every April. Without it, exports are not zero-rated.
4. **PayPal Business** for international, day one, no approval wait.
5. **Publish the real registered address and phone** on `legal.html` — the
   policy pages are written but the address line is a placeholder, and
   Razorpay's international approval checks for it.
6. **Pothi proof copy.** Order one 24pp A4 colour saddle-stitch on their
   120gsm coated stock and judge it. Their paper caps at 120gsm, which is the
   one real quality risk in the whole plan.
7. **Lulu sandbox account** (free) so the personalised pipeline can be tested
   end to end without spending anything.
8. **Email Gelato sales** one question: "Do you produce photo books at your
   India facility, and what is the INR price for a 24pp A4 softcover?" If yes,
   the entire book stack collapses into one provider with an API.
9. **Incorporate as Pvt Ltd if it isn't already.** A sole proprietorship
   paying foreign POD suppliers falls under the owner's personal LRS headroom;
   a company does not.

---

## 8. Instagram engine — zero-generation content

The channels already post reels. What is added here is the cheap layer:
existing frames + text, no new generation.

- `assets/frames/{brand}/` is a pool of scored, deduplicated stills. Any of
  them is a story post, a quiz background or a carousel panel.
- Colouring pages double as "print this tonight" story posts, which drive the
  free download, which captures the email.
- Comment mining: each comment about a real toddler incident becomes a story
  beat in the next Hive book. The book then gets sold back to the same parents
  who wrote it, which is the whole flywheel.
