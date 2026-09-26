# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Chốt ontology 4 class (`single_dashed`, `single_solid`, `double_dashed`, `double_solid`) không phân biệt màu trắng/vàng; thêm attribute `lane_change`; thêm rule vẽ 2 polyline riêng cho vạch đôi lệch nét; loại bỏ `crosswalk`, `road_curb`, `single_other` khỏi scope | Soi trực tiếp 26 ảnh `data/bdd100k` trước khi mở CVAT: phát hiện case vạch đôi 1 bên đứt 1 bên liền (không khớp ontology theo màu ban đầu) và nhiều case màu không ảnh hưởng quyết định đổi làn | sample_id `BDD10` (vạch đôi lệch nét), `BDD04`/`BDD15`/`BDD13` (ký hiệu/curb/crosswalk ngoài scope) |
