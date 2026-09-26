# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: EC01
Sample: BDD15
Scene: Vạch kẻ làn bị mờ đến mức không còn phân biệt được là loại nào
Observation: Vạch quá mờ, không đủ bằng chứng thị giác để xác định class
Decision: IGNORE — không vẽ
Expected: Không tạo object cho đoạn vạch này
Rationale: Ép đoán khi không đủ bằng chứng sẽ tạo nhãn sai, gây nhiễu dữ liệu cho downstream model — thà thiếu còn hơn sai
Common mistake: Cố đoán class dựa trên vị trí "chắc là" thay vì bằng chứng thị giác thật
Diversity: ambiguity, low_visibility, escalation (ranh giới giữa IGNORE và ESCALATE)

---

CASE ID: EC02
Sample: BDD26
Scene: Ban đêm, ánh sáng yếu, không nhìn rõ vạch thuộc làn nào
Observation: Vẫn thấy được hình dạng đường nét (dashed), nhưng không chắc chắn hoàn toàn do thiếu sáng
Decision: LABEL + ESCALATE
Expected: `lane/single_dashed`, `lane_change` gán theo quan sát, `needs_review = true`
Rationale: Khác case EC01 (quá mờ để thấy) — ở đây còn thấy hình dạng, chỉ không chắc 100%, nên vẫn ghi nhận nhưng gắn cờ để người khác kiểm tra lại thay vì bỏ qua hoàn toàn
Common mistake: Bỏ qua không vẽ (giống EC01) trong khi vẫn còn đủ bằng chứng để label
Diversity: low_visibility, escalation

---

CASE ID: EC03
Sample: BDD10
Scene: Đoạn đường đang thi công làm mất một khoảng ngắn của vạch nét liền
Observation: Vạch nét liền bị đứt đoạn do công trình (không phải do bản chất vạch là nét đứt)
Decision: LABEL
Expected: `lane/single_solid`, `lane_change = not_allowed`, `needs_review = false`, vẽ nối liền qua khoảng thi công
Rationale: Nguyên nhân gây gián đoạn (thi công) không làm thay đổi bản chất pháp lý của vạch — nếu gán nhầm thành `single_dashed` sẽ đảo ngược quyết định đổi làn (not_allowed → allowed), đây là lỗi **critical**
Common mistake: Thấy đoạn đứt quãng do thi công rồi nhầm gán thành `single_dashed`
Diversity: occlusion, critical

---

CASE ID: EC04
Sample: BDD22
Scene: Vạch đơn nét đứt tự nhiên, đoạn dài với nhiều khoảng hở đều nhau
Observation: Vạch có khoảng hở đều, lặp lại theo chu kỳ — cần phân biệt với vạch liền đã mòn/bong tróc
Decision: LABEL
Expected: `lane/single_dashed`, `lane_change = allowed`
Rationale: Khoảng hở đều và có quy luật là dấu hiệu phân biệt dashed thật với solid bị mòn (khoảng hở bất thường) — annotator phải nhìn kỹ trước khi gán
Common mistake: Nhầm vạch đứt tự nhiên với vạch liền bị mòn hoặc ngược lại
Diversity: ambiguity, small_far

---

CASE ID: EC05
Sample: BDD16
Scene: Vạch đứt trắng đang phân làn đang chạy thẳng bị bánh xe/thân xe phía trước che mất một khoảng
Observation: Object vẫn nằm trong scene nhưng bị vật khác (xe) che một phần
Decision: LABEL + ESCALATE
Expected: `lane/single_dashed`, `lane_change = allowed`, vẽ nối polyline qua đoạn bị xe che (suy luận từ 2 đầu còn thấy), `needs_review = true`
Rationale: Ontology hiện không có attribute `occluded` riêng, nên dùng `needs_review = true` để đánh dấu object này có phần bị che — vẫn suy luận và nối polyline qua đoạn che vì AI đọc output thực tế từ camera và có thể predict vạch tồn tại phía sau xe; nếu bỏ đoạn bị che (không nối), model sẽ học sai rằng vạch không tồn tại ở đó
Common mistake: Ngắt polyline tại điểm bị che thay vì nối qua; quên tick `needs_review`; hoặc nhầm lẫn với rule của EC06 (mờ do thời tiết) rồi không dám nối
Diversity: occlusion, conflict (dễ nhầm với rule EC06)

---

CASE ID: EC06
Sample: BDD17
Scene: Vạch kẻ bị mờ/che khuất bởi nước mưa và thời tiết xấu
Observation: Một phần vạch không nhìn rõ do nhoè nước mưa, không phải do vật cản đặc che
Decision: LABEL — chỉ phần nhìn thấy được
Expected: Polyline chỉ vẽ đoạn quan sát rõ, không kéo dài/nối qua đoạn bị mờ
Rationale: Mờ do thời tiết không cho bằng chứng chắc chắn về hình dạng thật ở đoạn bị che như khi bị vật cản đặc (khác EC05) — không được đoán, tránh nhiễu dữ liệu
Common mistake: Áp dụng rule "nối qua chỗ che" của EC05 (bị xe che) sang case này (bị mờ do thời tiết) — hai case có bản chất khác nhau dù đều là "occlusion"
Diversity: occlusion, low_visibility, conflict (dễ nhầm với rule EC05)

---

CASE ID: EC07
Sample: BDD13
Scene: Vạch kẻ chuyển đổi kiểu trên cùng một luồng giao thông — từ liền sang đứt quãng rồi lại trở thành liền
Observation: Cùng một vạch vật lý nhưng đổi class giữa đường (solid → dashed → solid)
Decision: LABEL — 2-3 đoạn riêng biệt
Expected: 3 polyline tách riêng theo đúng class từng đoạn: `single_solid`, `single_dashed`, `single_solid`
Rationale: Label riêng từng đoạn để model nhận diện được chính xác từng phần của vạch đường, tương ứng đúng quyết định đổi làn khác nhau tại mỗi đoạn
Common mistake: Vẽ một polyline xuyên suốt cả 3 đoạn, chọn 1 class duy nhất cho cả đường — làm mất thông tin đoạn nào thực sự cho phép đổi làn
Diversity: conflict, ambiguity

---

CASE ID: EC08
Sample: BDD02
Scene: Giao lộ trong thành phố, chỉ có vạch qua đường (crosswalk), không có vạch phân làn nào trong khung hình
Observation: Toàn bộ khung hình không có đối tượng nào thuộc scope (crosswalk là ngoài scope theo mục 5)
Decision: IGNORE cả ảnh
Expected: Không tạo object nào; ảnh này gắn tag `negative` trong `sample_pack.csv`
Rationale: Không có vạch phân làn không phải lỗi bỏ sót của annotator — cần phân biệt rõ với case "có vạch nhưng quên vẽ", để QA không tính nhầm thành thiếu sót khi review
Common mistake: Cố vẽ crosswalk hoặc vạch của làn quá xa để "có gì đó" trong ảnh, vi phạm mục 5 (ngoài scope)
Diversity: negative

---
