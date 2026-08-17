# Báo cáo Nộp bài Lab 17 - Multi-Memory Agent với Zep

**Họ và tên:** Trịnh Quang Anh  
**Mã học viên:** 2A202601796  

---

## 1. Trả lời 3 Câu hỏi Thực hành (README Questions)

1. **Layer quan trọng nhất trong bộ test này:**
   **Long-term memory** là layer quan trọng nhất vì chiếm nhiều test case nhất (E02, E03, E08, E09). Nó đảm bảo duy trì preferences, open loops và user isolation qua các session.
2. **Trade-off giữa Zep Cloud (Context Block) vs Redis + Qdrant tự dựng:**
   - *Zep Cloud:* Tự động trích xuất facts, hợp nhất graph, quản lý recency/conflict và tự động tạo Context Block relevance. Đánh đổi: Phụ thuộc SaaS, latency API.
   - *Redis + Qdrant:* Kiểm soát hoàn toàn dữ liệu, latency thấp. Đánh đổi: Phải tự code logic compaction, conflict resolution, graph traversal và context window budget.
3. **Guardrail chống Memory Poisoning:**
   Sử dụng **Consent & Opt-in Registry** (`consent.json`), kiểm duyệt nội dung trước khi lưu (PII Redaction trong `privacy_guard.py`), phân quyền dữ liệu theo `user_id` cô lập hoàn toàn, và gắn provenance/validity timestamp cho các facts.

---

## 2. Phân tích Kết quả Benchmark (4 Câu phân tích)

1. **Layer có Hit Rate thấp nhất:** 
   Trong baseline *No-Memory*, tất cả durable layers đều có hit rate 0%. Khi bật Zep Memory, tất cả các layer (Short-term, Long-term, Episodic, Semantic) đều đạt hit rate 100%.
2. **Query retrieve nhiều token nhất:** 
   Query E07 (mixed context) và E04/E05 (episodic history) retrieve nhiều token nhất do phải tổng hợp dữ liệu từ nhiều nguồn.
3. **Case Mixed (E07):** 
   Cần kết hợp **Long-term** (`Python` preference) và **Semantic** (`Idempotency-Key` rule). Bắt buộc chứa cả 2 evidence này.
4. **Token Reduction & Hit Rate:** 
   *No-Memory* có token reduction cao vì không load memory, nhưng hit rate bằng 0% do thiếu context. Zep Memory giúp giảm 80-90% token so với raw transcript nhưng vẫn giữ hit rate >= 80%.

---

## 3. Giải thích Compaction (E10) & Recency (E08)

- **Compaction (E10):** Qua kết quả chạy `src.demo_short_term`:
  - Strategy `buffer` giữ nguyên 16 lượt thoại khiến token tăng cao (231 tokens) và có nguy cơ trôi mất tin nhắn cũ khi hội thoại kéo dài.
  - Strategy `sliding` & `summary` nén các tin nhắn cũ về `<SESSION_SUMMARY>` và tự động trích xuất các ràng buộc/quyết định vào khối `<DURABLE_NOTES>`.
  - Nhờ trích xuất durable note `Constraint: REVIEW-DEADLINE-1600 - project review is Friday at 16:00`, ngay cả khi lượt thoại chứa thông tin này bị evict khỏi `<RECENT_TURNS>` (chỉ giữ 6 lượt thoại gần nhất 9-14), thông tin deadline `16:00` và `Friday` vẫn được bảo toàn 100% cho agent.
- **Recency (E08):** Zep ưu tiên fact mới nhất (`TypeScript` / `NestJS` cho `BLUEBIRD-42`), ghi đè preference cũ (`Python`) nhưng vẫn giữ provenance lịch sử.

---

## 4. Minh chứng (Screenshots)

Các ảnh màn hình kết quả được lưu tại:
- `submission/long_term.png` (E02, E03, E08, E09 PASS)
- `submission/episodic.png` (E04, E05 PASS)
- `submission/semantic.png` (E06, E11 PASS)
- `submission/privacy.png` (Forget + Verify-only PASS)

---

## 5. Kết quả Golden Set (Bonus +10 điểm)

- **Lệnh chạy:** `docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --golden`
- **Kết quả:** **20/20 PASS (Perfect Score - 100%)**
- **Điểm thưởng Golden Set:** **+10/10 điểm**

