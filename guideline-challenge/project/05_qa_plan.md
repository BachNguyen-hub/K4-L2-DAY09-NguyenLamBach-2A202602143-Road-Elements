# QA plan + quality gates

Threshold dưới đây là **đề xuất của nhóm**, không phải chuẩn ngành. Chúng được neo vào số đo thật của buổi này
(đồng thuận count calibration 87.5%, GTS 70.3, D 70%, C 66.7%, G 50%) chứ không lấy con số tròn cho đẹp.

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

- **Ai review, review bao nhiêu:** QA owner Đào Ngọc Hiếu review **100% ảnh có tag `critical` hoặc `ambiguity`**, cộng
  **30% ảnh `normal`** lấy mẫu ngẫu nhiên. Spec owner Nguyễn Lâm Bách review lại các ảnh QA owner đánh `question`.
  Annotator không tự review bài của chính mình. Lý do chọn 100% cho nhóm rủi ro: critical escape ở
  `wide_t2_376` cho thấy lỗi `Red-Yellow`→`Red` sống sót qua cả guideline có cảnh báo bằng chữ.
- **Chọn sample theo rule nào:** ưu tiên theo rủi ro, không random đều. Thứ tự: (1) ảnh có `Red`/`Red-Yellow` cùng xuất
  hiện với class `Green*` trong một ảnh; (2) ảnh có object rộng dưới 25px; (3) ảnh có từ 6 object trở lên; (4) ảnh của
  annotator mới trong 20 ảnh đầu; (5) phần còn lại random 30%. Ba nhóm đầu bám đúng ba nguồn lỗi đã đo được.
- **Issue được ghi ở đâu, đóng thế nào:** mỗi issue một dòng trong `06_calibration_report.csv` (giai đoạn calibration)
  hoặc `07_blind_handoff/transfer_score.csv` cột `note` (giai đoạn blind). Issue chỉ được đóng khi: annotator đã sửa,
  QA owner xác nhận lại trên chính ảnh đó, và nếu nguyên nhân là `guideline_gap` thì dòng tương ứng đã có trong
  `08_revision_log.md`. Không đóng issue bằng thoả thuận miệng.
- **Khi phát hiện guideline gap thì update và version ra sao:** ghi `diagnosis=guideline_gap` + `action` vào report,
  sửa `02_guideline.md`, tăng `Version` lên số nguyên kế tiếp, thêm dòng vào `08_revision_log.md` có trích sample_id
  làm bằng chứng. Nếu file đang trong FREEZE_FILES và đã nhận export của người test thì **không sửa** — dồn vào version
  sau và ghi rõ là hoãn, đúng như mục v4 trong revision log.

## Defect severity

Mapping bám theo `01_problem_statement.md` mục 3 (failure critical) chứ không theo mức độ "nhìn thấy rõ hay không".

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Sai tín hiệu dừng/đi hoặc sai hướng di chuyển: nhầm `Red`/`Red-Yellow` với bất kỳ class `Green*`, nhầm hướng giữa `Green-up`/`Green-left`/`Green-right`, hoặc bỏ sót một đèn dành cho xe đang sáng và nhìn thấy rõ. | `wide_t2_376` — 1 trong 3 `Red-Yellow` bị đọc thành `Red`. `narrow_t2_382` — `Green` 13×35px bị hạ thành `Empty`. | Reject cả batch. Annotator làm lại 100% ảnh trong batch, không chỉ ảnh bị bắt. Spec owner rà lại rule liên quan trước khi cho làm tiếp. |
| Major | Sai class không đổi tín hiệu dừng/đi, hoặc sai số object: vẽ thừa/thiếu `Empty`, `Empty-count-down`, bỏ sót `Count-down`. | `narrow_t2_146` — gold 1 `Empty`, annotator vẽ 3. `wide_t1_106` — vẽ thừa 2 `Empty` + 2 `Empty-count-down`. | Rework các ảnh bị bắt. Nếu cùng một lỗi major xuất hiện ở ≥ 20% ảnh sample thì coi là guideline gap, không phải lỗi annotator. |
| Minor | Geometry lệch quá ngưỡng nhưng class và số object đúng; box lẫn một phần giá đỡ. | `wide_t2_121` — 6/6 class đúng, 2 box lệch cạnh 4.4px và 7.7px. | Sửa tại chỗ, không reject batch. Ghi lại để theo dõi xu hướng. |
| Question | Annotator hoặc QA không quyết được, cần rule mới. | `narrow_t1_234` — đèn bị che, cần `needs_review` nhưng schema không có attribute. | Escalate cho spec owner trong 1 ngày làm việc. Không đoán rồi làm tiếp. Nếu quá hạn mà chưa có rule thì ảnh đó ra khỏi batch. |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Critical defect escape rate | số decision `severity=critical` bị sai / tổng decision critical | Downstream là model phân loại trạng thái đèn, nên một lỗi dừng/đi nặng hơn nhiều lỗi `Empty`. Đo riêng để nó không bị pha loãng trong con số tổng. |
| Inter-annotator count agreement | số (ảnh × label × phép đo) khớp / tổng, do `make calib` tính | Đo guideline có mơ hồ không mà **không cần biết ai đúng**. Dùng được ngay từ calibration, trước khi có gold. |
| Empty-class false positive rate | số object `Empty` + `Empty-count-down` vẽ thừa so với gold / tổng object `Empty*` trong gold | Tách riêng vì đây là nguồn lỗi lớn nhất đã đo: 5/5 điểm lệch calibration và 2/12 decision blind đều thuộc nhóm này. |
| Geometry compliance | số box có cả 4 cạnh lệch ≤ 2px so với gold / số box ghép được theo IoU ≥ 0.5 | Biến "ôm sát" thành số kiểm được. Không có ngưỡng thì mỗi người tự đặt một mức trong đầu. |
| Small-object recall | số object gold rộng < 25px được vẽ / tổng object gold rộng < 25px | Ba trong bốn lỗi nặng nhất của buổi này nằm ở object 7–35px. Metric tổng không lộ ra điều đó. |

Metric high-risk tách riêng: **critical defect escape rate** và **small-object recall**. Hai metric này không được gộp
vào điểm tổng, vì một batch có thể đạt tổng cao trong khi vẫn để thoát lỗi dừng/đi.

## Quality gate

```text
PASS if:
  critical defect escape rate  = 0
  AND inter-annotator count agreement >= 85%
  AND Empty-class false positive rate <= 10%
  AND geometry compliance >= 85%
  AND small-object recall >= 80%
REWORK if: critical escape = 0 nhưng bất kỳ metric còn lại rơi dưới ngưỡng
REJECT / ESCALATE if: critical defect escape rate > 0, HOẶC count agreement < 80%
```

Trade-off: đặt critical escape = 0 là ngưỡng đắt — một lỗi là reject cả batch, chi phí rework cao. Nhóm chọn vậy vì
downstream dùng nhãn để huấn luyện model quyết định dừng/đi; một nhãn `Red` bị đọc thành `Green` dạy model sai đúng
chỗ nguy hiểm nhất, và chi phí đó không nằm ở nhóm annotation mà đổ xuống hạ nguồn. Ngược lại geometry để 85% và
`Empty` false positive để 10% vì hai lỗi này làm nhiễu bounding box chứ không đảo tín hiệu, siết lên 100% sẽ đốt thời
gian mà không giảm rủi ro tương ứng.

Ngưỡng count agreement đặt **85%**, dưới mức nhóm thực đo được (87.5%), nên tiêu chí này đạt. Lý do chấp nhận được:
87.5% đo trên 5 ảnh calibration, cỡ mẫu nhỏ nên một ảnh lệch đã kéo tỷ lệ xuống vài phần trăm; đặt ngưỡng sát mức đo
được sẽ khiến gate bật/tắt theo nhiễu chứ không theo chất lượng. 85% giữ được khoảng đệm đó mà vẫn chặn được batch
tệ hẳn — dưới 80% là REJECT.

**Batch của buổi này vẫn không PASS**, và ngưỡng agreement không phải nguyên nhân. Nguyên nhân là
`critical defect escape rate = 1/3` (`wide_t2_376`, `Red-Yellow` bị đọc thành `Red`), trong khi PASS đòi escape = 0.
Nhóm giữ nguyên tiêu chí escape = 0 và **không hạ nó** để đổi lấy một chữ PASS: điều kiện này bắt nguồn trực tiếp từ
downstream contract ở `01_problem_statement.md` mục 3, hạ nó xuống thì cả phần severity mapping phía trên mất căn cứ.
Kết luận đúng của buổi này là REJECT kèm 6 việc sửa ở `08_revision_log.md`, không phải PASS.
