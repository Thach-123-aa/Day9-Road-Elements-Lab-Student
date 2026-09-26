# Problem statement + downstream contract

Tối đa nửa trang, viết **trước khi mở CVAT**. Đây là bằng chứng của gate G1 (topic lock). Thay mọi placeholder
mới là xong.

## Bài toán

Trong ảnh đường phố/cao tốc, phân loại từng đoạn vạch kẻ làn cạnh làn xe mình (ego lane) thành **được phép đổi
làn** hay **không được phép đổi làn**, dựa vào số nét (đơn/đôi) và kiểu nét (đứt/liền) — không phân biệt màu vạch.

## Downstream contract

1. **Downstream task / model / user là ai?** Module hỗ trợ/quyết định chuyển làn của xe tự lái (lane-change
   assist).
2. **Output annotation nào thực sự cần?** Polyline theo tim vạch, class (`single_dashed` / `single_solid` /
   `double_dashed` / `double_solid`), attribute `lane_change` (`allowed` / `not_allowed` / `unknown`).
3. **Failure nào gây hậu quả lớn nhất?** Gán `lane_change = allowed` cho một đoạn vạch liền (thực tế không được
   đổi làn) — xe có thể đổi làn sai luật, nguy hiểm. Đây là decision `critical` trong gold.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator tick `needs_review = true` và
   để `lane_change = unknown`; QA owner đọc lại các case này khi review (`05_qa_plan.md`).

## Scope

- **Trong scope (bắt buộc label):** mọi đoạn vạch phân làn (đơn/đôi, đứt/liền) cạnh làn ego hoặc làn liền kề,
  không phân biệt màu trắng/vàng.
- **Ngoài scope (ignore):** vạch qua đường (crosswalk), mép lề đường/vỉa hè (road curb), ký hiệu/chữ sơn trên mặt
  đường (mũi tên, "ONLY", biểu tượng xe đạp), biển báo, đèn tín hiệu.
- **Geometry tolerance:** polyline bám tâm vạch, lệch ≤ 3px mỗi điểm.

## Output chấm được

- **LABEL** một đoạn vạch = vẽ polyline đúng class + gán `lane_change`.
- **IGNORE** (crosswalk, road curb, ký hiệu sơn) = không tạo object.
- **UNKNOWN** = `lane_change = unknown` khi không chắc kiểu vạch.
- **ESCALATE** = `needs_review = true` trên object đó.

Mỗi loại quyết định đều nhìn thấy được trực tiếp trong file export CVAT qua class + 2 attribute
(`lane_change`, `needs_review`).

## Dữ liệu và giới hạn

Dùng ảnh trong `data/bdd100k` (26 ảnh, đa dạng highway/city street/residential, đủ thời tiết và cả night/dawn-dusk).
Giới hạn đã biết: một số ảnh thời tiết xấu (mưa, tuyết) khiến vạch gần như không thấy được, hoặc đường quá hẹp
không có làn kề để đổi — các ảnh này dùng làm case `negative`/`occlusion`, không ép phải có quyết định `allowed`/
`not_allowed` cho mọi ảnh.
