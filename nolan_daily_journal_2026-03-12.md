# 📝 Nolan's Daily Journal — March 12, 2026

## 🏷️ Theme: "From IG Scraper to Ads Intelligence Platform"

---

## 📊 What Got Done Today

### 1. StoreLead Bot v2.1 — Quality Filters + Fake Detection
**Time:** ~10:15-10:47
**Status:** ✅ Code complete, file sent

- Added quality filters that run BEFORE Apify scrape (saves credits)
- Added fake IG follower detection (follow-for-follow ratio, low engagement check)
- Rebalanced scoring from IG-heavy (100% IG) to multi-signal:
  - Store Quality: 30pts | IG Signals: 30pts | Tech Stack: 20pts | Validation: 20pts
- Country whitelist: North America + Asia + EU (excluded India, Middle East, Africa)
- Store age ≥ 2 years, min 10 products, custom domain required
- Spam domain keyword filter

**Files:** `storelead_bot_v2.py` (updated), `config_v2.py` (updated)

---

### 2. StoreLead Bot v3 — Pivot to Ads Intelligence
**Time:** ~11:01-11:30
**Status:** ✅ Code complete, file sent

**The Big Pivot:** Dropped IG as primary signal → replaced with Meta Ad Library (FREE).

- New stack: StoreLead + Meta Ad Library + StoreLead tech stack
- Ads burn scoring: 50+ ads = HEAVY SPENDER, 20-49 = MODERATE, etc.
- Score: Store Quality 30 + Ads Investment 35 + Tech Stack 20 + Validation 15
- Cost: ~$15/month (vs $50/month with Apify)
- No Apify dependency

**Files:** `storelead_bot_v3.py`, `config_v3.py`

---

### 3. Segment Bot — Interactive Telegram Query Tool
**Time:** ~13:37-15:11
**Status:** ✅ Code complete, running on local machine

- Built interactive Telegram bot: type commands → get segments instantly
- Preset commands: `/whales`, `/growers`, `/switch`, `/fresh`, `/compare`
- Custom queries: `/segment --country US --plan plus --revenue 50000`
- CSV export: `/export` after any segment
- Successfully running on Nolan's machine

**Files:** `segment_bot.py`

**Setup issues resolved:**
- `.env` encoding (PowerShell UTF-8 BOM) → fixed with Python one-liner
- 403 Forbidden on Telegram → bot not added to group
- StoreLead DNS timeout → ISP/network issue (under investigation)

---

### 4. Lead Magnet Landing Page
**Time:** ~15:47-16:01
**Status:** ✅ v2 complete, Joy.so guideline applied

- Built single HTML landing page for lead magnet (loyalty playbook)
- v1: Generic design
- v2: Rebuilt with Joy.so CSS guideline:
  - Font: Alexandria (headings) + Cabin (body)
  - Primary: #06038D (Joy indigo)
  - Accent: #00D2D3 (teal)
  - Background: #F9FDFE
  - Section labels: uppercase, letter-spacing 2px
- Sections: Hero + Social Proof + What's Inside + Preview + Testimonials + CTA
- Form hooked to Formspree (need to register)
- Deployable on Vercel free tier

**Files:** `leadmagnet-demo/index.html`

---

### 5. Enrichment Journal / Transcript
**Time:** ~11:30
**Status:** ✅ Complete

- Full transcript of all enrichment discussions, alternatives research, cost analysis
- Covers: data quality issues, fake followers, identity resolution, provider comparison

**Files:** `nolan_enrichment_journal_2026-03-12.md`

---

## 🔍 Key Discoveries

### Discovery 1: IG Followers ≠ Investment Signal
**What:** High follower count doesn't mean merchant is investing in growth. Many stores have IG but abandoned it. Fake followers inflated scores.
**Evidence:** `wardrobebysw.shop` had 77K followers but IG username didn't match domain — likely wrong account entirely.
**Impact:** Pivoted entire scoring model from IG-centric to ads-centric.

### Discovery 2: Meta Ad Library is FREE and More Accurate
**What:** Meta Ad Library API is 100% free, unlimited, and shows REAL money being spent on ads.
**Why better:** A store running 50+ Meta ads is provably spending money on growth. Unlike followers which can be bought for $10.
**Impact:** Replaced Apify ($50/mo) with Meta Ads ($0/mo) as primary signal.

### Discovery 3: StoreLead Data Quality Issues
**What:** StoreLead returns stores that are dead, default-named ("My Store"), from excluded countries, or missing most data fields.
**Evidence:** PK (Pakistan) stores passing through, "My Store" with no products/revenue, IG usernames not matching domains.
**Impact:** Built quality filter layer that runs BEFORE any enrichment API calls.

### Discovery 4: Identity Resolution is the Hardest Problem
**What:** Matching the same merchant across StoreLead, Google Shopping, Meta Ads, and Social Blade is non-trivial.
**Why hard:** `coolstore.com` on StoreLead might be "Cool Store LLC" on Google Shopping, "@thecoolstore" on IG, and "CoolStore" on Meta Ads.
**Approach:** 3-tier confidence (URL match = 1.0, brand name match = 0.8, fuzzy = skip). Don't try to match 100%.

### Discovery 5: "Scored Lead" ≠ "Qualified Lead"
**What:** Bot output is scored leads. Human must still qualify by hand. Bot reduces 50 → 10-15 for manual review.
**Lesson:** Don't overstate what automation does. Track qualify rate to tune scoring over time.

---

## ⚠️ Difficulties & Blockers

### 1. Apify Quota Exhausted
**Problem:** Free plan ($5/mo) ran out after 3 test runs.
**Impact:** Can't scrape IG anymore this month.
**Resolution:** Pivoted to Meta Ad Library (free) as primary signal. IG scrape becomes optional Phase 2.

### 2. Security — API Keys Leaked 3 Times
**Problem:** Nolan shared screenshots containing API keys in group chat. Three separate occasions.
**Keys exposed:** StoreLead API key, Telegram bot tokens (2 different tokens).
**Resolution:** Keys need to be rotated. Going forward: never screenshot terminal with keys visible.

### 3. StoreLead DNS Timeout
**Problem:** `api.storelead.com` DNS resolution timed out. Both on Nolan's machine and Cortana's.
**Status:** Unresolved. May be temporary StoreLead issue or ISP DNS problem.
**Workaround:** Test with different DNS (8.8.8.8, 1.1.1.1) or wait.

### 4. Windows Environment Issues
**Problem:** Multiple friction points running Python scripts on Windows:
- `.env` file saved as UTF-16 with BOM by PowerShell → `python-dotenv` crash
- Hidden files (`.env`) not visible in File Explorer
- PowerShell command syntax different from bash
**Resolution:** Used Python one-liner to create `.env` with correct encoding.

### 5. Data Quality from StoreLead
**Problem:** Raw StoreLead data includes dead stores, spam, wrong countries, missing fields.
**Impact:** Low qualify rate on first run (7 scored, but many were junk).
**Resolution:** Quality filter layer added (age, country, plan, products, revenue, domain spam check).

---

## 📈 Progress Tracker

| Project | Yesterday | Today | Tomorrow |
|---|---|---|---|
| Social Signal Bot v1 | Script done, 7 qualified | Added quality filters, fake detection | — |
| Social Signal Bot v2 (IG) | — | v2.1 complete | Deprecated by v3 |
| Social Signal Bot v3 (Ads) | — | ✅ Complete | Test with real Meta Ads data |
| Segment Bot | — | ✅ Complete + running | Query real segments, export CSV |
| Landing Page | — | ✅ v2 (Joy guideline) | Nolan to hijack competitor layout |
| Enrichment Journal | — | ✅ Complete | — |

---

## 💰 Cost Summary

| Stack | Cost/month | Status |
|---|---|---|
| v1: StoreLead + Apify IG | $50 | ❌ Apify quota exhausted |
| v3: StoreLead + Meta Ads | $15 | ✅ Active |
| Segment Bot | $0 (uses same StoreLead) | ✅ Active |
| Landing Page (Vercel) | $0 | 🔜 Ready to deploy |

---

## 🎯 Tomorrow's Plan

1. [ ] **Reset all leaked API keys** (StoreLead + Telegram bot tokens) — URGENT
2. [ ] **Test segment bot** with real queries: `/whales`, `/fresh`, `/switch`
3. [ ] **Get Facebook App Token** for Meta Ad Library (free, developers.facebook.com)
4. [ ] **Run v3 pipeline** with Meta Ads data on 50 stores
5. [ ] **Qualify scored leads by hand** — track accept/reject ratio
6. [ ] **Research competitor layouts** (Rivo, Smile.io) for landing page "hijack"
7. [ ] **Fix StoreLead DNS** — try different DNS or VPN

---

## 💡 Lessons Learned

1. **Start with free signals before paid ones.** StoreLead tech stack (free, already included) + Meta Ad Library (free API) gives more signal than Apify IG ($50/mo).
2. **Filter before enrich.** Running quality filters BEFORE API calls saves both money and time.
3. **Don't trust follower counts.** IG followers are easily gamed. Ads spend is not.
4. **Windows Python setup has friction.** Next time: consider WSL or Docker for cleaner environment.
5. **NEVER screenshot terminal with API keys.** Three leaks in one day is a pattern, not an accident.
6. **Bot output ≠ qualified leads.** Be honest about what automation can and can't do.

---

*Generated by Cortana — March 12, 2026, 4:01 PM*
