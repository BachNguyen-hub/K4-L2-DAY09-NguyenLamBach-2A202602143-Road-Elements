# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `Red` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có đèn đỏ tròn đang sáng. |
| `Yellow` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có đèn vàng tròn đang sáng. |
| `Green` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có đèn xanh tròn đang sáng. |
| `Red-Yellow` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có đèn đỏ và vàng cùng sáng. |
| `Green-up` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có mũi tên xanh hướng lên; mũi tên được xác định theo hình nhìn thấy trong ảnh. |
| `Green-left` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có mũi tên xanh hướng trái hoặc chếch trái; mũi tên được xác định theo hình nhìn thấy trong ảnh. |
| `Green-right` | Rectangle | class | N/A | N/A | N/A | Một vỏ đèn có mũi tên xanh hướng phải hoặc chếch phải; mũi tên được xác định theo hình nhìn thấy trong ảnh. |
| `Empty` | Rectangle | class | N/A | N/A | N/A | Nhìn thấy mặt đèn nhưng không có bóng nào sáng. |
| `Count-down` | Rectangle | class | N/A | N/A | N/A | Bộ đếm ngược đang hiển thị số; là object riêng, box chỉ ôm phần hiển thị số. |
| `Empty-count-down` | Rectangle | class | N/A | N/A | N/A | Bộ đếm ngược không hiển thị số; là object riêng, box chỉ ôm phần hiển thị số. |

Các dòng trên phản ánh đúng 10 label đang có trong `03_cvat_labels.json`. Guideline còn yêu cầu ba attribute dưới đây,
nhưng chúng **chưa có trong schema JSON**, nên chưa thể coi là cấu hình CVAT hoàn chỉnh:

| Name | Geometry | Type | Allowed values theo guideline | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `truncated` | Gắn với rectangle | attribute (checkbox) | `true`, `false` | `false` | Không | Ghi nhận object bị cắt tại biên ảnh; box phải dừng ở mép ảnh. |
| `needs_review` | Gắn với rectangle | attribute (checkbox) | `true`, `false` | `false` | Không | Đánh dấu trường hợp cần review thay vì che giấu sự không chắc chắn. |
| `note` | Gắn với rectangle | attribute (text) | Văn bản tự do; bắt buộc có nội dung khi `needs_review = true` | Chuỗi rỗng | Không | Lưu bằng chứng/lý do để QA owner xử lý trường hợp cần review. |

## Class hay attribute

Downstream cần trực tiếp huấn luyện/đánh giá cả detection và state classification, vì vậy trạng thái hiển thị và hướng
mũi tên được mã hoá thành 10 **class** loại trừ lẫn nhau. Mỗi rectangle mang đúng một class theo trạng thái nhìn thấy;
một vỏ đèn không được tách thành nhiều box theo từng bóng. Thiết kế này giúp class trong export dùng trực tiếp làm
target phân loại và giữ riêng hai loại object có geometry khác nhau: vỏ đèn và vùng hiển thị bộ đếm.

`truncated`, `needs_review` và `note` không thay đổi loại object hay trạng thái mục tiêu, nên là **attribute** của
rectangle. Hai checkbox mặc định `false` vì phần lớn object không bị cắt và không cần review; `note` mặc định rỗng và
chỉ bắt buộc khi `needs_review = true`. Default `false` có thể gây bias nếu annotator quên bật, nên self-QC phải kiểm
đặc biệt object chạm biên ảnh và các case mờ/loá/không rõ hướng. Vì mỗi ảnh được annotate độc lập bằng Shape, các
attribute này không mutable theo frame.

**Điểm chưa khớp cần xử lý:** `03_cvat_labels.json` hiện để `"attributes": []` cho cả 10 class. Do đó annotator chưa
thể lưu `truncated`, `needs_review` hoặc `note` theo đúng guideline trong file export CVAT.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): 2.74.1 (đã kiểm tra tại `http://localhost:8080`)
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `team07-calib-v1` — **Task ID:** `01`
- **Guide của task đã dán `02_guideline.md`?** Có theo thông tin đã ghi trong file; chưa có ảnh chụp/export hoặc kết
  quả setup test để kiểm chứng trực tiếp nội dung trên Task ID `01`. Guideline trong repo hiện là `v3`, còn tên task calibration giữ `v1` vì đây là task
  dùng ở vòng calibration trước các lần revision.
- **Nhóm dùng Track hay Shape, vì sao:** Shape. Guideline yêu cầu dùng `Rectangle / Shape` và gán nhãn từng ảnh độc
  lập, kể cả cặp ảnh `narrow_*` và `wide_*`; không copy annotation giữa hai ảnh.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

**Người setup test:** Nguyễn Hữu Huy — thành viên không trực tiếp cấu hình task.

**Kịch bản mô phỏng, cần Nguyễn Hữu Huy thực hiện và xác nhận trước khi coi là bằng chứng thật:** Huy mở Task ID `01`
chỉ với schema và Guide trong CVAT, không nhận giải thích miệng từ CVAT owner. Huy thử annotate
`narrow_t2_259.jpg`, trong đó có đèn đỏ và bộ đếm ngược, rồi kiểm tra một ảnh mũi tên như
`narrow_t2_220.jpg`. Các câu hỏi cần trả lời và kết quả kỳ vọng:

| Nội dung kiểm tra | Kết quả kỳ vọng từ Guide |
|---|---|
| Dùng công cụ nào? | Rectangle/Shape. |
| Một vỏ đèn và bộ đếm được vẽ thế nào? | Hai box riêng; box đèn ôm phần vỏ nhìn thấy, box bộ đếm chỉ ôm vùng hiển thị số. |
| Chọn label theo gì? | Theo màu/hình đang sáng; không suy đoán theo vị trí bóng. |
| Xử lý mũi tên trái? | Chọn `Green-left`; nếu hướng không rõ thì bật `needs_review` và ghi `note`. |
| Khi nào escalate? | Khi không chắc object, màu hoặc hướng; chọn class khả dĩ nhất, bật `needs_review`, ghi lý do để Đào Ngọc Hiếu review. |
| Có copy từ ảnh narrow sang wide không? | Không; hai ảnh phải annotate độc lập. |

**Kết quả hiện tại của kịch bản:** `REWORK`, chưa được ghi là PASS. Huy có thể hiểu 10 class và quy tắc geometry từ
Guide, nhưng Task ID `01` chưa thể cho nhập `truncated`, `needs_review` và `note` vì `03_cvat_labels.json` đang để
`"attributes": []`. Sau khi CVAT owner cập nhật schema, Huy cần chạy lại kịch bản và ghi kết quả thực tế, thời điểm
test cùng chỗ đã vấp.
