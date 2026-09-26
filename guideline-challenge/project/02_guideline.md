# Annotation Guideline — Traffic Light Detection
**Version:** v3
## 1. Mục tiêu

Gán nhãn **đèn tín hiệu giao thông dành cho phương tiện** và **trạng thái đang hiển thị** của chúng.

Mỗi ảnh được gán nhãn độc lập.

---

## 2. Đối tượng cần gán nhãn

### Vẽ box

Gán nhãn cho:

- Đèn tín hiệu dành cho xe, dạng **dọc** hoặc **ngang**.
- Đèn tín hiệu dạng **mũi tên**.
- Đèn tín hiệu đang tắt nhưng vẫn nhìn thấy mặt bóng.
- **Bộ đếm ngược** cạnh đèn giao thông.
- Đèn của các hướng/làn khác nếu vẫn nhìn thấy rõ trong ảnh.

### Không vẽ box

Không gán nhãn cho:

- Đèn người đi bộ.
- Mặt sau hoặc mặt hông của đèn khi không nhìn thấy mặt bóng.
- Biển báo và biển chỉ hướng.
- Cột, cần vươn, giá đỡ, dây treo.
- Đèn đường, đèn xe, đèn công trường hoặc đèn cảnh báo.
- Phản chiếu của đèn trên kính, xe hoặc mặt đường.
- Đèn xuất hiện trong biển quảng cáo, màn hình hoặc hình ảnh khác.

---

## 3. Quy tắc vẽ bounding box

### Đơn vị annotation

**Một box = một vỏ đèn giao thông (housing).**

Không vẽ riêng từng bóng đỏ, vàng, xanh trong cùng một vỏ đèn.

Ví dụ:

- Một vỏ có 3 bóng, hiện đang sáng đỏ → **1 box**, class `Red`.
- Một cụm có vỏ đèn và bộ đếm ngược → vẽ **2 box riêng**.

### Cách vẽ box

- Dùng **Rectangle / Shape** trong CVAT.
- Box ôm sát **toàn bộ phần vỏ đèn nhìn thấy được**.
- Bao gồm cả các bóng đang tắt nằm trong cùng vỏ.
- Bao gồm phần che nắng nếu thuộc thân đèn.
- Không bao gồm cột, cần vươn, dây, giá đỡ hoặc biển báo.
- Không để khoảng trống dư quanh object.

### Bộ đếm ngược

Bộ đếm ngược là một object riêng.

Box chỉ bao quanh **phần hiển thị số**, không bao gồm giá treo hoặc bộ phận hỗ trợ.

---

## 4. Labels

Sử dụng chính xác các label sau:

| Label | Khi sử dụng |
|---|---|
| `Red` | Đèn đỏ tròn đang sáng |
| `Yellow` | Đèn vàng tròn đang sáng |
| `Green` | Đèn xanh tròn đang sáng |
| `Red-Yellow` | Đỏ và vàng cùng sáng |
| `Green-up` | Đèn xanh hình mũi tên ↑ |
| `Green-left` | Đèn xanh hình mũi tên ← |
| `Green-right` | Đèn xanh hình mũi tên → |
| `Empty` | Thấy mặt đèn nhưng không có bóng nào sáng |
| `Count-down` | Bộ đếm ngược đang hiển thị số |
| `Empty-count-down` | Bộ đếm ngược không hiển thị số |

---

## 5. Cách chọn class

Thực hiện theo thứ tự:

### Bộ đếm ngược

- Có hiển thị số → `Count-down`
- Không hiển thị số → `Empty-count-down`

### Đèn giao thông

Nếu không nhìn thấy mặt bóng → **không vẽ**.

Nếu nhìn thấy mặt bóng:

- Không có bóng sáng → `Empty`
- Đỏ + vàng cùng sáng → `Red-Yellow`
- Đỏ sáng → `Red`
- Vàng sáng → `Yellow`
- Xanh tròn → `Green`
- Xanh mũi tên ↑ → `Green-up`
- Xanh mũi tên ← → `Green-left`
- Xanh mũi tên → → `Green-right`

**Không xác định trạng thái dựa vào vị trí của bóng trong vỏ đèn.**  
Luôn dựa vào **màu và hình dạng của bóng đang sáng**.

---

## 6. Quy tắc đối với mũi tên

Hướng mũi tên được xác định theo **hình mũi tên nhìn thấy trong ảnh**.

- ↑ → `Green-up`
- ← → `Green-left`
- → → `Green-right`
- ↖ → `Green-left`
- ↗ → `Green-right`

Nếu hướng mũi tên không xác định rõ → đánh dấu `needs_review`.

---

## 7. Đèn bị che hoặc bị cắt

### Bị che

Nếu vẫn nhìn thấy bóng đang sáng và xác định được trạng thái:

- Vẫn vẽ box.
- Box chỉ bao phần vỏ **thực sự nhìn thấy**.
- Không tự đoán phần bị che.

Nếu bị che quá nhiều và không đủ thông tin để xác định object/trạng thái → **không vẽ**.

### Bị cắt bởi mép ảnh

- Box dừng tại mép ảnh.
- Không kéo box ra ngoài ảnh.
- Không tự đoán phần nằm ngoài ảnh.
- Đánh dấu:

`truncated = true`

Nếu phần còn nhìn thấy đủ để xác định trạng thái → chọn class tương ứng.

---

## 8. Ảnh mờ, loá hoặc khó xác định

Nếu vẫn xác định được màu/hình của bóng → gán class bình thường.

Nếu không chắc chắn:

- Vẫn vẽ box nếu xác định được đó là đèn giao thông.
- Chọn class khả dĩ nhất.
- Đánh dấu:

`needs_review = true`

- Ghi lý do vào `note`.

Các trường hợp thường cần review:

- Không phân biệt được đỏ/vàng.
- Không rõ object là đèn hay biển báo.
- Không rõ hướng mũi tên.
- Không chắc là đèn xe hay đèn người đi bộ.
- Vỏ đèn có cấu trúc bất thường.

---

## 9. Quy tắc ảnh narrow / wide

Ảnh `narrow_*` và `wide_*` có thể là cùng một cảnh từ hai camera khác nhau.

**Phải gán nhãn từng ảnh độc lập.**

Không copy bounding box hoặc class từ ảnh camera kia nếu thông tin không nhìn thấy rõ trong ảnh đang annotate.

---

## 10. Các lỗi cần tránh

- Chỉ box bóng đang sáng thay vì toàn bộ vỏ đèn.
- Tách từng bóng trong cùng một vỏ thành nhiều box.
- Box lẫn cột, cần vươn hoặc biển báo.
- Gán nhãn đèn người đi bộ.
- Gán nhãn mặt sau của đèn.
- Đoán trạng thái dựa vào vị trí bóng.
- Nhầm `Red-Yellow` thành `Red`.
- Nhầm `Green` với các class mũi tên.
- Quên annotate bộ đếm ngược.
- Quên `Empty-count-down`.
- Vẽ box ra ngoài mép ảnh.
- Copy annotation giữa ảnh `narrow` và `wide`.

---

## 11. Checklist trước khi hoàn thành ảnh

Kiểm tra:

1. Đã kiểm tra toàn bộ ảnh và các đèn ở xa chưa?
2. Mỗi vỏ đèn chỉ có **1 box** chưa?
3. Bộ đếm ngược đã được vẽ box riêng chưa?
4. Box có lẫn cột, cần hoặc biển báo không?
5. Đỏ + vàng cùng sáng đã dùng `Red-Yellow` chưa?
6. Object bị cắt mép đã đánh dấu `truncated` chưa?
7. Có gán nhầm đèn người đi bộ hoặc mặt sau đèn không?
8. Các trường hợp không chắc chắn đã đánh dấu `needs_review` chưa?

---

## Quick Reference

```text
VẼ
- Đèn dành cho xe nhìn thấy mặt bóng
- Đèn mũi tên
- Đèn đang tắt
- Bộ đếm ngược

KHÔNG VẼ
- Đèn người đi bộ
- Mặt sau/mặt hông
- Biển báo
- Cột/cần/giá đỡ
- Phản chiếu

BOX
- 1 vỏ đèn = 1 box
- Ôm toàn bộ phần vỏ nhìn thấy
- Không box riêng từng bóng
- Không bao cột/cần/biển
- Bị cắt ảnh → dừng tại mép ảnh

CLASS
Red
Yellow
Green
Red-Yellow
Green-up
Green-left
Green-right
Empty
Count-down
Empty-count-down

KHÔNG CHẮC
→ needs_review = true
→ ghi note
```