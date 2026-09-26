# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `lane/single_dashed` | polyline | class | — | — | — | Vạch đơn nét đứt, không phân biệt màu |
| `lane/single_solid` | polyline | class | — | — | — | Vạch đơn nét liền, không phân biệt màu |
| `lane/double_dashed` | polyline | class | — | — | — | Vạch đôi, cả 2 nét đứt (hiếm) |
| `lane/double_solid` | polyline | class | — | — | — | Vạch đôi nét liền, phổ biến nhất trong data BDD |
| `lane/chevron_area` | **polygon** | class | — | — | — | Vạch xương cá — vùng cấm đi vào, là vùng chứ không phải đường nên dùng Polygon |
| `negative` | **tag** | class | — | — | — | **(v4)** Đánh dấu cả ảnh không có vạch phân làn nào để vẽ — phân biệt "cố ý không có gì" với "quên vẽ" |
| `lane_change` | attribute (select) | attribute | `__undefined__`, `allowed`, `not_allowed`, `unknown` | `__undefined__` | false | Quyết định chính cần chấm; `__undefined__` đứng đầu để ép annotator chọn, tránh bias mặc định thành `allowed` |
| `needs_review` | attribute (checkbox) | attribute | `false` | `false` | false | Escalate object khi không chắc chắn dù đã đọc guideline |

## Class hay attribute

Kiểu vạch (số nét + đứt/liền) là **class** vì đó là thứ mắt nhìn thấy trực tiếp, cố định trong ảnh. Quyết định đổi
làn là **attribute** (`lane_change`) vì đó là kết luận annotator rút ra từ class — tách riêng để kiểm tra được:
annotator nhìn đúng vạch (class đúng) nhưng kết luận sai (`lane_change` sai), hay ngược lại. Màu (trắng/vàng)
**không** đưa vào ontology — nhóm xác định màu không ảnh hưởng quyết định đổi làn trong bài toán này. Default
`lane_change = __undefined__` để annotator quên gán sẽ bị phát hiện ngay khi export.

**Quyết định (v3):** case bị vật cản che (EC05, BDD16) dùng **`needs_review = true`** để đánh dấu, không thêm
attribute `occluded` riêng — giữ ontology gọn 2 attribute (`lane_change`, `needs_review`).

**Quyết định (v4):** thêm tag `negative` thật vào `03_cvat_labels.json` (trước đó guideline chỉ nói ghi tag
`negative` ở `sample_pack.csv`, nhưng CVAT không có label kiểu tag tương ứng — peer Group 1 chỉ ra export ảnh cố ý
để trống và ảnh bị quên label trông giống hệt nhau, không phân biệt được). Việc thêm class/tag mới **sau khi
freeze vẫn hợp lệ** vì `gold_decisions.csv`/`sample_pack.csv` (2 file thật sự bị khoá) không đổi — chỉ
`02_guideline.md`/`03_cvat_labels.json` được phép tiếp tục sửa ở bước 08 theo đúng quy trình.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): CVAT 2.75.1 tại `http://localhost:8080`
- **Tên task calibration** (có version guideline, ví dụ `mixigaming-calib-v1`): `mixigaming-calib-v3` (5 ảnh
  calibration, dùng lúc guideline đang v3 — xem `06_calibration_exports/`)
- **Guide của task đã dán `02_guideline.md`?** Có (bản v3 lúc calibration; cần dán lại bản v4 nếu mở task mới)
- **Nhóm dùng Track hay Shape, vì sao:** Shape — task dùng ảnh tĩnh (`data/bdd100k`), không có video/track

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

Đinh Công Minh (chưa tham gia setup, đóng vai QA owner) mở task `mixigaming-calib-v3` — thao tác trơn tru, không
vấp chỗ nào: mở được Guide, hiểu ngay label nào dùng tool nào (polyline cho 4 class vạch, polygon cho
`chevron_area`), gán đủ 2 attribute (`lane_change`, `needs_review`) không cần hỏi thêm.
