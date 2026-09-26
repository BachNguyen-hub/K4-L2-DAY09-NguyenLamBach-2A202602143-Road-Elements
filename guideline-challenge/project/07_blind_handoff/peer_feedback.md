# Peer feedback + owner response

> ⚠️ **Đây KHÔNG phải blind test với nhóm peer thật.** Các nhóm khác trong lớp dùng dataset khác nên không trao đổi
> được gói blind. Thay vào đó nhóm dùng **internal blind test**: nhãn độc lập của annotator 2 (Mạnh) trên đúng 5 ảnh
> blind, lấy từ cùng lần gán nhãn đã dùng cho calibration. Không có dữ liệu nào được sinh giả.
>
> Giới hạn phải đọc kèm mọi con số ở đây:
> - Mạnh **cùng nhóm**, không phải annotator của nhóm khác, nên đã tiếp xúc guideline trước đó.
> - Mạnh **không trải qua blind window**: không nhận `blind-pack.zip`, không bị chặn hỏi, nên
>   `clarification_log.csv` có 0 dòng và **I = 100 là artifact của quy trình, không phải bằng chứng guideline rõ**.
>   Đọc GTS thì bỏ thành phần I ra, hoặc coi GTS thật ≈ 60 nếu tính I = 0.
> - Vì vậy GTS dưới đây đo được **D, C, G** (guideline chuyển giao tới người thứ hai tới đâu) nhưng **không đo được
>   độ độc lập**.

- **Nhóm peer:** không có — dùng internal blind test (lý do ở trên)
- **Người label blind:** Mạnh (annotator 2 của nhóm), file `peer_output/internal-blind-manh.zip`

## 1. Peer trả lời

**Không thu được.** Nhãn của Mạnh được lấy lại từ lần gán nhãn calibration, thời điểm đó chưa có khung 5 câu này nên
không ai hỏi. Không suy diễn câu trả lời thay người label. Phần dưới đây thay thế bằng thứ đo được trực tiếp từ file
export, đối chiếu `transfer_score.csv`:

1. **Rule chuyển giao tốt nhất:** mục 6 (mũi tên) và mục 3 (bộ đếm là object riêng). `Green-right x2` và
   `Count-down x1` đều khớp, không cần giải thích thêm.
2. **Rule mơ hồ nhất:** mục 5, ngưỡng `Empty`. Gây sai ở 2/12 decision và 5/5 bất đồng calibration.
3. **Sample làm guideline vỡ:** `narrow_t2_382` — cùng một vật, cùng toạ độ box, hai người đọc thành hai class khác nhau.
4. **Attribute/default gây thao tác sai:** không đánh giá được. `03_cvat_labels.json` vẫn `attributes: []` nên
   `needs_review`, `truncated`, `note` không tồn tại để mà sai.
5. **Thay đổi giúp ít hỏi hơn:** đưa ngưỡng `Empty` thành tiêu chí nhìn thấy được, kèm ảnh ví dụ cạnh nhau.

## 2. Owner phân loại

| Feedback / decision sai | Nguyên nhân | Xử lý | Bằng chứng |
|---|---|---|---|
| `narrow_t2_146` d3 — gold 1 `Empty`, Mạnh vẽ 3 | guideline gap | accept + revise: thêm ngưỡng nhìn thấy được cho mục 5 | `transfer_score.csv` dòng 5; `06_calibration_report.csv` dòng `wide_t2_080` |
| `narrow_t2_382` d2 — box đúng toạ độ nhưng class `Empty` thay vì `Green` | guideline gap | accept + revise: cùng rule trên; thêm ví dụ đèn nhỏ 13×35px | box Mạnh x[600-614] y[367-402] vs gold x[600-613] y[367-402] |
| `wide_t2_376` d1 — **critical escape**, `Red-Yellow` bị đọc thành `Red` | guideline gap | accept + revise: mục 10 đã cảnh báo bằng chữ nhưng vẫn sai → phải có ví dụ hình, không chỉ gạch đầu dòng | `transfer_score.csv` dòng 6; box chỉ 7–8px ngang |
| `wide_t2_121` d2 — 2/6 box lệch 4.4px và 7.7px | guideline gap | accept + revise: mục 3 nói "ôm sát" nhưng không nêu số; đưa ngưỡng 2px vào guideline vì hiện chỉ có trong `01_problem_statement.md` | IoU 0.84–0.96, lệch cạnh lớn nhất 7.7px |
| `wide_t2_454` d1 — đếm khớp nhưng 2/3 box sai vị trí | lỗi thiết kế gold, không phải lỗi người label | reject with evidence: giữ `correct=1` theo đúng chữ của gold đã freeze, ghi `gold sai:` trong note; sửa dạng viết gold ở v4 | gold `label=Yellow x3`; Mạnh có Yellow×3 nhưng chỉ 1 trùng toạ độ, thêm 2 box 8×17px và 6×17px |
| `needs_review` / `truncated` / `note` không tồn tại trong schema | guideline gap | add escalation rule: thêm 3 attribute vào ontology + labels JSON, rồi `make freeze REFREEZE=1` | `03_cvat_labels.json` `attributes: []`; `01_problem_statement.md` mục "Output chấm được" đã tự ghi phải thêm trước handoff |
