# SPEC — AI Product Hackathon

**Nhóm:** Group_3_E402

**Track:**  Vinmec 

---

## Problem statement
- Lễ tân và tổng đài hiện phải thủ công hỏi triệu chứng, tra cứu và tư vấn chuyên khoa cho từng bệnh nhân — mỗi ca mất 5–10 phút, dễ gây hàng chờ kéo dài khi lượng người tăng cao, ảnh hưởng đến trải nghiệm bệnh nhân và tạo áp lực lớn cho nhân viên tiếp nhận. Hệ thống AI được đề xuất để đảm nhận bước sàng lọc ban đầu — hỏi triệu chứng, gợi ý chuyên khoa phù hợp và đề xuất xếp phòng khám — giúp rút ngắn thời gian xử lý và tăng độ chính xác trong điều hướng, trong khi lễ tân vẫn là người xác nhận và quyết định cuối cùng.

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi** | User nào? Pain gì? AI giải gì? | Khi AI sai thì sao? User sửa bằng cách nào? | Cost/latency bao nhiêu? Risk chính? |
| **Trả lời** | Lễ tân | Người dùng và lễ tân đau. Thời gian phải chờ lễ tân chọn chuyên khoa, xếp phòng phù hợp từ mô tả triệu chứng của người dùng. AI từ danh sách các triệu chứng của bệnh nhân -> Đưa ra lựa chọn chuyên khoa và gợi ý xếp phòng phù hợp. | Chi phí API call, latency <3s. | Risk: Triệu chứng mô tả mơ hồ, nhiều khoa overlap.|

**Automation hay augmentation?** ☐ Automation · ☑ Augmentation
Justify: *Augmentation — lễ tân luôn nhìn thấy gợi ý và xác nhận trước khi điều hướng bệnh nhân. AI sai thì lễ tân sửa trong 1 giây, không ảnh hưởng flow.*

**Learning signal:**

1. User correction đi vào đâu? -> Log cặp (chuỗi triệu chứng -> chuyên khoa lễ tân đã chọn thay thế) vào correction database. Dùng làm fine-tuning data hoặc few-shot examples cho prompt sau.
2. Product thu signal gì để biết tốt lên hay tệ đi? -> (a) Tỷ lệ lễ tân giữ nguyên gợi ý AI (acceptance rate); (b) Tỷ lệ bệnh nhân phải chuyển khoa sau khi đã vào khám (downstream error rate); (c) Thời gian xử lý trung bình mỗi lượt.
3. Data thuộc loại nào? ☐ User-specific · ☑ Domain-specific · ☐ Real-time · ☑ Human-judgment · ☐ Khác:
   Có marginal value không? **Có** — dữ liệu triệu chứng -> chuyên khoa theo ngữ cảnh Vinmec (tên khoa, quy trình nội bộ, phân loại bệnh nhân VIP/thường) là domain-specific, model nền chưa biết. Mỗi correction của lễ tân là human-judgment label có giá trị cao.

---

## 2. User Stories — 4 paths

Mỗi feature chính = 1 bảng. AI trả lời xong → chuyện gì xảy ra?

### Feature: *tên feature*

**Trigger:** *VD: User nhận email mới → AI phân loại → ...*

| Path | Câu hỏi thiết kế | Mô tả |
|------|-------------------|-------|
| Happy — AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | *Email tự gắn nhãn "Urgent", user thấy đúng, tiếp tục làm việc* |
| Low-confidence — AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | *Hiện 2 nhãn gợi ý + confidence %, user chọn 1* |
| Failure — AI sai | User biết AI sai bằng cách nào? Recover ra sao? | *Email quan trọng bị gắn "FYI" → user thấy khi review → sửa nhãn* |
| Correction — user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | *Kéo thả sang nhãn đúng → correction log → cải thiện model* |

*Lặp lại cho feature thứ 2-3 nếu có.*

---

## 3. Eval metrics + threshold

**Optimize precision hay recall?** ☐ Precision · ☐ Recall
Tại sao? ___
Nếu sai ngược lại thì chuyện gì xảy ra? *VD: Nếu chọn precision nhưng low recall → user không tìm thấy kết quả cần → bỏ dùng*

| Metric | Threshold | Red flag (dừng khi) |
|--------|-----------|---------------------|
| *VD: Accuracy phân loại đúng* | *≥85%* | *<70% trong 1 tuần* |
|   |   |   |
|   |   |   |

---

## 4. Top 3 failure modes

*Liệt kê cách product có thể fail — không phải list features.*
*"Failure mode nào user KHÔNG BIẾT bị sai? Đó là cái nguy hiểm nhất."*

| # | Trigger | Hậu quả | Mitigation |
|---|---------|---------|------------|
| 1 | *VD: Email chứa thuật ngữ ngoài domain* | *AI gắn nhãn sai, tự tin cao* | *Detect low-confidence → hỏi user xác nhận* |
| 2 |   |   |   |
| 3 |   |   |   |

---

## 5. ROI 3 kịch bản

|   | Conservative | Realistic | Optimistic |
|---|-------------|-----------|------------|
| **Assumption** | *100 user/ngày, 60% hài lòng* | *500 user/ngày, 80% hài lòng* | *2000 user/ngày, 90% hài lòng* |
| **Cost** | *$50/ngày inference* | *$200/ngày* | *$500/ngày* |
| **Benefit** | *Giảm 2h support/ngày* | *Giảm 8h/ngày* | *Giảm 20h, tăng retention 5%* |
| **Net** |   |   |   |

**Kill criteria:** *Khi nào nên dừng? VD: cost > benefit 2 tháng liên tục*

---

## 6. Mini AI spec (1 trang)

- Hiện tại, lễ tân và tổng đài phải tự hỏi triệu chứng, tra cứu và quyết định chuyên khoa cho từng bệnh nhân, mỗi ca mất khoảng 5–10 phút. Khi lượng bệnh nhân tăng, quy trình này nhanh chóng trở thành nút thắt cổ chai, gây ùn tắc, kéo dài thời gian chờ và tạo áp lực lớn lên nhân viên, đồng thời làm giảm trải nghiệm chung của bệnh nhân.

- Sản phẩm đề xuất giải quyết vấn đề này bằng cách đưa AI vào bước sàng lọc ban đầu. AI sẽ tự động hỏi các triệu chứng cơ bản, phân tích và gợi ý chuyên khoa phù hợp, thậm chí đề xuất phòng khám tương ứng. Tuy nhiên, hệ thống chỉ đóng vai trò hỗ trợ **augmentation**, không thay thế con người — lễ tân vẫn là người xác nhận và đưa ra quyết định cuối cùng.

- Về chất lượng, hệ thống được thiết kế ưu tiên **recall** cao để tránh bỏ sót các ca bệnh quan trọng, chấp nhận việc đôi khi đưa ra nhiều gợi ý khi không chắc chắn. Kèm theo đó là hiển thị mức độ confidence và cơ chế hỏi thêm hoặc cảnh báo rule-based trong các trường hợp có dấu hiệu nguy hiểm.

- Rủi ro chính nằm ở việc AI có thể gợi ý sai trong các ca mơ hồ hoặc nghiêm trọng, nên cần luôn giữ lễ tân trong vòng kiểm soát, đồng thời thiết kế các cơ chế fallback rõ ràng để giảm thiểu sai sót.

![alt text](image.png)
