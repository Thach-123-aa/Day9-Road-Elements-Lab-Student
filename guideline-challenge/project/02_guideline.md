# Annotation guideline — Vạch kẻ làn nào cho phép đổi làn

**Version:** v2

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

**Trong scope:** mọi đoạn vạch phân chia làn xe (đơn, đôi, cả vạch xương cá) cạnh làn xe ego hoặc làn kề liền sát,
nhìn thấy được trong ảnh.

**Ngoài scope:** vạch qua đường (crosswalk), mép lề đường/vỉa hè (road curb) dù sơn màu gì, ký hiệu hoặc chữ sơn
trên mặt đường (mũi tên, chữ "ONLY", biểu tượng xe đạp), biển báo, đèn tín hiệu.

## 2. Annotation unit

Task dùng **ảnh tĩnh** (image), không phải video/track. Một instance là **một đoạn vạch liên tục cùng class**
(cùng màu, cùng số nét, cùng kiểu nét). Khi kiểu vạch đổi giữa đường (ví dụ từ dashed chuyển sang solid trước vạch
dừng), kết thúc polyline hiện tại và bắt đầu polyline mới tại điểm đổi kiểu.

## 3. Geometry rule

- Dùng **Polyline** cho mọi loại vạch kẻ, **trừ `lane/chevron_area`** — vạch xương cá là một **vùng** (không phải
  đường), phải dùng **Polygon**.
- Vẽ theo đúng **tim/biên** lane marking theo quy ước của bài, không nối tắt qua vùng không có vạch.
- Giữ hướng vẽ nhất quán trong cùng dataset.
- Kết thúc polyline tại điểm lane marking không còn đủ bằng chứng thị giác.
- Không dùng Polygon để thay cho lane marking chỉ vì vùng vạch có bề rộng (trừ chevron_area).
- Không tự thêm class lane mới ngoài danh sách cho phép.
- Geometry tolerance: lệch tâm vạch ≤ 3px mỗi điểm.

## 4. Taxonomy

**v2 — thêm phân biệt nét đứt/nét liền cho vạch đơn.** Lý do đổi từ v1: nhiều ảnh trong `data/bdd100k` có cả đoạn
nét đứt lẫn nét liền của cùng một màu vạch, gộp chung vào một class sẽ mất thông tin quyết định đổi làn (dashed
= allowed, solid = not_allowed) — hai kiểu nét khác nhau phải là hai class khác nhau, không thể coi là cùng một
class rồi tự suy luận qua attribute.

| Class | Định nghĩa & phạm vi áp dụng | Quyết định |
|---|---|---|
| `lane/double_white` | Vạch đôi trắng (liền/đứt) — phân chia các làn xe cùng chiều, không được lấn làn/chuyển làn tuỳ tiện | `not_allowed` |
| `lane/double_yellow` | Vạch đôi vàng (liền/đứt) — phân chia 2 chiều xe chạy ngược chiều nhau, cấm lấn làn/vượt | `not_allowed` |
| `lane/single_white_dashed` | Vạch đơn trắng nét đứt — phân chia làn cùng chiều | `allowed` |
| `lane/single_white_solid` | Vạch đơn trắng nét liền — phân chia làn cùng chiều | `not_allowed` |
| `lane/single_yellow_dashed` | Vạch đơn vàng nét đứt — tim đường 2 chiều, phía nét đứt được vượt | `allowed` |
| `lane/single_yellow_solid` | Vạch đơn vàng nét liền — tim đường 2 chiều hoặc mép đường | `not_allowed` |
| `lane/chevron_area` | Vạch xương cá — vùng cấm đi vào | `not_allowed` |

Quyết định đổi làn nằm ngay trong **tên class** (không cần attribute riêng để chọn `allowed`/`not_allowed`) — vì
mỗi class đã có nghĩa cố định theo luật giao thông. Mỗi class có 2 attribute:

- **`occluded`** (checkbox): tick khi vạch bị vật khác che một phần nhưng vẫn suy luận được hình dạng thật (xem
  mục 6, ví dụ BDD16).
- **`needs_review`** (checkbox): tick khi không chắc chắn về class dù đã đọc guideline.

Bảng đầy đủ và JSON khớp 1:1 ở `03_ontology_and_cvat_setup.md` / `03_cvat_labels.json` — hai nơi phải khớp nhau.

## 5. Inclusion / exclusion

**Bắt buộc label:** mọi đoạn vạch phân làn (đơn/đôi, đứt/liền, cả chevron_area) cạnh làn ego hoặc làn liền kề.

**Ignore (không vẽ):**
- Vạch qua đường (crosswalk), kể cả loại màu vàng dạng thang.
- Mép lề đường/vỉa hè (curb), kể cả curb sơn sọc đỏ-trắng (khu cấm dừng đỗ).
- Chữ, mũi tên, biểu tượng sơn trên mặt đường (ví dụ ký hiệu làn xe đạp, chữ "ONLY").
- Vạch của làn quá xa, không liên quan đến ego lane hoặc làn kề trực tiếp.

## 6. Visibility / occlusion

Rule cụ thể đúc kết từ soi ảnh thật trong `data/bdd100k`:

- **Vạch quá mờ, không còn phân biệt được loại nào** (ví dụ BDD15): **có thể không vẽ** — không ép đoán khi bằng
  chứng thị giác không đủ.
- **Ban đêm, không đủ sáng để xác định làn** (ví dụ BDD26): **không đánh** (không tạo object) khi không đủ ánh
  sáng để xác định.
- **Đoạn đường đang sửa làm vạch nét liền bị mất một khoảng ngắn** (ví dụ BDD10): **vẫn gán là vạch nét liền** —
  khoảng mất do công trình không đổi bản chất của vạch.
- **Vạch dài nhưng có các đoạn đứt quãng đều nhau** (ví dụ BDD22): **vẫn tính là vạch nét đứt** — cần nhìn kỹ
  khoảng cách giữa các đoạn để phân biệt với vạch liền đã bị mòn/bong tróc.
- **Vạch bị xe phía trước che một phần** (ví dụ BDD16, vạch đứt trắng bị bánh xe/thân xe che mất một khoảng):
  **LABEL — vẫn vẽ, gán `occluded = true`.** Lý do: downstream model đọc output thực tế từ camera và có thể
  predict được vạch tồn tại phía sau xe che; nếu **không** gán (bỏ đoạn bị che), model sẽ học sai rằng vạch không
  tồn tại ở đó, trong khi thực tế vạch vẫn liên tục, chỉ là camera không thấy được đoạn đó.

## 7. Ambiguity / escalation

| Tình huống | Quyết định | Thể hiện trong CVAT |
|---|---|---|
| Vạch quá mờ, không phân biệt được loại | IGNORE (không vẽ) | Không tạo object |
| Ban đêm không đủ sáng xác định làn | IGNORE (không vẽ) | Không tạo object |
| Vạch bị công trình/vật cản che một phần nhưng suy luận được | LABEL | Class đúng + `occluded = true` |
| Không chắc chắn về class dù đã đọc guideline | ESCALATE | `needs_review = true` |

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh.

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Cao tốc nhiều làn, vạch trắng đứt + vạch vàng liền mép trái rõ | `single_white_dashed`; `single_yellow_solid` mép trái | Mục 4 |
| BDD07 | Ngã tư khu dân cư, vạch đôi liền vàng ở giữa, có crosswalk | `double_yellow`; crosswalk **không vẽ** | Mục 5 (ignore crosswalk) |
| BDD15 | Vạch mờ, không phân biệt được loại | **Không vẽ** | Mục 6 |
| BDD16 | Vạch đứt trắng bị bánh xe phía trước che một khoảng | `single_white_dashed`, `occluded = true` | Mục 6 |

## 10. Common mistakes

- Gộp nét đứt và nét liền cùng màu vào một class — sai từ v2, phải tách theo đúng bảng mục 4.
- Vẽ luôn crosswalk hoặc road curb vì trông giống "vạch" — ngoài scope, không vẽ.
- Bỏ qua đoạn vạch bị che thay vì gán `occluded = true` — làm model học sai là vạch không tồn tại ở đó.
- Cố đoán class khi ảnh quá mờ/quá tối thay vì để trống (không vẽ) theo mục 6.
- Dùng Polygon cho vạch thường (chỉ `chevron_area` mới dùng Polygon).
