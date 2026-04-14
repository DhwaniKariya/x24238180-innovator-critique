# 05 — Prototype + Design Specifications

---

## Part 1: Design Specifications

### 1.1 Brand Colour Palette

| Token | Name | Hex | Usage |
|---|---|---|---|
| `--navy` | Deep Ocean Navy | `#0A1628` | Primary background, dominant dark surface |
| `--red` | Watermelon Red | `#E8334A` | Alert states, prize mechanic highlights, urgency |
| `--green` | Watermelon Green | `#2D6A4F` | Secondary accent, ingredient cards, nature/provenance signals |
| `--cream` | Sun-Bleached White | `#F5F0E8` | Body text on dark, card backgrounds in light mode |
| `--gold` | Aged Pirate Gold | `#C9A84C` | Primary brand accent — headlines, borders, CTA buttons, Loro's iconography |
| `--navy-mid` | Mid Navy | `#132236` | Card backgrounds, elevated surfaces on navy |
| `--navy-light` | Light Navy | `#1A2F45` | Hover states, secondary card surfaces |
| `--muted` | Muted Grey | `#9CA3AF` | Body copy on dark backgrounds, secondary labels |

**Colour hierarchy rule:** Navy is the world. Gold is the secret. Red is the prize. Green is the land. Cream is the page. Never use red and gold together at equal weight — one must dominate.

---

### 1.2 Typography

**Google Fonts only — no licensing cost, fast CDN delivery.**

| Role | Font | Weight(s) | Usage |
|---|---|---|---|
| Headings | **Playfair Display** | 400 (regular), 700 (bold), 400 italic | All H1–H3, pull quotes, brand statements, Loro's "speech" in the QR experience |
| Body | **Inter** | 300, 400, 500, 600 | All body copy, UI labels, form fields, navigation, data tables |

**Type scale (base 16px):**
- Display: 4rem / 64px — hero statements only
- H1: 2.75rem / 44px
- H2: 2rem / 32px
- H3: 1.375rem / 22px
- Body: 1rem / 16px
- Small / label: 0.75rem / 12px — always uppercase, letter-spacing 0.15em

**Import string:**
```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Inter:wght@300;400;500;600&display=swap" rel="stylesheet">
```

---

### 1.3 Logo and Mascot — Loro the Parrot

**Logo Mark:**
A circular emblem. Outer ring: a braided rope in aged gold (#C9A84C), styled as ship rigging. Inner field: deep ocean navy (#0A1628). Centre illustration: Loro perched on a crossed pair of oars (not swords — oars, referencing both fishing and seafaring). Loro faces left. Above the oars, a small stylised wave. Below, a single five-pointed star.

**Logotype:** "CORSARIO DEL MAR" in Playfair Display Bold, spaced at letter-spacing 0.2em, in gold on navy. All caps. The word "DEL" is set at 60% scale between "CORSARIO" and "MAR" — it visually recedes, making the larger words feel like a title and a place name.

**Loro — Character Specifications (for illustrator brief):**
- Species: Monk parakeet (*Myiopsitta monachus*) — the invasive Spanish urban parrot, chosen deliberately for its cultural specificity
- Pose: Perched slightly forward, weight on left foot, right foot slightly raised — alert, watchful, about to speak
- Colouring: Natural monk parakeet plumage (grey face, green body, lighter grey chest) with deliberate anachronistic additions: a small gold hoop earring on the left ear, a faded red bandana tied loosely around the neck, one slightly ragged wing tip (he has seen things)
- Expression: Knowing. Not cartoonish — this is not a children's mascot. Loro's eye is intelligent, slightly narrowed. He is appraising you.
- Rendering style: Detailed flat illustration with fine line work. No gradients. Consistent with a woodcut or engraving aesthetic — as if Loro appeared in an old ship's log.
- Scale rule: Loro must be legible at 16px (favicon), 64px (can label), and 400px (hero illustration). Design at 400px, verify at smaller sizes before sign-off.

---

### 1.4 Complete User Journey

```
[DISCOVERY]
User sees Corsario del Mar content (TikTok teaser / Instagram Reel / influencer unboxing)
    ↓
Visits corsariodel mar.com landing page
    ↓
Subscribes: email + payment (Stripe Checkout) → ReCharge subscription created

[FULFILMENT]
ReCharge triggers ShipBob fulfilment order
    ↓
ShipBob picks: 4 uniquely-QR-coded cans + branded box + printed Crew Brief card
    ↓
Ships (target: 5 working days from billing date)
    ↓
Subscriber receives box → unboxing moment

[PER-CAN ENGAGEMENT LOOP]
Subscriber opens a can → notices QR on base
    ↓
Scans QR with phone camera (no app, opens mobile browser)
    ↓
PWA loads → Screen 1: animated Loro intro (6 seconds)
    ↓
[If not authenticated] → magic link email sent via Resend → subscriber clicks link → JWT session issued
[If authenticated] → proceeds directly
    ↓
Screen 2: Cinematic video (60–90s, unnamed location, Loro narrates)
    ↓
Screen 3: Historical fact card (120 words max, pull-quote in gold)
    ↓
Screen 4: Local ingredient reveal (producer name + portrait + region dot on map)
    ↓
Screen 5: Photo challenge (reference image + upload form + prize draw entry count)
    ↓
Subscriber either:
  (a) Saves for later — challenge remains open until month-end
  (b) Shares mystery on social (pre-filled text + blurred location still) → organic acquisition
  (c) Immediately begins researching the location

[RESOLUTION — PRIZE PATH]
Subscriber travels to Spain → finds the exact location
    ↓
Recreates the reference photograph from the identical viewpoint
    ↓
Uploads via Screen 5 form (image + email + subscription ID)
    ↓
AI pre-screen (CLIP similarity score) → if above threshold → human review queue
    ↓
Human reviewer verifies match within 48 hours (SLA)
    ↓
Confirmation email: "Mystery solved. You have [N] entries in this month's draw."
    ↓
Month-end draw (last day of calendar month):
  → 1 Grand Prize winner: flights + accommodation + guided site visit + 3 months free
  → 2–3 runners-up: free subscription months + Loro merchandise
    ↓
Winner announced publicly (email to all subscribers + social post)
Winner's trip documented → published as brand content → organic acquisition loop
```

---

### 1.5 QR Code Technical Architecture — Scaling to Millions of Cans

**Core principle:** Every can is unique. Every QR code encodes a unique URL. The URL maps to a specific mystery in the CMS. The mapping is stored in a relational database and linked to the production batch and individual can ID at print time.

**URL structure:**
```
https://app.corsariodelman.com/m/{uuid}
```
Where `{uuid}` is a Version 4 UUID (e.g. `a3f7c912-84b1-4e2d-9c3a-000f12ab45cd`) — 2¹²² possible values, effectively infinite.

**Data flow:**

```
Production Order Placed (e.g. 10,000 cans, Batch #047, Location: Cabo Ortegal)
    ↓
QR Generation Service: generates 10,000 unique UUIDs
    ↓
Each UUID → written to PostgreSQL table: can_registry
  (uuid, batch_id, location_id, production_date, status: 'unscanned')
    ↓
QR codes printed on can bases at co-packer (via EPS vector, print resolution 600dpi minimum)
    ↓
UUID → URL → QR matrix generated by qrcode library at print time
    ↓
[At scan time]
UUID extracted from URL → looked up in can_registry
→ location_id retrieved → fetches content from Sanity CMS via location_id
→ mystery experience rendered for that specific location
→ can_registry row updated: status → 'scanned', scanned_at timestamp, subscriber_id (if authenticated)
```

**Database schema (simplified):**

```sql
-- Locations (content lives in Sanity CMS; this is the relational anchor)
CREATE TABLE locations (
  id          UUID PRIMARY KEY,
  sanity_id   VARCHAR(100) UNIQUE NOT NULL,
  region      VARCHAR(100),
  active_from DATE,
  active_until DATE
);

-- Production batches
CREATE TABLE batches (
  id          UUID PRIMARY KEY,
  location_id UUID REFERENCES locations(id),
  batch_code  VARCHAR(50),
  quantity    INTEGER,
  produced_at DATE
);

-- Individual cans
CREATE TABLE can_registry (
  uuid           UUID PRIMARY KEY,
  batch_id       UUID REFERENCES batches(id),
  status         VARCHAR(20) DEFAULT 'unscanned',
  scanned_at     TIMESTAMPTZ,
  subscriber_id  UUID REFERENCES subscribers(id)
);

-- Photo submissions
CREATE TABLE submissions (
  id             UUID PRIMARY KEY,
  can_uuid       UUID REFERENCES can_registry(uuid),
  subscriber_id  UUID REFERENCES subscribers(id),
  photo_url      VARCHAR(500),
  submitted_at   TIMESTAMPTZ,
  ai_score       DECIMAL(5,4),
  verified       BOOLEAN DEFAULT FALSE,
  verified_by    VARCHAR(100),
  verified_at    TIMESTAMPTZ,
  draw_month     VARCHAR(7) -- e.g. '2026-05'
);
```

**Scale note:** PostgreSQL on a single managed instance (e.g. Supabase or Railway) handles tens of millions of UUID lookups per day at sub-10ms latency with a single index on `can_registry.uuid`. No custom infrastructure required at launch. Migrate to sharded architecture only when can volume exceeds 50 million units per year.

---

### 1.6 Photo Submission Platform Architecture

**Submission flow:**

```
Screen 5 upload form → subscriber selects photo from camera roll
    ↓
Client-side: image resized to max 4MB (Canvas API) → base64 encoded
    ↓
POST to /api/submissions (authenticated endpoint, JWT verified)
Body: { can_uuid, subscriber_id, photo_base64, gps_metadata (optional) }
    ↓
Server: uploads image to Cloudflare R2 (object storage) → returns public URL
    ↓
CLIP embedding generated for submitted image
Reference image embedding retrieved from can_registry → location → CMS
Cosine similarity score computed
    ↓
If score ≥ 0.82 → flagged HIGH confidence match → fast-track human review (24h SLA)
If score 0.60–0.81 → flagged POSSIBLE match → standard human review (48h SLA)
If score < 0.60 → auto-reject → subscriber emailed: "This doesn't look like a match yet"
    ↓
Human reviewer accesses internal dashboard:
  - Reference image on left, submission on right
  - Key compositional elements checklist (provided per location)
  - Approve / Reject / Request resubmission
    ↓
On approval → submissions table updated: verified = TRUE
→ Klaviyo event triggered → "Mystery Solved" confirmation email sent
→ Monthly draw pool updated
```

**Winner verification at draw time:**
All verified submissions for the calendar month form the draw pool. A simple weighted random draw (1 entry per verified submission, up to 4 per subscriber) runs server-side. Winner record written to `winners` table. Notification email dispatched via Klaviyo. Trip booking process initiated by ops team within 48 hours.

---

### 1.7 MVP Feature List with 30 / 60 / 90 Day Timeline

**30 Days — Core QR Experience (Internal Alpha)**

- [ ] Next.js PWA scaffold with brand design system applied
- [ ] QR URL routing: `/m/{uuid}` resolves to a mystery experience page
- [ ] 5-screen mystery reveal flow (static content, single hardcoded location)
- [ ] Cloudflare Stream video player embedded on Screen 2
- [ ] Email magic link authentication (Resend API)
- [ ] Photo upload form → Cloudflare R2 storage → submission record in database
- [ ] Sanity CMS schema defined; one location loaded end-to-end
- [ ] Internal alpha: team of 10 tests full flow on real cans (label prototypes)

**60 Days — Subscription + Closed Beta**

- [ ] Stripe Billing + ReCharge integration live
- [ ] Landing page (`corsariodelman.com/index.html`) live on custom domain
- [ ] 4 mystery locations fully loaded in CMS (enough for one full subscription month)
- [ ] Batch management: UUID generation script + database seeding workflow
- [ ] AI similarity pre-screening (CLIP embeddings) integrated into submission pipeline
- [ ] Internal reviewer dashboard (Retool or custom) for human verification
- [ ] Klaviyo email flows: welcome series, mystery reveal reminder, "solved" confirmation, draw announcement
- [ ] Closed beta: 50 subscribers in Ireland, boxes shipped, first QR experiences live

**90 Days — Public Launch + Prize Draw #1**

- [ ] 12 mystery locations loaded (3 months of content buffer)
- [ ] ShipBob 3PL integration live (automated fulfilment from subscription orders)
- [ ] Age verification at checkout (Stripe Identity, over-18 confirmation)
- [ ] Prize draw mechanics live: monthly draw pool, weighted random draw, winner notification
- [ ] Social share buttons on Screen 5 (pre-filled text + blurred still from location video)
- [ ] Analytics: PostHog product events (scan, screen_advance, submission, share) + GA4 (marketing)
- [ ] Public launch: waitlist converted, paid acquisition begins, first winner announced end of Month 3

---

## Part 2: Prototype Description — prototype.html

The prototype is a **single self-contained HTML file** simulating the complete QR scan-to-submission experience for one specific mystery location. It requires no backend, no API calls, and no login. It is the minimum viable demonstration of the core product interaction — what a subscriber experiences from the moment they scan a can.

### Structure (5 Screens, Slide Navigation)

The page renders as a full-screen mobile-first interface. Screens advance via swipe (touch) or arrow button (desktop). A progress bar at the top shows position within the 5-screen flow.

---

**Screen 1 — Loro Intro**
- Full-screen dark navy background (#0A1628)
- Loro illustration (SVG inline) centred, animates in with a subtle float
- Gold text (Playfair Display italic): *"A new mystery awaits, crew."*
- Inter body text: "Scan complete. Loro has something to show you."
- Auto-advances after 4 seconds OR tap to skip
- A **working QR code** (generated in-browser using `qrcode.js` library) displayed in the bottom quarter of the screen, inside a gold-bordered card
  - Label: "This is what your can's QR code looks like. Scan it."
  - QR encodes the URL of this prototype page — scanning it reopens the same experience, simulating the real mechanic
- A **countdown timer** visible in the top-right corner: "Prize draw closes in: [DD:HH:MM:SS]" — JavaScript calculates time remaining until the last day of the current calendar month at 23:59 UTC

---

**Screen 2 — The Video Reveal**
- Full-bleed video background (autoplay, muted, loop) — a cinematic embed of the Galician coastal location (described in Part 3 below)
- Since this is a static prototype, the video is replaced by a **high-resolution photograph** of Castro de Baroña with a CSS video-filter treatment (subtle brightness reduction, slight desaturation to feel cinematic) and an overlaid animated "play" indicator
- Loro voiceover is represented as styled text in the bottom third of the screen, inside a semi-transparent navy panel:
  *"Somewhere on a coast that has been watching the Atlantic for two thousand years, there is a place that has seen more ships come and go than it will ever admit. The water remembers. The stones remember. You just have to know where to look."*
- No location name. No coordinates. No captions on the image.

---

**Screen 3 — Historical Fact Card**
- Navy card (#132236) with gold left border (3px)
- Section label (gold, 0.7rem, uppercase, letter-spaced): "THE SECRET"
- Heading (Playfair Display, H2): *"The Corsairs Knew This Coast"*
- Body copy (Inter, 0.95rem, cream #F5F0E8):
  The documented historical fact about Castro de Baroña (see Part 3) — approximately 100 words
- Pull-quote in Playfair Display italic, gold: *"The Atlantic was never empty. The question was always who was watching."*
- Source citation in muted grey, 0.7rem: "Source: Consello da Cultura Galega, Maritime Archive, 2019"

---

**Screen 4 — Local Ingredient Reveal**
- Split card: left half watermelon green (#2D6A4F), right half navy
- Left: large illustration of the Padrón pepper (SVG or emoji-based stylised icon), producer name in gold below: **"Xosé Manuel Blanco, A Pobra do Caramiñal"**
- Right: ingredient description (Inter body, cream):
  - Name: **"Wild Padrón Pepper Extract"**
  - Provenance: "Hand-picked from a micro-farm on the hillside above the Ría de Arousa, 12km from this can's hidden location."
  - In the can: "A single-batch cold infusion of dried Padrón peppers added to the watermelon Tinto de verano during secondary conditioning — a faint warmth that arrives after the watermelon, reminiscent of something left in the sun too long."
- Small dotted map of Spain (inline SVG) with a single gold dot on Galicia — no label, no name

---

**Screen 5 — The Photo Challenge + Submission Form**
- Top half: **the reference photograph** (the cryptic challenge image — described in Part 3) in full bleed, with a subtle gold-frame border effect
- Challenge text overlay at the bottom of the image: *"You know what to do. Find this place. Stand exactly here. Take this photograph. Send it to us."*
- Below the image: submission form
  - `<input type="text" placeholder="Your name">` (required)
  - `<input type="email" placeholder="Subscriber email">` (required)
  - `<input type="file" accept="image/*" capture="environment">` — triggers camera on mobile
  - Submit button (gold, full width): "Submit My Discovery"
  - On submit (JavaScript): form validation → success state displayed: "Loro is on his way to check. Watch your inbox." — no real backend call in prototype, purely simulated
- Below the form: **Social Share Buttons**
  - Twitter/X: pre-filled text: `"I think I know where this is. Do you? 🦜🏴‍☠️ @CorsarioDelMar #EveryCanHidesASecret"`
  - WhatsApp: same text
  - Copy link button: copies prototype URL to clipboard
- **Draw countdown** repeated at the bottom: "This month's prize draw closes in [timer]"

---

### Additional Prototype Functionality

- **Keyboard and swipe navigation:** arrow keys (desktop), swipe left/right (mobile touch events)
- **Progress indicator:** 5 gold dots at the top; active dot is filled, inactive are outlined
- **Screen transition:** horizontal slide animation (CSS transform + transition, 300ms ease)
- **Responsive layout:** works at 320px (small mobile) through 1440px (desktop); the experience is centred and max-width capped at 480px on larger screens to simulate phone viewport

---

## Part 3: Hidden Location Selection — Castro de Baroña, Galicia

### The Location

**Castro de Baroña** — a Celtic Iron Age hillfort (*castro*) located on a dramatic rocky promontory at **Porto do Son, A Coruña, Galicia**. The castro sits on a narrow headland surrounded by the Atlantic on three sides, its dry-stone walls still partially intact after two millennia of Atlantic weather. Classified as a *Bien de Interés Cultural* (Spain's highest heritage designation), it receives under 30,000 visitors per year — virtually all of them Spanish — and appears in no major international travel publication. At low tide, the promontory's black volcanic rock is covered with goose barnacles (*percebes*), harvested at dawn by local *percebeiros* exactly as their ancestors harvested them when the Celtic inhabitants built these walls in approximately the 2nd century BCE.

### Three-Sentence Description

Castro de Baroña is a fortified Celtic promontory settlement on the Atlantic coast of Galicia, built in the 2nd century BCE and occupied through the Roman period, its circular stone dwellings still standing to knee height above a coastline of black volcanic rock and cold green Atlantic water. The headland on which it sits — a narrow finger of land accessible only on foot via a path from a small car park — is surrounded on three sides by ocean, the same ocean that Barbary corsairs crossed and English privateers navigated when raiding the Galician coast in the 16th and 17th centuries, using the Ría de Muros e Noia as a hiding position before attacking merchant and fishing vessels heading north toward the Cantabrian ports. It is a place that has watched two thousand years of Atlantic history without ever being named in a guidebook written in English.

### Historical Connection to Piracy and Fishing

The waters of the Ría de Muros e Noia — the estuary immediately east of Castro de Baroña — were a documented operational area for Barbary corsairs in the 16th century and English privateers operating under letters of marque in the same period. The town of Muros, 18km north, was attacked and partially burned by English corsairs in 1589, one year after the Armada's defeat, as Francis Drake's fleet retreated north along the Galician coast. The castro promontory itself, visible from several nautical miles offshore, served as a coastal navigation landmark for centuries of fishing vessels working the sardine and tuna runs of the Rías Baixas. The *percebeiros* of Porto do Son continue to fish and harvest the headland's rocks under a regulated quota system — one of the most dangerous and lucrative artisan food harvesting practices in Europe.

### Local Ingredient for the Corsario del Mar Can

**Wild Padrón Pepper (*Pimento de Padrón*) Cold Extract**
Sourced from small-holding farms in the municipality of Padrón, A Coruña, Galicia — approximately 35km from Castro de Baroña. Padrón peppers (*Capsicum annuum*) are a Protected Designation of Origin (PDO) product unique to this area of Galicia, harvested from July through October. For the Corsario del Mar can linked to this location, a cold-extraction concentrate of dried Padrón peppers is added to the watermelon Tinto de verano base during secondary conditioning — introducing a faint, late-arriving warmth beneath the watermelon sweetness that is unmistakably Galician. The producing farm: a 2.3-hectare family plot worked by **Xosé Manuel Blanco** outside A Pobra do Caramiñal, selling directly to Corsario del Mar under a fair-trade sourcing contract, with Xosé Manuel named and photographed in every Screen 4 ingredient reveal for all cans in this batch.
