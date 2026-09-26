# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v2 | Tăng số version. **Nội dung rule không đổi** — `02_guidelinev1.md` và `02_guidelinev2.md` giống nhau từng byte. Bốn rule change do calibration chỉ ra đã được ghi thành đề xuất nhưng chưa viết vào guideline. | Calibration đo được 5 điểm lệch, đồng thuận count 87.5%, tất cả lệch một chiều (A1=0, A2>0). Bốn hướng sửa: (1) ngưỡng nhìn thấy được cho `Empty`, (2) dấu hiệu nhận ra `Empty-count-down` khi tối, (3) ví dụ chuẩn cho cảnh nhiều đầu đèn ở xa, (4) thêm 3 attribute vào schema. | `06_calibration_report.csv` 4 dòng; `06_calibration_measure.csv` |
| v3 | Tăng số version trước khi freeze gold. **Nội dung rule vẫn không đổi** so với v2. | Cần version ≥ v2 để `make freeze` chạy, và ≥ v3 cho gate G6. Gold 12 decision được freeze trên bản v3 này (`FREEZE.txt` `guideline_version=3`). | `project/FREEZE.txt`; tag git `gold-freeze` |
| v4 (hoãn — **chưa thực hiện**) | Bốn rule change của v2 cộng bốn phát hiện mới từ blind test. Danh sách cụ thể ở dưới. | `02_guideline.md` và `03_cvat_labels.json` nằm trong FREEZE_FILES. `peer_output/` đã có export nên `make freeze REFREEZE=1` bị tool từ chối. Sửa lúc này sẽ làm G4 đỏ không phục hồi được. Tool chỉ dẫn đúng tình huống này: giữ gold, ghi `gold sai:` vào note, dồn sửa vào version sau. | `lab9/freeze.py` điều kiện chặn refreeze; `transfer_score.csv` note dòng `wide_t2_454` d1 |

## Việc phải làm ở v4

Xếp theo mức nghiêm trọng đo được, không theo cảm nhận.

1. **Ngưỡng nhìn thấy được cho `Empty`** — gây sai nhiều nhất: 5/5 điểm lệch calibration và 2/12 decision blind
   (`narrow_t2_146` d3, `narrow_t2_382` d2). Sửa mục 5: chỉ vẽ khi tách được ranh giới ít nhất một bóng đèn trong vỏ;
   không dùng `Empty` thay cho "không đọc được".
2. **Ba attribute `needs_review` / `truncated` / `note` vào `03_cvat_labels.json` + ontology table** — guideline mục 6,
   7, 8, 10, 11 đều yêu cầu gán, nhưng schema để `attributes: []` nên không ai gán được. `01_problem_statement.md` mục
   "Output chấm được" đã tự ghi phải thêm trước handoff. Escalation path hiện là rule chết.
3. **Đưa ngưỡng geometry 2px vào `02_guideline.md`** — hiện chỉ nằm trong `01_problem_statement.md`, là file **không
   gửi cho peer**. Hệ quả đo được: `wide_t2_121` d2 sai, 2/6 box lệch 4.4px và 7.7px dù class và số lượng khớp hoàn toàn.
4. **Ví dụ hình cho `Red-Yellow` vs `Red`** — mục 10 đã cảnh báo bằng chữ mà vẫn xảy ra critical escape ở
   `wide_t2_376` d1 (box chỉ 7–8px ngang). Cảnh báo dạng gạch đầu dòng không đủ.
5. **Dấu hiệu nhận ra `Empty-count-down` khi không hiển thị số** — `wide_t1_106`, A1 vẽ 0 / A2 vẽ 2.
6. **Chép rule + ví dụ từ EC-01…EC-04 sang `02_guideline.md` mục 7 và 9** — card về ảnh calibration hiện chỉ nằm ở
   kho nội bộ, peer không đọc được.

## Việc phải làm ở gold v2 (lần freeze sau)

- Viết `expected` kèm định vị, không chỉ số đếm. Bằng chứng: `wide_t2_454` d1 — gold `label=Yellow x3` được thoả bằng
  số đếm trong khi chỉ 1/3 box trùng vị trí, 2 box gold còn lại bị đọc thành `Red-Yellow` và `Empty`, cộng 2 box thừa
  8×17px và 6×17px. Đây là lỗi thiết kế gold, đã ghi `gold sai:` trong `transfer_score.csv`.
- Soi lại `narrow_t2_382` d2 ở kích thước gốc. Box 13×35px, hai annotator vẽ trùng khít toạ độ nhưng một người đọc
  `Green`, người kia `Empty`. Nếu A1 sai thì gold sai.
