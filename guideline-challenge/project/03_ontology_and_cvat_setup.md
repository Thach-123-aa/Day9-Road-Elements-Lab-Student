# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `lane/double_white` | polyline | class | — | — | — | Vạch đôi trắng — không được đổi làn |
| `lane/double_yellow` | polyline | class | — | — | — | Vạch đôi vàng — không được vượt |
| `lane/single_white_dashed` | polyline | class | — | — | — | Vạch đơn trắng nét đứt — được đổi làn |
| `lane/single_white_solid` | polyline | class | — | — | — | Vạch đơn trắng nét liền — không được đổi làn |
| `lane/single_yellow_dashed` | polyline | class | — | — | — | Vạch đơn vàng nét đứt — phía nét đứt được vượt |
| `lane/single_yellow_solid` | polyline | class | — | — | — | Vạch đơn vàng nét liền — không được vượt |
| `lane/chevron_area` | **polygon** | class | — | — | — | Vạch xương cá — vùng cấm đi vào, là vùng chứ không phải đường nên dùng Polygon |
| `occluded` | attribute (checkbox) | attribute | `false` | `false` | false | Đánh dấu vạch bị vật khác che một phần nhưng vẫn suy luận được hình dạng thật (ví dụ BDD16) |
| `needs_review` | attribute (checkbox) | attribute | `false` | `false` | false | Escalate object khi không chắc chắn về class dù đã đọc guideline |

## Class hay attribute

Từ v2: **màu (trắng/vàng) và kiểu nét (đứt/liền) đều đưa vào class**, vì soi ảnh thật trong `data/bdd100k` cho
thấy rất nhiều ảnh có cả đoạn nét đứt lẫn nét liền của cùng một màu vạch — gộp chung sẽ mất thông tin quyết định
đổi làn (dashed = allowed, solid = not_allowed). Quyết định `allowed`/`not_allowed` vì vậy nằm cố định trong **tên
class** theo đúng luật giao thông, không cần thêm attribute `lane_change` để annotator tự chọn nữa (khác v1).
`occluded` và `needs_review` là 2 attribute duy nhất còn lại, dùng để mô tả **tình trạng nhìn thấy** của object,
không phải để quyết định class.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): TODO — điền khi mở CVAT lần đầu
- **Tên task calibration** (có version guideline, ví dụ `mixigaming-calib-v1`): TODO — điền khi tạo task
- **Guide của task đã dán `02_guideline.md`?** TODO (có / chưa)
- **Nhóm dùng Track hay Shape, vì sao:** Shape — task dùng ảnh tĩnh (`data/bdd100k`), không có video/track

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

TODO — điền sau khi tạo task calibration thật và có người test
