# Annotation guideline — Vạch kẻ làn nào cho phép đổi làn

**Version:** v4

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

**Trong scope:** mọi đoạn vạch phân chia làn xe (đơn, đôi, cả vạch xương cá — chevron) cạnh làn xe ego hoặc làn kề,
nhìn thấy được trong ảnh — **kể cả vạch sơn mép ngoài cùng của làn xe** (edge line, nằm sát lề nhưng vẫn là vạch sơn
phân định làn, ví dụ BDD01). "Làn kề" không giới hạn ở làn sát ego — bao gồm cả vạch ranh giới phía ngoài của làn kề
(giữa làn kề và làn tiếp theo), miễn còn nhìn thấy trong khung hình.

**Ngoài scope:** vạch qua đường (crosswalk), **mép lề đường/vỉa hè vật lý** (road curb — gờ bê tông, đá, không phải
vạch sơn) dù sơn màu gì, ký hiệu hoặc chữ sơn trên mặt đường (mũi tên, chữ "ONLY", biểu tượng xe đạp), biển báo, đèn
tín hiệu.

> **v4 — làm rõ khác biệt "road curb" (ngoài scope) và "edge line mép làn" (trong scope):** ở v3, mục 1/5 nói loại
> trừ "mép lề đường" nhưng ví dụ BDD01 mục 9 lại label vạch vàng liền mép trái, gây mâu thuẫn — annotator (Group 1
> peer-test) phải tự suy diễn theo ví dụ. v4 tách rõ 2 khái niệm: **road curb** = ranh giới vật lý ngoài cùng của
> mặt đường (không phải vạch sơn) → ignore; **edge line** = vạch sơn phân làn nằm ở vị trí ngoài cùng (dù sát lề) →
> vẫn label bình thường theo class tương ứng.

## 2. Annotation unit

Task dùng **ảnh tĩnh** (image), không phải video/track. Một instance là **một đoạn vạch liên tục cùng class**. Khi
kiểu vạch đổi giữa đường (ví dụ từ liền chuyển thành đứt rồi lại thành liền — xem mục 6, BDD13), kết thúc polyline
hiện tại và bắt đầu polyline mới tại điểm đổi kiểu — **không** vẽ một đường xuyên suốt qua nhiều kiểu.

## 3. Geometry rule

- Dùng **Polyline** cho mọi loại vạch kẻ (`single_dashed`, `single_solid`, `double_dashed`, `double_solid`).
- Riêng `lane/chevron_area` (vạch xương cá) là một **vùng**, không phải đường — dùng **Polygon**.
- Vẽ theo đúng tim/biên lane marking theo quy ước của bài, không nối tắt qua vùng không có vạch.
- Giữ hướng vẽ nhất quán trong cùng dataset.
- Kết thúc polyline tại điểm lane marking không còn đủ bằng chứng thị giác.
- Không tự thêm class lane mới ngoài danh sách cho phép.
- Geometry tolerance: lệch tâm vạch ≤ 3px mỗi điểm.

## 4. Taxonomy

| Class | Định nghĩa & phạm vi áp dụng | Quyết định |
|---|---|---|
| `lane/single_dashed` | Vạch đơn nét đứt — phân chia làn cùng chiều | `allowed` |
| `lane/single_solid` | Vạch đơn nét liền — phân chia làn cùng chiều | `not_allowed` |
| `lane/double_dashed` | Vạch đôi nét đứt (hiếm) | `allowed` |
| `lane/double_solid` | Vạch đôi nét liền — phân chia 2 chiều xe chạy ngược chiều nhau, cấm lấn làn/vượt | `not_allowed` |
| `lane/chevron_area` | Vạch xương cá — vùng cấm đi vào | `not_allowed` |

Mỗi class có 2 attribute:

- **`lane_change`** (select, mặc định `__undefined__` để ép chọn): `allowed` / `not_allowed` / `unknown`.
- **`needs_review`** (checkbox): tick khi không chắc chắn dù đã đọc guideline.

Không phân biệt màu (trắng/vàng gộp chung) — quyết định đổi làn dựa trên số nét (đơn/đôi) và kiểu nét (đứt/liền),
annotator tự gán `lane_change` theo bảng trên, không suy luận thêm. Bảng đầy đủ khớp 1:1 với
`03_ontology_and_cvat_setup.md` / `03_cvat_labels.json`.

## 5. Inclusion / exclusion

**Bắt buộc label:** mọi đoạn vạch phân làn (đơn/đôi, đứt/liền, chevron) cạnh làn ego hoặc làn liền kề.

**Ignore (không vẽ):**
- Vạch qua đường (crosswalk).
- Mép lề đường/vỉa hè **vật lý** (curb — gờ bê tông/đá, không phải vạch sơn).
- Chữ, mũi tên, biểu tượng sơn trên mặt đường.
- Vạch của làn quá xa, không còn nhìn thấy được trong khung hình.

**v4 — vạch đứt tự nhiên vẽ nối xuyên suốt:** một đoạn vạch đứt (`single_dashed`/`double_dashed`) vốn có khoảng hở
giữa các nét sơn — đây là **bản chất của chính nó**, không phải bị che. Vẽ **một polyline duy nhất nối qua các
khoảng hở này** (đi theo tim của toàn bộ đoạn vạch đứt), không tách thành nhiều polyline riêng cho từng nét sơn.
Khác với mục 6: nối qua khoảng hở tự nhiên của vạch đứt ≠ nối qua đoạn bị **vật cản/thời tiết che** — hai rule độc
lập, xem ví dụ phân biệt ở mục 9 (BDD22 vs BDD16/BDD17).

## 6. Visibility / occlusion

Rule đúc kết từ soi ảnh thật trong `data/bdd100k` (edge case đầy đủ ở `04_edge_cases/edge_case_cards.md`):

- **Vạch quá mờ, không còn phân biệt được loại nào** (BDD15): **có thể không vẽ** — không ép đoán khi bằng chứng
  thị giác không đủ.
- **Ban đêm, không nhìn rõ vạch thuộc làn nào** (BDD26): **vẫn Label**, nhưng tick `needs_review = true` — khác
  với case quá mờ (BDD15), ở đây vẫn còn thấy được hình dạng vạch, chỉ không chắc chắn hoàn toàn nên đánh dấu để
  người khác xem lại, không bỏ qua.
- **Đoạn đường đang sửa làm vạch nét liền bị mất một khoảng ngắn** (BDD10): **vẫn gán là vạch nét liền**
  (`single_solid`, `lane_change = not_allowed`) — khoảng mất do công trình không đổi bản chất của vạch, không tick
  `needs_review`.
- **Vạch dài nhưng có các đoạn đứt quãng đều nhau** (BDD22): **vẫn tính là vạch nét đứt** — cần nhìn kỹ khoảng cách
  giữa các đoạn để phân biệt với vạch liền đã bị mòn/bong tróc.
- **Vạch bị vật cản (xe) che một phần** (BDD16, vạch đứt trắng bị bánh xe/thân xe phía trước che mất một khoảng):
  **Label + tick `needs_review = true`** — vẫn vẽ nối qua đoạn bị che, suy luận hình dạng thật dựa trên 2 đầu còn
  thấy được, gán `lane_change` theo đúng kiểu vạch quan sát được. Lý do: downstream model đọc output thực tế từ
  camera và có thể predict được vạch tồn tại phía sau xe che; nếu bỏ đoạn bị che, model sẽ học sai rằng vạch không
  tồn tại ở đó. Tick `needs_review` để người review biết object này có phần suy luận từ occlusion, không phải quan
  sát trực tiếp 100%.
- **Ảnh không có vạch phân làn nào trong khung hình** (BDD02, giao lộ chỉ có crosswalk): **IGNORE cả ảnh** — không
  tạo object nào, gắn **tag `negative`** (từ v4, `negative` là 1 CVAT tag thật trong `03_cvat_labels.json`, không
  chỉ ghi ở `sample_pack.csv` như v3 — annotator bấm **Setup tag** để đánh dấu ảnh, phân biệt rõ "cố ý không có gì
  để vẽ" với "quên vẽ"). Đây không phải lỗi bỏ sót.
- **[v4 — bắt buộc, sửa lỗi critical phát hiện qua blind test với Group 1, sample BDD24]** Chỉ nhìn thấy được **một
  đoạn ngắn** của vạch, và **cả 2 đầu đoạn đó đều bị che/mờ** (không còn bằng chứng ở đầu nào để xác định đoạn tiếp
  theo là đứt hay liền): **bắt buộc `lane_change = unknown`** và `needs_review = true`. **Không được** tự chọn
  `allowed`/`not_allowed` dựa trên đoạn ngắn nhìn thấy dù đã tick `needs_review` — tick `needs_review` một mình
  không thay thế được việc phải chọn đúng `unknown`. Đây là gap khiến peer (Group 1) chọn `single_dashed/allowed`
  cho BDD24 thay vì `unknown/unknown` như gold kỳ vọng, gây 1 critical escape trong `transfer_score.csv` — v3 chỉ
  nói "tick needs_review khi không chắc" nhưng không nói rõ trường hợp này còn phải đổi `lane_change` thành
  `unknown`, nên peer hiểu lầm là được phép vừa escalate vừa tự đoán class.
- **Vạch bị mờ/che bởi thời tiết (mưa, sương)** (BDD17): **Label — chỉ phần nhìn thấy được**, khác với case bị xe
  che ở trên. Không được đoán phần bị mờ, không nối polyline qua đoạn không thấy rõ, tránh nhiễu dữ liệu — vì mờ
  do thời tiết không cho bằng chứng chắc chắn về hình dạng thật như khi bị vật cản đặc che khuất.
- **Vạch chuyển đổi kiểu trên cùng một luồng giao thông** (BDD13: liền → đứt → liền): **Label thành các đoạn
  polyline riêng biệt**, mỗi đoạn đúng class của nó (`single_solid`, `single_dashed`, `single_solid`) — không vẽ
  một đường xuyên suốt, để model nhận diện được từng phần riêng biệt của vạch đường.

## 7. Ambiguity / escalation

| Tình huống | Quyết định | Thể hiện trong CVAT |
|---|---|---|
| Vạch quá mờ, không phân biệt được loại nào | IGNORE (không vẽ) | Không tạo object |
| Ban đêm, thấy hình dạng nhưng không chắc chắn | LABEL | Class đúng + `needs_review = true` |
| Bị vật cản (xe) che một phần, suy luận được hình dạng thật | LABEL + ESCALATE | Class đúng, nối qua đoạn che, `needs_review = true` |
| Bị mờ/che bởi thời tiết (mưa, sương) | LABEL phần thấy được | Chỉ vẽ đoạn nhìn rõ, không đoán phần còn lại |
| Vạch chuyển đổi kiểu giữa đường | LABEL nhiều đoạn | Mỗi đoạn 1 polyline riêng, đúng class của nó |
| Ảnh không có vạch phân làn nào trong khung hình | IGNORE cả ảnh | Không tạo object; tick CVAT tag `negative` |
| Chỉ thấy 1 đoạn ngắn, cả 2 đầu bị che/mờ, không xác định được đứt/liền | ESCALATE bắt buộc | `lane_change = unknown` **và** `needs_review = true` — không được tự đoán class |
| Không chắc chắn về class dù đã đọc guideline | ESCALATE | `needs_review = true` |

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh.

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Cao tốc nhiều làn, vạch trắng đứt + vạch vàng liền mép trái rõ | `single_dashed(allowed)`; `single_solid(not_allowed)` mép trái | Mục 4 |
| BDD16 | Vạch đứt trắng bị bánh xe phía trước che một khoảng | `single_dashed(allowed)`, nối qua đoạn che, `needs_review=true` | Mục 6 |
| BDD17 | Vạch bị mờ do mưa | `single_*` chỉ phần nhìn thấy được | Mục 6 |
| BDD13 | Vạch chuyển từ liền sang đứt rồi lại liền trên cùng luồng | 3 polyline riêng: `single_solid`, `single_dashed`, `single_solid` | Mục 6 |
| BDD02 | Giao lộ chỉ có crosswalk, không có vạch phân làn | Không tạo object, tag `negative` | Mục 6 |
| BDD24 | Chỉ 1 đoạn ngắn nhìn thấy, 2 đầu bị tuyết/capo che, không rõ đứt hay liền | `lane_change = unknown`, `needs_review = true` — không tự đoán class | Mục 6 (v4) |

## 10. Common mistakes

- Vẽ một polyline xuyên suốt qua đoạn vạch đổi kiểu (BDD13) thay vì tách thành nhiều đoạn.
- Nối qua đoạn bị mờ do thời tiết (BDD17) như thể suy luận được — chỉ được nối qua khi bị **vật cản** che (BDD16),
  không phải khi bị **mờ/nhoè** do thời tiết.
- Bỏ qua (không vẽ) khi chỉ là "không chắc chắn" (BDD26) — chỉ bỏ qua khi thật sự **quá mờ không phân biệt được**
  (BDD15); còn lại vẫn Label + `needs_review`.
- Vẽ luôn crosswalk hoặc road curb **vật lý** (gờ bê tông/đá) vì trông giống "vạch" — ngoài scope, không vẽ; nhưng
  đừng nhầm ngược lại — **vạch sơn mép ngoài làn (edge line) vẫn phải vẽ**, không phải cứ nằm sát lề là bỏ qua.
- Quên đổi `lane_change` khỏi `__undefined__` trước khi export.
- **[v4]** Chỉ thấy 1 đoạn ngắn, 2 đầu bị che/mờ, vẫn tự chọn `allowed`/`not_allowed` (dù đã tick `needs_review`)
  thay vì bắt buộc chọn `unknown` — đây là lỗi **critical**, đã gây 1 lần sai khi Group 1 peer-test blind pack.
