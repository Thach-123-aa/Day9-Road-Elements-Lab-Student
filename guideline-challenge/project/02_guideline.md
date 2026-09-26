# Annotation guideline — Vạch kẻ làn nào cho phép đổi làn

**Version:** v1

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

Xác định từng đoạn vạch kẻ làn cạnh làn xe mình (ego lane) là **được phép đổi làn** hay **không được phép đổi làn**,
phục vụ module hỗ trợ/quyết định chuyển làn của xe tự lái.

**Trong scope:** mọi đoạn vạch phân chia làn xe (đơn, đôi — bất kể màu trắng hay vàng) cạnh làn xe ego hoặc làn kề
liền sát, nhìn thấy được trong ảnh.

**Ngoài scope:** vạch qua đường (crosswalk), mép lề đường/vỉa hè (road curb) dù sơn màu gì, ký hiệu hoặc chữ sơn
trên mặt đường (mũi tên, chữ "ONLY", biểu tượng xe đạp), biển báo, đèn tín hiệu.

## 2. Annotation unit

Task dùng **ảnh tĩnh** (image), không phải video/track. Một instance là **một đoạn vạch liên tục cùng kiểu**
(cùng số nét single/double, cùng kiểu nét dashed/solid). Khi kiểu vạch đổi giữa đường (ví dụ từ dashed chuyển
sang solid trước vạch dừng), kết thúc polyline hiện tại và bắt đầu polyline mới tại điểm đổi kiểu.

## 3. Geometry rule

- Dùng **Polyline**, không dùng Polygon dù vạch có bề rộng nhìn thấy được.
- Vẽ theo đúng **tim vạch** (giữa 2 mép sơn).
- Không nối tắt polyline qua đoạn đường không có vạch (ví dụ qua giao lộ không sơn).
- Kết thúc polyline tại điểm vạch không còn đủ bằng chứng thị giác (bị che hẳn, hết khung hình, hoặc đổi kiểu).
- Giữ hướng vẽ nhất quán trong toàn bộ dataset: **luôn vẽ từ điểm gần xe (đáy ảnh) ra xa xe (đỉnh ảnh)**.
- Geometry tolerance: lệch tâm vạch ≤ 3px mỗi điểm.

## 4. Taxonomy

4 class, **không phân biệt màu** (trắng và vàng gộp chung), phân biệt theo số nét và kiểu nét:

| Class | Ý nghĩa |
|---|---|
| `lane/single_dashed` | Vạch đơn nét đứt |
| `lane/single_solid` | Vạch đơn nét liền |
| `lane/double_dashed` | Vạch đôi, cả 2 nét đứt |
| `lane/double_solid` | Vạch đôi, cả 2 nét liền |

Mỗi class có 2 attribute:

- **`lane_change`** (bắt buộc chọn, mặc định `__undefined__` để ép chọn): `allowed` / `not_allowed` / `unknown`.
  Đây là **class** cho kiểu vạch (cái mắt nhìn thấy), còn `lane_change` là **attribute quyết định** (cái annotator
  kết luận) — tách riêng hai trường để có thể kiểm tra annotator nhìn đúng vạch nhưng kết luận sai, hay ngược lại.
- **`needs_review`** (checkbox): tick khi không chắc, dù đã đọc guideline.

Bảng đầy đủ và JSON khớp 1:1 ở `03_ontology_and_cvat_setup.md` / `03_cvat_labels.json` — hai nơi phải khớp nhau.

## 5. Inclusion / exclusion

**Bắt buộc label:** mọi đoạn vạch phân làn (đơn/đôi, đứt/liền) cạnh làn ego hoặc làn liền kề, không phân biệt màu.

**Ignore (không vẽ):**
- Vạch qua đường (crosswalk), kể cả loại màu vàng dạng thang.
- Mép lề đường/vỉa hè (curb), kể cả curb sơn sọc đỏ-trắng (khu cấm dừng đỗ).
- Chữ, mũi tên, biểu tượng sơn trên mặt đường (ví dụ ký hiệu làn xe đạp, chữ "ONLY").
- Vạch của làn quá xa, không liên quan đến ego lane hoặc làn kề trực tiếp.

**Case đặc biệt — vạch đôi lệch nét (1 bên nét đứt, 1 bên nét liền):** không dùng class `double_dashed` hay
`double_solid` cho case này (không khớp nghĩa). Thay vào đó, **vẽ 2 polyline riêng**: một `single_dashed` cho nửa
nét đứt, một `single_solid` cho nửa nét liền — mỗi polyline tự chọn `lane_change` theo đúng nét của nó (phía nét
đứt → `allowed`, phía nét liền → `not_allowed`).

## 6. Visibility / occlusion

- Vạch bị che **một phần** (xe, cọc tiêu, tuyết): vẫn vẽ nếu còn đủ đoạn liên tục để xác định kiểu nét. Nếu phần
  che làm mất khả năng phân biệt dashed/solid → `lane_change: unknown` + `needs_review: true`.
- Vạch mờ do thời tiết (mưa, sương, ánh sáng yếu) nhưng vẫn đoán được kiểu với độ tin cậy cao → vẽ bình thường.
  Không chắc → `unknown` + `needs_review`.
- Vạch **bong tróc/mòn**: phân biệt dashed thật với solid đã mòn bằng quy luật khoảng hở — khoảng hở đều, lặp lại
  theo chu kỳ → tính là `dashed`; khoảng hở bất thường, có vệt sơn mờ liên tục bên dưới → tính là `solid`.
- Ảnh quá tối/mưa nặng gần như không còn thấy vạch nào: không cố vẽ, không đoán — ảnh đó gắn tag `negative` trong
  `sample_pack.csv`, không tính là annotator bỏ sót.
- Ảnh chỉ có 1 làn, không có làn kề để đổi (ví dụ phố hẹp có xe đậu 2 bên): vẫn vẽ đúng class vạch nhìn thấy, nhưng
  `lane_change: unknown` vì câu hỏi "đổi làn được không" không áp dụng được khi không có làn thứ 2.

## 7. Ambiguity / escalation

| Tình huống | Quyết định | Thể hiện trong CVAT |
|---|---|---|
| Không chắc dashed hay solid | UNKNOWN | `lane_change = unknown`, `needs_review = true` |
| Vạch cong biên vùng gore (chỗ tách nhánh exit, không có làn thật phía bên kia) | LABEL, coi như ranh giới không được vượt | Class `single_solid`, `lane_change = not_allowed` |
| Vạch đôi lệch nét | LABEL theo rule mục 5 | 2 object `single_dashed` + `single_solid` tách riêng |
| Cả ảnh không xác định được (quá tối/mờ) | IGNORE cả ảnh | Không tạo object; gắn tag `negative` ở `sample_pack.csv` (ontology không có tag escalate cấp ảnh trong CVAT — ghi chú thủ công vào `06_calibration_report.csv`/`peer_feedback.md` khi cần) |

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh.

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Cao tốc nhiều làn, vạch trắng đứt + vạch vàng liền mép trái rõ | `single_dashed(allowed)` giữa các làn; `single_solid(not_allowed)` mép trái | Mục 4, 5 |
| BDD07 | Ngã tư khu dân cư, vạch đôi liền ở giữa, có crosswalk | `double_solid(not_allowed)`; crosswalk **không vẽ** | Mục 5 (ignore crosswalk) |
| BDD04 | Phố dốc SF, vạch đôi liền vàng + ký hiệu làn xe đạp + chữ "ONLY" sơn trên đường | `double_solid(not_allowed)` cho vạch; **không vẽ** ký hiệu/chữ | Mục 5 (ngoài scope) |

## 10. Common mistakes

- Vẽ vạch đôi lệch nét thành 1 object `double_solid` duy nhất — sai, phải tách 2 polyline (mục 5).
- Vẽ luôn crosswalk hoặc road curb vì trông giống "vạch" — ngoài scope, không vẽ.
- Cố đoán kiểu vạch khi ảnh quá tối/mờ thay vì để `unknown` + `needs_review`.
- Quên đổi `lane_change` khỏi `__undefined__` — CVAT không chặn submit khi còn undefined, phải tự kiểm trước khi
  export (mục Self-QC trong `05_qa_plan.md`).
- Gán `lane_change = allowed` cho vạch liền chỉ vì làn bên cạnh trông "trống", trong khi loại nét mới là căn cứ.
