# Nolan's Insight Log — 12/03/2026
## "Một ngày thay đổi cách nghĩ về lead scoring"

---

## Bắt đầu ngày với gì?

Sáng nay bắt đầu với 1 con bot scrape Instagram qua Apify, chấm điểm dựa trên followers + engagement, gửi kết quả qua Telegram. Nghe thì hay, chạy thì ra rác.

---

## Những ngõ cụt đã đâm vào

### Ngõ cụt 1: Followers cao = lead tốt?

Chạy lần đầu: 7 stores "qualified". Mở ra review tay thì thấy `wardrobebysw.shop` — tên store là "My Store", country Pakistan, không có sản phẩm, không revenue, không email. Nhưng 77K followers trên IG.

Vấn đề: cái username IG `@the.wardrobe.shop` thậm chí không match với domain `wardrobebysw.shop`. Khả năng cao Apify scrape nhầm profile.

**Bài học: IG followers là vanity metric. Mua 77K followers tốn $20. Không phải signal để đánh giá merchant có đang invest hay không.**

### Ngõ cụt 2: Data từ StoreLead không sạch như tưởng

Giả định ban đầu: StoreLead trả về stores đang hoạt động, có data đầy đủ. Thực tế: stores chết, tên mặc định, từ countries mình đã muốn exclude, thiếu gần hết thông tin.

Nếu không filter trước mà cứ scrape hết → tốn Apify credit vào stores rác, output ra danh sách mà qualify rate gần 0.

**Bài học: Garbage in, garbage out. Filter TRƯỚC khi enrich, không phải sau.**

### Ngõ cụt 3: Apify Free hết quota nhanh hơn tưởng

Free plan $5/tháng. Chạy 3 lần test, mỗi lần 50 stores → hết. Chưa kịp test scoring model đã hết credit.

Đây là lúc nhận ra: nếu signal chính (IG) phụ thuộc vào 1 nguồn trả phí mà quota thấp, thì pipeline không sustainable.

**Bài học: Đừng build pipeline phụ thuộc vào nguồn data mà mình không kiểm soát được quota.**

### Ngõ cụt 4: Ghép nhiều platform lại thì gãy ở đâu?

Thảo luận về việc dùng Social Blade + Apify + StoreLead + Google Shopping + Meta Ads. Nghe thì đa chiều, nhưng vấn đề lớn nhất: **làm sao biết "Cool Store LLC" trên Google Shopping là cùng 1 merchant với `coolstore.com` trên StoreLead?**

Identity resolution — mỗi platform gọi merchant 1 tên khác nhau. Không có ID chung. Fuzzy matching thì ~15% sai với brand name generic.

**Bài học: Thêm nguồn data ≠ thêm chất lượng. Nếu không match được thì chỉ thêm noise.**

---

## Điểm ngoặt trong ngày

### Câu hỏi thay đổi mọi thứ:

> "Em không muốn dùng social signal chỉ để bổ trợ cho phần followers mà em muốn dùng nó để xem như tín hiệu merchant đang đầu tư vào kênh này"

Đây là lúc nghĩ lại: cái mình THẬT SỰ cần đo không phải "store này có bao nhiêu followers" mà là **"store này có đang CHI TIỀN để grow hay không?"**

IG followers = có thể fake. Nhưng chạy 50 ads trên Meta = đang đốt tiền thật. Không ai đốt tiền quảng cáo cho 1 store không nghiêm túc.

→ Pivot từ IG-centric sang Ads-centric.

---

## Đánh giá các kịch bản cost

### Kịch bản A: Stack cũ (StoreLead + Apify IG Scraper)

| | |
|---|---|
| Chi phí | ~$50/tháng (StoreLead $49 + Apify $0.75) |
| Signal chính | IG followers + engagement |
| Ưu | Có data IG cụ thể (từng post, likes, comments) |
| Nhược | Fake followers, quota thấp, hết credit nhanh |
| Chất lượng output | THẤP — nhiều store rác, IG không match domain |
| Verdict | ❌ Đắt mà output kém |

### Kịch bản B: Stack mới (StoreLead + Meta Ad Library)

| | |
|---|---|
| Chi phí | ~$15/tháng (chỉ StoreLead, Meta Ads free) |
| Signal chính | Ads volume + ad pixels + tech stack |
| Ưu | Free signal, chứng minh merchant đang chi tiền thật |
| Nhược | Không có IG detail, Meta Ads Library có thể trả kết quả không match |
| Chất lượng output | CAO — ads = money = serious merchant |
| Verdict | ✅ Rẻ hơn 70%, signal đáng tin hơn |

### Kịch bản C: Full stack (StoreLead + Meta Ads + SerpAPI Google Shopping)

| | |
|---|---|
| Chi phí | $15-65/tháng (SerpAPI free hoặc $50 paid) |
| Signal | Ads + Google Shopping presence + tech stack |
| Ưu | Đa chiều nhất, biết merchant có bán trên Google Shopping không |
| Nhược | Identity resolution vẫn là vấn đề, SerpAPI free chỉ 100 calls/tháng |
| Verdict | ⚠️ Phase 2 — khi Phase 1 ổn |

### Kịch bản D: Full stack + Social Blade (IG historical)

| | |
|---|---|
| Chi phí | $16-101/tháng |
| Signal | Ads + Google Shopping + IG historical growth |
| Ưu | Biết IG đang grow hay chết, detect follower spikes bất thường |
| Nhược | Thêm complexity, thêm matching problem |
| Verdict | ⚠️ Phase 3 — khi cần validate IG data |

### So sánh nhanh

| Kịch bản | Cost/tháng | Stores/tháng | Signal quality | Recommend |
|---|---|---|---|---|
| A (IG) | $50 | 1,500 | ⭐⭐ | ❌ Bỏ |
| B (Ads) | $15 | 1,500 | ⭐⭐⭐⭐ | ✅ Bắt đầu từ đây |
| C (+Google) | $15-65 | 1,500 | ⭐⭐⭐⭐⭐ | Phase 2 |
| D (+SB) | $16-101 | 1,500 | ⭐⭐⭐⭐⭐ | Phase 3 |
| Clay.com | $149-349 | ~200 | ⭐⭐⭐⭐⭐ | Quá đắt cho team nhỏ |
| Intern tay | $300-500 | ~100 | ⭐⭐⭐ | Chậm + không scale |

**Kết luận: Kịch bản B ($15/tháng) cho 80% giá trị của kịch bản D ($101/tháng). Start here.**

---

## Cái chưa có câu trả lời

1. **StoreLead API có đang down không?** DNS resolve timeout cả buổi chiều. Chưa biết là ISP, DNS, hay StoreLead có vấn đề.

2. **Meta Ad Library match rate thực tế là bao nhiêu?** Search brand name "coolstore" trên Meta Ads, bao nhiêu % trả đúng store mình cần? Chưa test real data.

3. **Qualify rate thực tế?** Trong 10-15 scored leads, bao nhiêu cái Sales thật sự outreach được? Cần chạy 2-3 tuần mới biết.

4. **Landing page conversion rate?** Page đẹp nhưng chưa có traffic. Cần chạy ads hoặc SEO để test.

5. **Segment Bot có giúp workflow nhanh hơn không?** Hay cuối cùng vẫn export CSV rồi filter trên Google Sheets?

---

## Tóm lại ngày hôm nay

Bắt đầu ngày nghĩ rằng scrape IG sẽ cho mình biết store nào đáng approach. Kết thúc ngày nhận ra: **người ta có thể mua followers, nhưng không ai đốt tiền ads cho store không nghiêm túc.**

Signal tốt nhất không phải cái dễ scrape nhất. Signal tốt nhất là cái chứng minh merchant đang đặt tiền lên bàn.

Ngày mai: test pipeline mới với data thật, xem lý thuyết có đúng thực tế không.

---

*Nolan Nguyễn — 12/03/2026*
