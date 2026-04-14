# 04 — Solution Design

## Corsario del Mar: Full Solution Design

**"Every can hides a secret. Every secret hides a place."**

---

## 1. The Product Offering

### The Can

**330ml premium craft Tinto de verano** — Spanish red wine blended with natural watermelon soda and one unique locally-sourced botanical ingredient per batch.

| Attribute | Specification |
|---|---|
| Format | 330ml aluminium can |
| ABV | 5% |
| Base | Spanish red wine — Monastrell (Murcia) or Tempranillo (Rioja/Castilla), rotated by batch to reflect the mystery region |
| Core flavour | Natural watermelon essence sourced from Almería province — Spain's primary sandía-producing region and Europe's largest watermelon supplier |
| Hidden ingredient | One additional locally-sourced botanical, herb, salt, or coastal ingredient per monthly batch, unique to the mystery location featured in that can (examples: Galician Atlantic sea salt, La Vera smoked pimentón, Tarifa cliff rosemary, Valencian dried orange blossom) |
| Carbonation | Light natural carbonation — Mediterranean-style, not heavily fizzed |
| Additives | None. No artificial sweeteners, no concentrates, no flavour enhancers |
| Dietary | Vegan. Contains sulphites. Gluten-free. |
| Production | Co-produced with a craft beverage partner in Murcia, Spain. Contract co-packing with full recipe IP retained by Corsario del Mar Ltd. |
| Shelf life | 12 months from production date |
| Packaging | Matte-finish dark navy can with gold foil stamping. Full-bleed illustration of Loro the parrot in corsair aesthetic. Tagline embossed on the base ring. QR code printed on the base, unique per can unit. |

### Brand Identity

- **Name:** Corsario del Mar ("Corsair of the Sea")
- **Mascot:** Loro — an illustrated monk parakeet styled as a corsair's first mate. Weathered, knowing, theatrical. He appears on every surface and narrates every QR experience. He knows where the secret is. He will not tell you directly.
- **Brand DNA:** Spanish maritime heritage, corsair mythology, the pull of the undiscovered. Not kitsch pirate costume — buried history, literary weight, the gravitas of a place that remembers more than it shows.
- **Colour system:** Navy (#0d1b2a), Gold (#c9a84c), Watermelon pink (#e8635a)
- **Typography:** Playfair Display (headings — serif, literary), Inter (body — clean, legible)
- **Tone:** Precise, evocative, slightly mysterious. The brand speaks the way old Fermín would speak — it tells you just enough.

---

## 2. The Digital Experience Layer

### The QR Mechanic — Architecture

Each can carries a **unique, single-use QR code** printed on the base. The code encodes a URL pointing to a mystery experience page hosted on Corsario del Mar's progressive web app (PWA) — no app download required, no account creation at the scan point.

Scanning the QR launches a **five-screen sequential mystery reveal** on the user's mobile browser:

**Screen 1 — "A new mystery awaits"**
Animated brand intro. Loro speaks a single line of voiceover teasing the location without naming it. Duration: 6 seconds. Transition is automatic.

**Screen 2 — The Video**
A 60–90 second cinematic video of the hidden location. Shot in high resolution using drone and ground-level footage. No text overlay, no captions, no geotag. No name is spoken. The footage is atmospheric — dawn light on a cove, the sound of fishing boats, stone walls of an unnamed village — edited to provoke recognition in someone who has been there, and desire in someone who has not. Loro narrates in English, 3–4 sentences, describing what the place *feels* like without identifying it.

**Screen 3 — The Historical Fact**
A single documented story from that location — a specific corsair raid with a date and ship name, a record of a fishing cooperative that supplied the Spanish Armada, a coastal watchtower built against Barbary pirates — sourced from regional Spanish archives and verified for accuracy. Presented as a styled text card. Maximum 120 words. One pull-quote in gold type.

**Screen 4 — The Local Ingredient Reveal**
The specific ingredient used in this can's batch: its name, the name of the producer who supplied it (a real named individual or cooperative), a single sentence about how it is grown or harvested at this location, and a dot on a stylised (non-specific) map of Spain indicating the approximate region only. The producer's first name and a portrait photograph appear — humanising the supply chain and making the ingredient feel like testimony, not marketing.

**Screen 5 — The Photo Challenge**
One cryptic reference photograph of the mystery location — taken from a specific angle and distance, showing a specific compositional detail (a doorway, a rock formation, a painted boat hull, a section of harbour wall). The subscriber's challenge: travel to that location, find that exact viewpoint, and recreate the photograph. Submit the matching image via the upload form embedded in this screen.

**Subscriber authentication:** First-time scanners are shown Screens 1–2 only, then prompted to subscribe or log in. Authenticated subscribers access all five screens. Authentication is handled via email magic link (Resend API) — no password required.

### Technology Stack

| Layer | Tool | Rationale |
|---|---|---|
| Frontend PWA | Next.js (React) | Server-side rendering, fast mobile load, PWA manifest for home screen install |
| CMS (location content) | Sanity.io | Headless, structured content for 500+ locations; custom schema per mystery |
| Video hosting | Cloudflare Stream | Cost-per-minute pricing, global CDN, HLS adaptive streaming |
| QR generation | Custom: UUID per can → encoded URL, printed at co-packer | Each UUID maps to one mystery in the CMS |
| Photo submission | Custom upload portal (Cloudflare R2 storage) + AI similarity pre-screen (CLIP embeddings) | Initial triage; human verification for all finalists |
| Authentication | Resend (magic link email) + JWT session tokens | Frictionless, no password cognitive load |
| Subscription billing | Stripe Billing + ReCharge | Industry standard D2C subscription stack |
| Fulfilment integration | ShipBob (IE + UK) via API | Real-time order sync, tracking, returns |
| Analytics | PostHog (product) + GA4 (marketing) | Full funnel from QR scan to subscription conversion |
| CRM | Klaviyo | Email flows: welcome, mystery reveal reminder, challenge deadline, winner announcement |

---

## 3. Subscription and Prize Mechanics

### Subscription Tiers

| Tier | Price | Contents | Prize Entries |
|---|---|---|---|
| First Mate (monthly) | €25/month | 4 cans · 4 mysteries | Up to 4/month |
| Corsario (6-month) | €22/month (billed €132) | Same + priority dispatch + Loro enamel pin | Up to 4/month + 1 bonus entry |
| Gift Box (3-month) | €75 one-time | 3 months · no renewal | Up to 4/month per recipient |

Cancel or pause anytime. No lock-in beyond the 6-month tier commitment.

### Prize Mechanic — Full Rules of Engagement

- Every verified photo submission earns **one prize draw entry** for that calendar month
- **Four verified submissions in one month** = four entries (one per can in the box)
- Prize draw runs on the **last day of each calendar month**
- One winner drawn per month, publicly announced via email to all subscribers and on brand social channels
- **Grand Prize:** Return flights to Spain (economy, from subscriber's nearest major airport), 3 nights accommodation near the mystery location, a private guided visit to the exact site with a Corsario del Mar representative, and 3 months free subscription
- **Secondary prizes** (drawn for runners-up): 1 month free subscription, Loro merchandise (branded glassware, tote, enamel pin)

### Photo Verification Process

1. Subscriber uploads photo via the in-app form on Screen 5
2. **AI pre-screen** (CLIP-based image similarity scoring) flags matches above threshold → passes to human review queue
3. **Human reviewer** (internal team, SLA: 48 hours) confirms compositional match against the reference image — same angle, same distance, same identifiable landmark or detail
4. Verified submissions receive a confirmation email: "Mystery solved. You are in the draw."
5. GPS metadata is logged (where available) as a secondary verification signal — not the primary criterion, as GPS can be spoofed; the photograph itself is the proof

This verification is rigorous by design. The prize is real. The challenge is real. If the bar is low, the mechanic loses its meaning and the tourism activation does not occur.

---

## 4. Provider Structure and Partnerships

### Corporate Structure

**Corsario del Mar Ltd** — registered in Ireland (EU single market access, D2C subscription regulatory framework, English-language operations base). Production, sourcing, and content operations managed through a Spain-based subsidiary or contracted partner network.

### Production Partner

Co-packing agreement with an established craft beverage producer in Murcia — selected for proximity to Almería watermelon supply, existing wine access (Monastrell DO Jumilla region), and beverage-grade co-packing infrastructure. Corsario del Mar retains full recipe IP, brand ownership, and QR/CMS data. Co-packer produces to specification under NDA.

### Institutional Partnership Targets

| Partner | Role | Value Exchange |
|---|---|---|
| **Turespaña** (Spain's National Tourism Board) | Content co-production and endorsement | Corsario provides measurable footfall data to named regions; Turespaña provides archival content access, location scouting support, and formal endorsement reducing customer acquisition friction |
| **Almería Regional Tourism Agency** | Primary ingredient region partner; pilot location provider | Revenue-sharing on prize trip bookings; naming rights in content for Almería locations |
| **Murcia, Galicia, Valencia regional agencies** | Rotating location and ingredient partners | Same model — economic activation of undiscovered sub-regions in exchange for content and logistical access |
| **Local fishing cooperatives and agricultural producers** | Named ingredient suppliers | Fair-trade sourcing contract; producer named and photographed in every can experience; commercial income from ingredient supply |
| **Regional film schools (Spain)** | Content production partners | Below-market-rate cinematic production in exchange for portfolio content, mentored production experience, and brand credit |

Turespaña partnership is acceleration, not dependency. The model is commercially viable without it. With it, content production costs fall by an estimated 30–40% and institutional legitimacy significantly reduces D2C customer acquisition cost.

---

## 5. Customer Journey — First Scan to Prize Submission

```
[Acquisition]
Social media content (Loro teaser videos, unsolved mystery clips)
→ Paid social (Meta/TikTok) + micro-influencer seeding
→ Landing page (corsariodel mar.com)
→ Subscription sign-up (Stripe/ReCharge)

[Fulfilment]
Order confirmed → ShipBob picks and ships branded box
→ Box arrives (target: within 5 working days of billing date)
→ Unboxing moment: 4 cans, a printed "Crew Brief" (flavour notes + mechanic reminder), Loro postcard

[Engagement — Per Can]
Subscriber opens a can
→ Scans QR on base (mobile camera, no app)
→ Magic link email sent if not authenticated → clicks to authenticate
→ Full 5-screen mystery reveal loads in browser
→ Watches video · reads historical fact · sees local ingredient · views photo challenge

[Discovery — Optional but Designed For]
Subscriber shares the video or mystery on Instagram Stories / TikTok
→ Organic word-of-mouth: "Can you figure out where this is?"
→ Community speculation (subreddit, Discord) — brand-facilitated but user-driven
→ Some subscribers research the location clues obsessively before travelling

[Resolution — Prize Path]
Subscriber travels to Spain (or is already there)
→ Finds the location using clues from video, historical fact, and reference photo
→ Recreates the photograph from the exact spot
→ Uploads via Screen 5 submission form
→ Receives AI pre-screen result within minutes; human verification within 48 hours
→ Confirmation email: "Mystery solved. You are entered in this month's draw."
→ Month-end draw → winner notified publicly

[Retention]
Non-winners remain subscribed (4 new mysteries next month)
Winner's trip documented and published as brand content
Winners become brand ambassadors — organic high-trust acquisition channel
```

---

## 6. Content Production Pipeline

### Location Sourcing and Qualification

Each mystery location must pass a four-point qualification test before entering the content pipeline:

1. **Obscurity:** Not featured in major travel guides, not appearing in top 50 Google results for "Spain hidden gems," not accessible via standard TripAdvisor search
2. **Historical documentation:** At least one verifiable, citable historical record connecting the location to maritime, fishing, corsair, or coastal cultural heritage
3. **Ingredient availability:** A locally-produced botanical, agricultural, or coastal ingredient available in sufficient quantity (minimum: one batch of 10,000 cans) from a named local producer willing to be featured
4. **Photographability:** A specific compositional viewpoint that can be captured as a reference image and recreated by a visitor without specialist equipment — i.e., a real camera angle exists that proves presence at that location

### Production Workflow (Per Location)

| Stage | Task | Owner | Timeline |
|---|---|---|---|
| 1. Scouting | Location identified, qualified, GPS-logged internally | Spain Country Manager | Rolling |
| 2. Research | Historical fact sourced and verified from primary or academic archive | Content Lead | 1 week |
| 3. Ingredient | Producer contacted, contracted, sample tested in batch | Spain Country Manager + Co-packer | 2–4 weeks |
| 4. Production | 1–2 day film shoot (drone + ground crew, 2-person team) | Regional film partner | Per shoot |
| 5. Edit | 60–90 second cut, Loro voiceover recorded, colour grade | In-house or post-production partner | 1 week |
| 6. CMS upload | Video, historical fact, ingredient info, reference photo, producer portrait loaded to Sanity CMS | Content Lead | 2 days |
| 7. QR mapping | UUID batch mapped to CMS location entry, verified against test scan | Developer | 1 day |

**Monthly production requirement:** 4 locations (one per can per box). With 12-week lead time, 3 production cycles run in parallel at all times. Target: 48 locations produced in Year 1; 96 in Year 2.

**Content longevity:** Each location's content is produced once. Once a location has been used as an active mystery, it enters a 24-month lockout before it can be reused. Spain alone — conservatively — provides 500+ qualifying locations, giving 10+ years of unique content at the Year 1 production rate before any location repeats.

---

## 7. Geographic Expansion Roadmap

### Phase 1 — Spain (Year 1–2)

Launch market. 17 autonomous regions available. Priority sourcing sequence driven by existing regional tourism agency partnerships:
- **Andalusia** (Cádiz, Almería, Huelva coast) — Year 1, Months 1–6
- **Murcia and Valencia coast** — Year 1, Months 7–12
- **Galicia (Atlantic coast)** — Year 2, Q1
- **Cantabria and Basque coast** — Year 2, Q2–Q4
- **Balearics and Canaries (interior/lesser-known zones only)** — Year 2 onwards

### Phase 2 — Portugal and Italy (Year 2–3)

- Same model. New waters. Corsario del Mar becomes a platform.
- **Portugal:** Atlantic corsair history, Alentejo coast, Algarve interior, Verde wine region as base spirit
- **Italy:** Sicilian fishing villages, Calabrian coast, Ligurian maritime heritage, Primitivo wine base
- Local mascot variant: a culturally appropriate companion animal to Loro (a seagull for Portugal, an octopus for Sicily — brand extensions, not replacements)
- Turespaña partnership template adapted for Turismo de Portugal and ENIT (Italian national tourism agency)

### Phase 3 — Greece, France, India (Year 3–5)

- **Greece:** Aegean corsair history, island fishing villages, Assyrtiko wine base
- **France:** Atlantic coast, Basque Country, Corsican maritime heritage
- **India:** Goan Portuguese maritime legacy, Malabar coast, Kerala backwater fishing communities — the corsair mythology connects through the Portuguese Estado da India trade routes

### Expansion Principles (Non-Negotiable)

1. Each new country requires a local ingredient, a local historical archive source, and a local production partner before launch — no generic "international" product
2. The mascot Loro remains consistent across all markets — he is a global corsair, not a local character
3. QR content is always produced in English for the Northern European subscriber base; local-language versions added only when subscriber base in that language exceeds 2,000
4. Prize trip destination always matches the mystery location — the winner goes to the place they found. This is not negotiable: it is the brand's entire promise.
