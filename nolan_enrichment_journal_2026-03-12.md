# 📝 Enrichment & Segmentation Journal
## Date: 2026-03-12 | Author: Nolan Nguyễn + Cortana

---

## 1. Xuất phát điểm

**Mục tiêu ban đầu:** Build "mini Clay" bot scrape IG data → score leads → alert Telegram.

**Stack ban đầu:** StoreLead (store discovery) + Apify `apify/instagram-scraper` (IG data) + Brave Search (fallback).

**Scoring v1:** 0-100, 4 chiều × 25 điểm: IG followers + engagement + activity + revenue.

---

## 2. Vấn đề phát hiện khi chạy thực tế

### 2.1 Data rác từ StoreLead
- **Store lạ, unfound:** Store tồn tại trên StoreLead nhưng thực tế đã chết hoặc không hoạt động
- **Ví dụ cụ thể:** `wardrobebysw.shop` — tên "My Store" (default Shopify), country PK (Pakistan), plan/products/revenue đều trống, nhưng IG 77K followers
- **Gốc rễ:** Không có quality filter, mọi store từ StoreLead đều được đưa vào pipeline

### 2.2 IG data quality không đáng tin
- **Fake followers phổ biến:** 77K followers nhưng IG username `@the.wardrobe.shop` không khớp domain `wardrobebysw.shop` → khả năng cao IG thuộc store khác
- **IG quá đại trà:** Nhiều store có IG nhưng bỏ hoang, followers không = đang invest
- **Không có historical data:** Chỉ có snapshot tại thời điểm scrape, không biết đang grow hay chết

### 2.3 Apify quota hết
- Free plan $5/month, hết quota sau vài lần chạy test
- Cần giải pháp thay thế hoặc giảm phụ thuộc Apify

### 2.4 3 luồng data gộp lại không clean
- Nhiều store rác, địa chỉ outdate
- Một số region nhiều store scam (Pakistan, Bangladesh, Nigeria...)
- Segment quá rộng → qualify rate thấp

---

## 3. Các giải pháp đã thảo luận

### 3.1 Quality Filter (IMPLEMENTED trong v2.1 và v3)
**Quyết định:** Filter TRƯỚC khi enrich để tiết kiệm API calls.

| Filter | Rule | Lý do |
|---|---|---|
| Store age | ≥ 2 năm | Loại stores mới tạo thử |
| Products | ≥ 10 | Store có hàng thật |
| Custom domain | Required | Dấu hiệu chuyên nghiệp |
| Revenue | ≥ $1K/mo | Đang kinh doanh thật |
| Country | Whitelist (NA + Asia + EU) | Loại scam regions |
| Plan | Exclude trial/development | Đang trả tiền |
| Store name | Exclude "My Store", "Test Store"... | Chưa setup |
| Domain spam | Exclude "cheap", "replica"... | Spam/dropship |
| Missing data | ≥ 3/4 fields trống → loại | StoreLead không có đủ info |

### 3.2 Fake IG Detection (IMPLEMENTED trong v2.1)
- Follow-for-follow ratio check (following > 80% × followers = suspicious)
- Low engagement + high followers = suspicious (< 0.3% engagement + > 5K followers)
- Zero posts = suspicious
- Suspicious profiles bị cap IG score tối đa 10/30

### 3.3 IG Username vs Domain Mismatch (DISCUSSED, partially implemented)
- Dùng `SequenceMatcher` fuzzy matching
- `wardrobebysw` vs `the.wardrobe.shop` = ~0.5 → LOW confidence
- Nếu confidence < 0.7 → flag hoặc giảm IG score
- **Khó khăn:** Nhiều store có IG username hoàn toàn khác domain (rebranded, creative names)

### 3.4 Chuyển từ IG-centric sang Ads-centric (IMPLEMENTED trong v3)

**Insight quan trọng nhất của session:**
> IG followers ≠ đang invest. Merchant chạy 50+ Meta ads = đang đốt tiền thật = signal mạnh hơn nhiều.

**Scoring v3 (rebalanced):**

| Dimension | Weight | Source | API Cost |
|---|---|---|---|
| Store Quality | 30/100 | StoreLead | Included |
| Ads Investment | 35/100 | Meta Ad Library + StoreLead tech | FREE |
| Tech Stack | 20/100 | StoreLead technologies | Included |
| Validation | 15/100 | StoreLead contact_info | Included |

---

## 4. Alternatives đã research

### 4.1 Apify Alternatives

| Tool | Free tier | Ưu | Nhược | Khả thi |
|---|---|---|---|---|
| Bright Data | Trial $5 | Proxy mạnh nhất | Đắt, UI phức tạp | ⚠️ Scale phase |
| Crawlee (by Apify) | Free, open-source | Tự host, $0 | Setup server + proxy | ✅ Nếu có server |
| Phantombuster | 2h/ngày | Có IG scraper, no-code | Free tier hạn chế | ❌ Quá ít quota |
| Instaloader | Free | Python, local | Dễ bị IG block | ⚠️ Cần burner IG account |

### 4.2 IG-specific Data

| Tool | Ưu | Nhược | Khả thi |
|---|---|---|---|
| Social Blade | Historical data có sẵn! | API $4.99/mo, free chỉ web | ✅ Apify actor `epctex/socialblade-scraper` ~$0.50/1K |
| HypeAuditor | Detect fake followers built-in | $299+/mo | ❌ Quá đắt |
| Phyllo | Official APIs | Cần account owner authorize | ❌ Không áp dụng cho outbound |

### 4.3 Ads/Commerce Signals

| Tool | Data | Free? | Khả thi |
|---|---|---|---|
| **Meta Ad Library** | FB/IG ads active, volume, creative | ✅ 100% free | ✅✅ Best option |
| Google Ads Transparency | Google ads active | ✅ Free | ✅ Manual hoặc scrape |
| SerpAPI Google Shopping | Products on Google Shopping | 100 free/month | ✅ Phase 2 |
| TikTok Creative Center | TT ads | ✅ Free browse | ⚠️ Manual only |
| SimilarWeb | Traffic sources, % paid | 5 results/ngày | ⚠️ Quá ít |
| SpyFu | Google Ads spend estimate | Limited free | ⚠️ Limited |

### 4.4 All-in-one (thay pipeline)

| Tool | Cost | Verdict |
|---|---|---|
| Clay.com | $149-349/mo | Đắt, nhưng feature rich |
| Apollo.io | $49-99/mo | IG data hạn chế |
| Databar.ai | Free tier | Mới, ecosystem nhỏ |

---

## 5. Khó khăn chính: Identity Resolution

**Vấn đề:** Mỗi platform biết merchant bằng identifier khác nhau.

| Platform | Identifier | Ví dụ |
|---|---|---|
| StoreLead | domain | `coolstore.com` |
| Google Shopping | merchant name | `Cool Store LLC` |
| Meta Ads | page name | `CoolStore` |
| Social Blade/IG | username | `@thecoolstore` |

**Các case sẽ gãy:**

| Case | Tỷ lệ ước tính | Giải pháp |
|---|---|---|
| Brand name quá generic ("The Shop") | ~15% | Skip, chỉ match nếu URL chứa domain |
| Multi-brand company | ~5% | Chấp nhận miss |
| Rebranded (domain cũ ≠ tên mới) | ~5% | StoreLead thường update |
| Same name different country | ~10% | Filter theo country đã set |

**Approach:** 3 tầng confidence
- HIGH (1.0): URL/domain exact match
- MEDIUM (0.7-0.9): Brand name fuzzy match
- LOW (< 0.7): Bỏ qua, không trust

**Kết luận:** Đừng cố match 100%. Filter trước, enrich sau. Chỉ deep enrich cho stores đã qua quality filter.

---

## 6. Cost Analysis

### Phase 1 (Current — v3)
| Item | Cost/tháng |
|---|---|
| StoreLead | $49 (hoặc ~$15 nếu dùng ít) |
| Meta Ad Library | $0 |
| Telegram | $0 |
| **Total** | **$15-49** |

### Phase 2 (thêm Google Shopping)
| Item | Cost/tháng |
|---|---|
| Phase 1 | $15-49 |
| SerpAPI free | $0 (100/tháng) |
| SerpAPI paid | $50 (nếu scale) |
| **Total** | **$15-99** |

### Phase 3 (thêm IG validate)
| Item | Cost/tháng |
|---|---|
| Phase 2 | $15-99 |
| Apify Social Blade | ~$0.75 |
| **Total** | **$16-100** |

### So sánh
| Stack | Cost/tháng | Leads/tháng | Quality |
|---|---|---|---|
| Nolan's bot Phase 1 | $15-49 | ~200 scored | Ads-based (high) |
| Clay.com | $149-349 | ~200 | Multi-signal (high) |
| Apollo.io | $49-99 | ~200 | Email-focused (medium) |
| Intern research | $300-500 | ~100 | Human judgment (highest) |

---

## 7. Key Insight: "Scored Lead" ≠ "Qualified Lead"

**Correction từ Nolan:** Bot output = scored leads, không phải qualified leads. Sales rep vẫn phải qualify tay.

**Pipeline thực tế:**
```
Raw stores (50/run) → Filtered (30-35) → Scored (10-15) → Qualified bằng tay (5-8) → Outreach
```

Bot giúp thu hẹp 50 → 10-15, tiết kiệm thời gian. Không thay thế human judgment.

**Metrics cần track:**
- Qualify rate = bao nhiêu scored leads được approve bằng tay
- Reasons reject = tại sao loại → feedback loop tune scoring
- Cost per qualified lead = tính SAU khi có data thật

---

## 8. Decisions Made

| # | Decision | Rationale |
|---|---|---|
| 1 | Chuyển từ IG-centric → Ads-centric | Ads = tiền thật, IG followers có thể fake |
| 2 | Quality filter chạy TRƯỚC enrich | Tiết kiệm API calls + credit |
| 3 | Country whitelist (NA + Asia + EU) | Loại scam regions |
| 4 | Store age ≥ 2 năm | Loại stores thử nghiệm |
| 5 | Meta Ad Library làm signal chính | Free + chính xác (ads = spending money) |
| 6 | Giữ IG nhưng giảm weight (nếu có data) | IG là 1 signal, không phải signal chính |
| 7 | Multi-signal scoring (store + ads + tech + validation) | Không phụ thuộc 1 nguồn |
| 8 | Provider pattern cho future sources | Thêm nguồn mới = thêm 1 class, không sửa logic |
| 9 | MAX_STORES = 50 | Tăng từ 30, đủ rộng nhưng manageable |
| 10 | Social Blade qua Apify (Phase 2) | $0.50/1K results, có historical data |

---

## 9. Open Questions

1. **StoreLead data freshness:** Bao nhiêu % stores trên StoreLead đã chết/outdate? Cần verify store alive?
2. **Meta Ad Library accuracy:** Search by brand name có miss nhiều không? False positive rate?
3. **Optimal SCORE_THRESHOLD:** 40 có phù hợp với scoring mới? Cần chạy thực tế rồi tune
4. **Social Blade Phase 2 timing:** Khi nào cần historical IG data? Sau khi ads-based scoring ổn?
5. **Instaloader as free IG backup:** Có đáng setup không? Risk bị IG ban?

---

## 10. Next Steps

- [ ] Nolan: Reset Telegram bot token + StoreLead API key (leaked cũ)
- [ ] Nolan: Tạo Facebook App → lấy Access Token (free) cho Meta Ad Library
- [ ] Nolan: Download `storelead_bot_v3.py` + `config_v3.py`, fill API keys
- [ ] Nolan: Chạy v3 trên 50 stores, review output
- [ ] Nolan: Track qualify rate (bao nhiêu scored leads approve bằng tay)
- [ ] Nolan: Feedback rejected leads → Cortana tune scoring
- [ ] Phase 2: Thêm SerpAPI Google Shopping (khi Phase 1 ổn)
- [ ] Phase 3: Thêm Social Blade via Apify (khi cần IG validate)

---

## 11. Files Created/Updated Today

| File | Version | Description |
|---|---|---|
| `storelead_bot_v2.py` | v2.1 | Added quality filters + fake IG detection + multi-signal scoring |
| `config_v2.py` | v2.1 | Added quality filter configs + country whitelist |
| `storelead_bot_v3.py` | v3.0 | NEW: Ads-centric, StoreLead + Meta Ad Library, no Apify |
| `config_v3.py` | v3.0 | NEW: Config for v3 stack |
| `social_signal.py` | v1.0 | Original standalone pipeline (unchanged) |

---

*Generated by Cortana — 2026-03-12*
