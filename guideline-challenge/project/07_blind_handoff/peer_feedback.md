# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Group 1 — RoadElements
- **Người label blind:** Trần Thẩm Anh Toàn

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?** Bảng taxonomy mục 4: class quyết định luôn `lane_change`,
   chỉ cần đếm số nét (đơn/đôi) và xem kiểu nét (đứt/liền). Rule gộp màu trắng/vàng cũng giúp khỏi phân vân.
2. **Rule nào mơ hồ hoặc phải tự suy diễn?**
   - Vạch mép đường có trong scope không (mục 1/5 loại "curb", nhưng ví dụ BDD01 mục 9 lại label vạch vàng mép
     trái) — peer suy diễn theo ví dụ, label cả vạch mép ở BDD05/BDD08/BDD19.
   - "Làn liền kề" rộng tới đâu — có vẽ vạch ngoài của làn kề không.
   - Vạch đứt vẽ 1 polyline nối qua khoảng trống hay mỗi đoạn 1 polyline.
   - Chưa có rule khi nào dùng `lane_change = unknown`.
3. **Sample nào khiến guideline "vỡ"?** BDD24 (tuyết, chỉ thấy 1 đoạn, 3 rule áp dụng ra 3 kết quả khác nhau);
   BDD05 (vùng sọc chéo song song không phải hình chữ V — có tính `chevron_area` không, polygon đóng ở đâu khi
   vùng ra khỏi khung hình); BDD19 (khe nối bê tông trông giống vạch liền mờ).
4. **Attribute/default nào trong CVAT dễ gây thao tác sai?** `lane_change` là attribute chọn tay dù class đã quyết
   định sẵn giá trị — CVAT vẫn cho gán lệch nhau (`single_dashed` + `not_allowed`) mà không ai phát hiện.
   `needs_review` mặc định `false` nên quên tick trông như đã chắc chắn. `cvat_labels.json` không có label kiểu
   tag cho case "ảnh không có vạch" (`negative`), nên ảnh cố ý để trống và ảnh bị quên label trông giống hệt nhau.
5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?** Thêm bảng "vạch nào vẽ/không vẽ" kèm ảnh ví dụ cho 5 điểm
   mơ hồ ở câu 2, và thêm tag `negative` vào labels JSON.

*(Toàn văn feedback + 5 câu hỏi clarification: `../../../../Downloads/group1-blind-feedback.md` — đã copy 5 câu hỏi vào `clarification_log.csv`.)*

## 2. Owner phân loại

| Feedback / decision sai | Nguyên nhân | Xử lý | Bằng chứng |
|---|---|---|---|
| BDD19 d2 — thiếu `needs_review=true` trên vạch mòn mép phải | execution_error | coaching (nhắc rule mục 6: vạch mòn/bong tróc phải tick needs_review) | `transfer_score.csv` BDD19 d2 |
| BDD25 d1 — chỉ 1/3 đoạn vạch được tick `needs_review` dù cả đoạn cùng nằm trong điều kiện mưa/loá | guideline_gap | revise_rule — guideline v4 cần nói rõ: điều kiện ánh sáng/thời tiết áp dụng cho **cả đoạn vạch nhìn thấy**, không phải áp dụng lẻ tẻ theo từng object | `transfer_score.csv` BDD25 d1; `06_calibration_measure.csv` |
| BDD24 d1 — peer chọn `single_dashed/allowed` thay vì `unknown/unknown` khi chỉ thấy 1 đoạn, 2 đầu bị che/mờ | guideline_gap | add_escalation — thêm rule rõ: "chỉ thấy 1 đoạn, không xác định được cả 2 đầu → bắt buộc `lane_change=unknown`", không chỉ tick `needs_review` rồi tự đoán class | `transfer_score.csv` BDD24 d1; đúng case peer tự nêu ở câu 3 |
| Vạch mép đường (curb) — guideline mục 1/5 nói loại trừ nhưng ví dụ mục 9 lại label | guideline_gap | revise_rule — xoá mâu thuẫn giữa rule và ví dụ ở guideline v4 | Feedback câu 2, dòng 1 |
| Không có label `negative` cho ảnh trống trong `cvat_labels.json` | guideline_gap | add_escalation — thêm tag `negative` vào `03_cvat_labels.json` | Feedback câu 4, dòng 3 |

**Critical escape:** 1/2 (BDD24). Theo `RUBRIC.md`, vì đây là `guideline_gap` (guideline chưa ép `lane_change=unknown`
khi thiếu bằng chứng ở 2 đầu) chứ không có rule/escalation nào ngăn được — Blind Handoff bị áp **critical cap tối đa
10/20**. Guideline v4 phải sửa đúng gap này trước khi nộp.
