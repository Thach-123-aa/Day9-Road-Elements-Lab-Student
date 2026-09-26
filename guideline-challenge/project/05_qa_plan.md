# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** QA owner (Đinh Công Minh, hỗ trợ Phạm Nguyễn Tuân) review 100% bài blind của
  peer gửi về (`transfer_score.csv`); với calibration nội bộ, review toàn bộ 5/5 annotator trên cùng bộ ảnh
  calibration (không random, vì số người ít nên xem hết rẻ hơn lấy mẫu).
- **Chọn sample theo rule nào:** Blind — 100% (mọi quyết định trong `gold_decisions.csv` đều bị chấm, không sample).
  Calibration — 100% annotator, nhưng ưu tiên đọc kỹ trước các sample có tag `critical`/`conflict` trong
  `sample_pack.csv` (`BDD10`, `BDD16`) vì đây là nơi rủi ro cao nhất.
- **Issue được ghi ở đâu, đóng thế nào:** Bất đồng lúc calibration → `06_calibration_report.csv` (đóng bằng cách
  sửa guideline lên version tiếp theo + ghi dòng vào `08_revision_log.md`). Câu hỏi lúc peer vẽ blind →
  `clarification_log.csv` (đóng bằng cách xác nhận guideline đã trả lời được câu đó ở version nào, không trả lời
  miệng). Nhận xét sau khi chấm → `peer_feedback.md`.
- **Khi phát hiện guideline gap thì update và version ra sao:** Tăng số `Version` trong `02_guideline.md`
  (`v1 → v2 → v3`), thêm dòng vào `08_revision_log.md` ghi rõ đổi gì/vì sao/bằng chứng (sample_id cụ thể). Không
  sửa guideline mà không tăng version — nếu không, `make status`/gate G6 không nhận ra guideline đã thay đổi.

## Defect severity

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | `lane_change` bị gán ngược (`allowed` ↔ `not_allowed`) trên vạch mà sai thì hậu quả lớn | Gán `allowed` cho `lane/single_solid` (BDD10 — đoạn công trình dễ nhầm thành đứt) | Chặn, review lại toàn bộ case cùng loại vạch, xét có phải guideline thiếu rule không |
| Major | Sai class (nhầm `single_dashed` ↔ `single_solid`) nhưng chưa chắc đổi `lane_change`; hoặc bỏ sót object rõ ràng | Nhìn nhầm vạch đứt tự nhiên (BDD22) thành liền bị mòn | Ghi vào `06_calibration_report.csv`; sửa guideline nếu lặp lại ≥2 annotator |
| Minor | Class/attribute đúng nhưng hình học polyline lệch tolerance, hoặc quên tick `needs_review` ở case rõ ràng nên tick | Polyline lệch tâm vạch vài px | Không chặn, ghi note |
| Question | Annotator tick `needs_review = true` nhưng không rõ vì sao (thiếu note) | | QA đọc lại ảnh, tự quyết định gold, ghi rõ lý do vào `gold_decisions.csv`/note |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Calibration count agreement | Từ `06_calibration_measure.csv`, cột `agree` với `measure = count` | Đo annotator có thấy **cùng số object** trên cùng ảnh không — số liệu thật đo được (3/5 annotator, trước khi đủ 5 người): **46.7%** — thấp, cho thấy guideline v3 vẫn còn mơ hồ ở việc annotator có vẽ hay không vẽ một số đoạn vạch |
| Calibration attribute agreement | Từ `06_calibration_measure.csv`, cột `agree` với `measure = attr:lane_change` | Đo annotator có **cùng kết luận `allowed`/`not_allowed`** trên cùng object không — đo được: **10%** — rất thấp, đây là rủi ro lớn nhất vì đây chính là quyết định cuối bài toán |
| Lane-change decision accuracy (GTS) | Số dòng `correct=1` / tổng dòng `transfer_score.csv` | Câu hỏi chính của bài toán: quyết định đổi làn của peer có đúng không |
| Critical defect escape rate | Số case `severity=critical` sai / tổng case critical trong `gold_decisions.csv` | Rubric có "critical cap 10/20" nếu lọt lỗi nặng |

Metric high-risk tách riêng: **attribute agreement 10%** (đo ở trên) — thấp hơn nhiều so với count agreement
(46.7%), nghĩa là annotator thường **vẽ đúng chỗ có vạch** nhưng **kết luận sai** đổi làn được hay không. Đây là
tín hiệu để nhóm sửa guideline v3 → v4 trước khi freeze, tập trung làm rõ hơn rule `lane_change` thay vì rule vẽ
hình học.

## Quality gate

```text
PASS if: GTS ≥ 80% và không có critical decision nào sai
REWORK if: GTS 60–79% hoặc có 1 critical sai nhưng guideline đã có rule/escalation ngăn được (peer không theo)
REJECT / ESCALATE if: GTS < 60% hoặc ≥2 critical sai không có rule/escalation ngăn
```

Trade-off: ngưỡng 80/60 ưu tiên an toàn (sai lệch làn là lỗi nguy hiểm cho downstream lane-change assist) hơn là
tốc độ ra guideline — chấp nhận REWORK nhiều lần thay vì PASS một guideline còn rủi ro critical.
