# Problem statement + downstream contract

## Bài toán

Phát hiện bằng bounding box các đèn tín hiệu dành cho phương tiện và phân loại trạng thái đang hiển thị trong ảnh giao
thông đô thị ban ngày từ hai góc camera `narrow`/`wide`. Các case khó gồm đầu đèn nhỏ/xa, nhiều đầu đèn cho các hướng
khác nhau, che khuất/cắt biên, chói sáng và thông tin quan sát khác nhau giữa hai camera.

## Downstream contract

1. **Downstream task / model / user:** dữ liệu dùng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đèn
   giao thông và phân loại trạng thái từ ảnh camera gắn trên xe; người dùng đầu ra là nhóm phát triển và QA mô hình.
2. **Output cần thiết:** rectangle ôm phần vỏ đèn nhìn thấy hoặc phần hiển thị số của bộ đếm; một trong 10 class
   `Red`, `Yellow`, `Green`, `Red-Yellow`, `Green-up`, `Green-left`, `Green-right`, `Empty`, `Count-down`,
   `Empty-count-down`; cùng các attribute `truncated`, `needs_review`, `note` khi áp dụng.
3. **Failure critical:** nhầm tín hiệu dừng/đi hoặc hướng di chuyển (`Red`/`Red-Yellow` với các class `Green*`), hoặc
   bỏ sót một đèn dành cho phương tiện đang sáng và nhìn thấy rõ. Các lỗi này làm sai tín hiệu mục tiêu của model.
4. **Escalation path:** annotator chọn class khả dĩ nhất, đặt `needs_review = true` và ghi bằng chứng trong `note`.
   QA owner Đào Ngọc Hiếu review; nếu là guideline gap, spec owner Nguyễn Lâm Bách chốt rule cùng nhóm và cập nhật
   version trong `02_guideline.md` và `08_revision_log.md`.

## Scope

- **Trong scope:** đèn tín hiệu dành cho xe dạng dọc/ngang; đèn mũi tên; đèn tắt nhưng còn nhìn thấy mặt bóng; bộ
  đếm ngược; đèn của hướng/làn khác nếu nhìn thấy rõ. Một vỏ đèn là một object; bộ đếm là object riêng.
- **Ngoài scope:** đèn người đi bộ; mặt sau/hông không thấy mặt bóng; biển báo; cột/cần/giá đỡ/dây; đèn đường, đèn
  xe, đèn công trường/cảnh báo; phản chiếu; đèn trong quảng cáo, màn hình hoặc ảnh khác.
- **Geometry tolerance:** box ôm phần vỏ thực sự nhìn thấy, gồm phần che nắng thuộc thân đèn nhưng không gồm bộ phận
  hỗ trợ; box bị cắt dừng tại mép ảnh. So với gold, mỗi cạnh được lệch tối đa 2 px; vượt ngưỡng hoặc bao nhầm cột,
  cần, giá đỡ hay biển là không đạt geometry.

## Output chấm được

Blind test chấm được LABEL/IGNORE, class, số object và geometry từ rectangle trong export. UNKNOWN/ESCALATE được
chấm qua `needs_review = true` và nội dung `note`; `truncated` dùng để kiểm case cắt biên. Trước handoff phải thêm ba
attribute này vào `03_cvat_labels.json`, vì schema hiện tại vẫn để `attributes: []`. Ego relevance chưa được định
nghĩa trong guideline/schema nên không thuộc output của scope hiện tại.

## Dữ liệu và giới hạn

`data/images` có 30 ảnh JPEG 1920×1080, gồm 15 cặp cùng thời điểm (`narrow_*`, `wide_*`). Dữ liệu có đèn tròn, mũi
tên, bộ đếm ngược, nhiều đầu đèn và object nhỏ/xa; mỗi ảnh phải annotate độc lập. Các ảnh đã xem đều là cảnh ban ngày
và chưa có thời tiết xấu, nên chưa đánh giá được khả năng áp dụng guideline cho đêm, mưa, tuyết hoặc sương mù.
