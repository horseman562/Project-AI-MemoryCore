# Restaurant Website Pitch Workflow
*End-to-end process for building demo websites and pitching paid web dev to local F&B owners*

## Overview
- **Trigger**: Identify a local restaurant with active social media but no proper website
- **Actors**: Naqiuddin (developer), AI (research + build + copy), restaurant owner
- **End State**: Demo deployed, pitch messages sent across all contact channels
- **Duration**: ~2–3 hours per restaurant (research 45min, build 60–90min, pitch 15min)

---

## Flow

```
Find target → Research → Analyze brand → Build demo → Deploy → Pitch
```

---

## Step 1: Target Identification
- **Trigger**: Restaurant has Facebook/Instagram/TikTok but no website (or bad website)
- **Input**: Restaurant name + at least one social media URL
- **Output**: Confirmed target with social handles

## Step 2: Research (Playwright MCP + WebSearch + WebFetch)
- **Action**: Scrape all public social media using Playwright browser
  - Facebook page → About tab (navigate directly to `?sk=about` to bypass login dialog)
  - Instagram → use specific post URLs (login required for profiles)
  - TikTok → public, scrape bio + video captions
  - FoodPanda/GrabFood listing → use Playwright (403 on WebFetch)
  - Google Maps → WebFetch or Playwright for address/hours
- **Key data to extract**:
  - Full menu + prices
  - Business hours, address
  - Phone, WhatsApp, email
  - Owner name (check Facebook About + personal profiles)
  - Brand personality (tone of captions, visual style, audience)
  - Social stats (followers, engagement)
- **Output**: `notes.md` in restaurant folder with all data

## Step 3: Brand Identity Analysis
- **Action**: Derive design direction from research
  - Color palette: pull from logo, dominant colors in photos
  - Tone: casual/bold/premium/underground — from caption style
  - Target audience: age, vibe, price range
  - Layout inspiration: what layout matches their energy
- **Output**: UI/UX direction section in `notes.md`

## Step 4: Build Demo Website (Pure HTML/CSS)
- **Action**: Single-page site with these sections:
  1. Sticky nav (frosted glass)
  2. Hero (full-screen, brand-forward)
  3. Ticker / marquee strip
  4. About / brand story
  5. Menu (grid / bento / collage — varies per brand)
  6. "Why us" / differentiators
  7. Order CTA (GrabFood + FoodPanda buttons)
  8. Location + hours
  9. Footer with social links
- **Mandatory CSS craft** (see `library/theme/restaurant-website-css.md`):
  - Grain texture on `body::before`
  - `clamp()` for fluid typography
  - Brand-matched color tokens as CSS vars
  - Ghost/outline text for background depth
  - Hover glow effects on cards
- **Rule**: Each restaurant must look visually distinct — different layout, different color scheme, different hero approach
- **Output**: `index.html` deployed at `[slug].bayuratech.com`

## Step 5: Deploy
- Deploy to subdomain: `[restaurantslug].bayuratech.com`
- Test on mobile — all sections must be responsive

## Step 6: Save Research
- Save all data to `notes.md` in restaurant folder
- Save owner contacts + outreach tracker to `contacts.md`

## Step 7: Pitch (Multi-channel)

### Pitch Message Formula (Malay, casual)
```
Assalamualaikum!

Saya web developer, and saya ada build demo website untuk [Restaurant] 
sebab rasa brand korang nampak menarik dan deserve lagi ramai orang 
nampak dekat platform online:

👉 [demo URL]

Tak charge apa-apa untuk tengok — ni just nak tunjuk macam mana 
[Restaurant] patut nampak online. Kalau berminat nak customize ikut 
menu penuh, tambah order system, atau apa-apa — boleh kita berbincang.

Portfolio saya boleh tengok dekat sini:
🌐 https://www.bayuratech.com/

Harap dapat membantu, apa-apa boleh contact saya semula ya!
```

### Channel Sequence
| Day | Channel | Target |
|-----|---------|--------|
| Day 1 | Instagram DM | Business account @handle |
| Day 2 | TikTok DM | Owner/content person handle |
| Day 3 | WhatsApp | Business number |
| Day 5 | Email | Business email (formal version) |
| Day 7 | Facebook Messenger | Owner personal FB (if found) |
| No reply | Walk in | Physical location |

### Email Version (more formal)
- Subject: `Demo Website Percuma untuk [Restaurant] — Saya Dah Siapkan`
- Address: `Assalamualaikum [Restaurant],`
- Add bullet points of what a full website includes
- Sign off with full name + Bayuratech + portfolio URL

---

## Decision Points

| Condition | Path |
|-----------|------|
| Can't find owner name | Address to business email generically |
| Instagram requires login | Use post URLs or skip IG DM |
| FoodPanda 403 | Use Playwright browser instead of WebFetch |
| Playwright browser drops | Full VS Code restart (not just taskkill) |
| Restaurant has no email | WhatsApp first, email secondary |

---

## Error Handling
- **Playwright browser crashes between sessions**: Full VS Code restart is the only reliable fix
- **Facebook login dialog blocks clicks**: Navigate directly to `facebook.com/[page]/about` or `?sk=about`
- **Can't find owner**: Use business contact, note in `contacts.md` as "Owner: Unknown"

---

## File Structure Per Restaurant
```
[restaurantname]/
├── index.html       ← demo website
├── notes.md         ← all research + UI/UX direction
└── contacts.md      ← owner contacts + outreach tracker
```

---

## Completed Restaurants (2026)
| Restaurant | Theme | Demo | Status |
|------------|-------|------|--------|
| Smoky Stacks | Dark ember (#FF5E00) | smokyweb.bayuratech.com | Pitched |
| BGM Burgerman | Red+Yellow (#D62B2B) | bgmburgerman.bayuratech.com | Pitched |
| Chapo's Burger | Dark+Neon (#BEFF00) | chapos.bayuratech.com | Pitching |

---

## Projects Using This
- Bayuratech Restaurant Pitch Campaign 2026
- Applicable to: any local F&B, retail, clinic, or small business with active social but no website

---
*Documented: 2026-05-03*
