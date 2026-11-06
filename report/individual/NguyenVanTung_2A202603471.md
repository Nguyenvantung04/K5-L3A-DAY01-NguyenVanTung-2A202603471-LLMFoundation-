# Báo cáo Lab Ngày 1 — Lọc tin nhắn rác (Spam / Ham)

- **MSSV**: 2A202603471
- **Họ và tên**: Nguyễn Văn Tùng
- **SEED**: 5407
- **Thử thách cá nhân**: Đề #2 (ML · Giảm vocab còn 100 từ và tăng lên 5000 từ)

---

## 1. Tóm tắt kết quả các phương pháp (4 thế hệ AI)

| Phương pháp | F1 (test) | F1 (SUB 40) | Độ trễ p95 (ms) | Nhận xét chính |
|---|---|---|---|---|
| **1. BoW + LogReg** | ~0.92 | ~0.90 | < 1 ms | Rất nhanh, nhẹ; bắt tốt spam theo từ khóa nhưng thất bại khi né từ khóa hoặc thiếu ngữ cảnh. |
| **2. BoW + MLP** | ~0.94 | ~0.92 | ~2 ms | Cải thiện nhẹ so với LogReg; tuy nhiên với dữ liệu BoW nhỏ và thưa, deep learning chưa tạo đột phá lớn. |
| **3b. Encoder + LogReg** | ~0.96 | ~0.95 | ~15–20 ms | Đạt chất lượng cao nhất trong nhóm đáp ứng SLA (<50 ms); hiểu tốt ngữ cảnh và câu văn diễn đạt tự nhiên. |
| **4. LLM (Zero-shot)** | - | ~0.95 | > 300 ms | Hiểu sâu ngữ cảnh, không cần tập train; tuy nhiên độ trễ cao và chi phí lớn, dễ dính prompt injection nếu không có thẻ phòng thủ. |

---

## 2. Kết quả Thử thách cá nhân (Đề #2)

- **Vocab = 100 từ**: $F_1$ sụt giảm rõ rệt (Recall thấp) do từ điển quá nhỏ, bỏ lọt nhiều từ vựng đặc trưng của spam (underfitting).
- **Vocab = 5000 từ**: $F_1$ đạt mức tối đa (~0.93–0.95), bao phủ tốt hơn; tuy nhiên mức cải thiện so với vocab 2000 từ là không nhiều do các từ bổ sung có tần suất rất thấp.

---

## 3. Đề xuất hệ thống thực tế (Production System)

1. **Phương án chọn**: Kết hợp mô hình lai (**Router**):
   - **Tầng 1 (LogReg / BoW)**: Xử lý ~90–95% lượng tin nhắn có độ tin cậy cao ($p < 0.2$ hoặc $p > 0.8$), đảm bảo thông lượng lớn và độ trễ $<5\text{ ms}$.
   - **Tầng 2 (MiniLM + LogReg hoặc LLM)**: Chỉ định tuyến ~5–10% tin nhắn ở vùng "lưỡng lự" ($0.2 \le p \le 0.8$) lên tầng sâu hơn để phân tích ngữ cảnh.
2. **Lợi ích**: Giữ cho $95\%$ yêu cầu đạt SLA $<50\text{ ms}$, tiết kiệm tối đa chi phí API mà vẫn đảm bảo độ chính xác tổng thể ở mức cao nhất.

