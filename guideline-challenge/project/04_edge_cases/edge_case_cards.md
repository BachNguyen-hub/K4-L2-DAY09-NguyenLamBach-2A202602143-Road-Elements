# Edge-case library

Kho nội bộ của nhóm, **không gửi cho peer**. Mọi card dưới đây dựng từ bất đồng **đo được** giữa hai annotator
(`06_calibration_measure.csv` cho ảnh calibration, `07_blind_handoff/transfer_score.csv` cho ảnh blind), không phải
từ phỏng đoán. Số đo là số object và toạ độ box trong file export.

Quy ước: **A1** = annotator 1 (`gt.zip`), **A2** = annotator 2 (`manh.zip`).

> Hạn chế đã biết: `Observation` viết từ dữ liệu annotation (class, số object, kích thước box px), không phải từ mô tả
> thị giác của ảnh. Card về ảnh calibration đáng lẽ phải chép rule + ví dụ sang `02_guideline.md` mục 7 và 9 cho peer
> đọc được, nhưng file đó **đã freeze** nên việc này hoãn sang v4 (xem `08_revision_log.md`).

---

CASE ID: EC-01
Sample: wide_t2_080 (calibration)
Scene: Góc camera wide, ít object.
Observation: A1 vẽ 0 box `Empty`, A2 vẽ 4. Đây là điểm lệch lớn nhất toàn bộ split calibration.
Decision: IGNORE — không vẽ khi không tách được ranh giới bóng đèn trong vỏ.
Expected: 0 object `Empty`; vật chỉ hiện ra như khối tối không tách được bóng thì không phải object trong scope.
Rationale: `01_problem_statement.md` giới hạn scope ở "đèn còn nhìn thấy mặt bóng". Vẽ thừa `Empty` tạo nhãn rác cho model.
Common mistake: Dùng `Empty` như thùng rác cho mọi vật nghi là đèn nhưng không đọc được trạng thái.
Diversity: ambiguity

---

CASE ID: EC-02
Sample: wide_t1_106 (calibration)
Scene: Đèn ở xa trong khung wide.
Observation: A1 vẽ 0 `Empty` và 0 `Empty-count-down`; A2 vẽ 2 và 2.
Decision: ESCALATE — guideline hiện không đủ để quyết định, cần rule mới.
Expected: Rule phải nêu dấu hiệu nhận ra bộ đếm ngược khi nó không hiển thị số; chưa có rule thì không vẽ.
Rationale: Mục 4 định nghĩa `Empty-count-down` là "bộ đếm không hiển thị số" nhưng không nói nhận ra bằng gì khi nó tối và ở xa.
Common mistake: Suy ra "đây là bộ đếm" từ vị trí cạnh đèn thay vì từ đặc điểm nhìn thấy được.
Diversity: small_far, ambiguity

---

CASE ID: EC-03
Sample: wide_t1_284 (calibration)
Scene: Nhiều đầu đèn cho các hướng khác nhau, khung wide.
Observation: A1 vẽ 0 `Count-down` và 0 `Green`; A2 vẽ 2 và 2. Đây là ảnh calibration duy nhất lệch cả ở class đang sáng.
Decision: LABEL — nhưng ngưỡng "đủ rõ để vẽ" phải được định nghĩa trước.
Expected: Thống nhất một ngưỡng nhìn thấy được, rồi áp cho cả `Count-down` và `Green`.
Rationale: Lệch ở class đang sáng nguy hiểm hơn lệch ở đèn tắt, vì nó đổi tín hiệu dừng/đi mà model học.
Common mistake: Mỗi người tự đặt một ngưỡng "đủ rõ" trong đầu rồi không ghi ra.
Diversity: small_far, conflict

---

CASE ID: EC-04
Sample: narrow_t1_234 (calibration)
Scene: Đèn bị phương tiện che khuất.
Observation: Hai annotator ra cùng số object. Nhưng cả hai **đều không gán được** `needs_review`: `03_cvat_labels.json` để `attributes: []`.
Decision: ESCALATE — và escalation path hiện không thực hiện được.
Expected: `needs_review = true` + `note` ghi bằng chứng. Hiện không tồn tại trong schema nên không chấm được.
Rationale: Mục 7–8 guideline bảo dùng `needs_review`; `01_problem_statement.md` mục "Output chấm được" cũng ghi phải thêm 3 attribute trước handoff. Chưa làm.
Common mistake: Tin rằng "đồng thuận attribute 100%" từ `make calib` nghĩa là tốt — thực chất không có gì để đo.
Diversity: occlusion, escalation

---

CASE ID: EC-05
Sample: wide_t2_376 (blind)
Scene: Nhiều đầu đèn đỏ và xanh thuộc các hướng khác nhau.
Observation: Gold 3 `Red-Yellow`; A2 chỉ 2, đọc 1 cái thành `Red`. Hai trong ba box chỉ rộng 7–8px.
Decision: LABEL — `Red-Yellow` khi đỏ và vàng cùng sáng.
Expected: `label=Red-Yellow x3` (gold `wide_t2_376` d1).
Rationale: `01_problem_statement.md` mục 3 xếp nhầm `Red`/`Red-Yellow` với `Green*` vào critical failure.
Common mistake: Đúng cái mục 10 đã cảnh báo — "nhầm `Red-Yellow` thành `Red`". Cảnh báo bằng chữ không chặn được lỗi; cần ví dụ hình.
Diversity: critical, small_far

---

CASE ID: EC-06
Sample: narrow_t2_382 (blind)
Scene: Một đèn đỏ lớn gần và một đèn nhỏ ở xa.
Observation: Gold `Green` tại x[600-613] y[367-402], box 13×35px. A2 vẽ box **gần như trùng khít** x[600-614] y[367-402] nhưng đặt class `Empty`. Không phải bỏ sót — là bất đồng class.
Decision: LABEL — `Green`.
Expected: `label=Green x1` (gold `narrow_t2_382` d2).
Rationale: Bỏ sót một đèn đang sáng và nhìn thấy rõ là critical theo downstream contract của nhóm.
Common mistake: Ở đèn nhỏ, coi bóng sáng mờ là "không có bóng nào sáng" rồi hạ xuống `Empty`.
Diversity: critical, small_far, ambiguity

---

CASE ID: EC-07
Sample: wide_t2_454 (blind)
Scene: Ba đầu đèn vàng bên phải khung.
Observation: Gold 3 `Yellow` tại x673, x902, x1033. A2 cũng có `Yellow`×3 nên **đếm khớp**, nhưng chỉ 1 box trùng vị trí: x903 A2 đọc thành `Red-Yellow`, x1035 thành `Empty`, và A2 thêm 2 box `Yellow` nhỏ 8×17px, 6×17px ở chỗ gold không có.
Decision: LABEL — nhưng gold viết theo số lượng nên không bắt được sai lệch vị trí.
Expected: Gold hiện là `label=Yellow x3`; dạng đúng phải là `label=Yellow x3 tại 3 đầu đèn bên phải`.
Rationale: Đây là **lỗi thiết kế gold**, không phải lỗi người label. Gold đã freeze nên vẫn chấm theo chữ của nó và ghi `gold sai:` trong note.
Common mistake: Viết gold bằng số đếm rồi tưởng đã chấm được vị trí.
Diversity: conflict

---

CASE ID: EC-08
Sample: wide_t2_121 (blind)
Scene: Sáu vỏ đèn xanh tròn nhìn rõ.
Observation: Ghép 6/6 box theo IoU 0.84–0.96 — class và số lượng khớp hoàn toàn. Nhưng 2 box lệch cạnh 4.4px và 7.7px so với gold.
Decision: LABEL đúng, geometry không đạt.
Expected: `geometry:` mỗi cạnh lệch tối đa 2px (gold `wide_t2_121` d2).
Rationale: Ngưỡng 2px chỉ nằm trong `01_problem_statement.md`, **không có trong `02_guideline.md`** — peer đọc guideline thì không biết ngưỡng này tồn tại.
Common mistake: Vẽ "ôm sát bằng mắt" mà không có số cụ thể để tự kiểm.
Diversity: geometry

---

CASE ID: EC-09
Sample: narrow_t2_146 (blind)
Scene: Đèn xanh và bộ đếm ngược nhìn tương đối rõ.
Observation: `Green`×4 và `Count-down`×1 khớp hoàn toàn. Chỉ `Empty` lệch: gold 1, A2 vẽ 3.
Decision: LABEL — 1 `Empty`.
Expected: `label=Empty x1` (gold `narrow_t2_146` d3).
Rationale: Cùng root cause với EC-01 và EC-02. Đây là ảnh gắn tag `normal` mà vẫn lệch, chứng tỏ lỗ hổng `Empty` không chỉ xuất hiện ở case khó.
Common mistake: Thấy ảnh dễ thì bỏ qua checklist mục 11.
Diversity: ambiguity
