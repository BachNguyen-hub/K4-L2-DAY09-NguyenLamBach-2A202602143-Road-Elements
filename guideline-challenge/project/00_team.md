# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** `team07`
- **Nhóm peer test bài của mình:** TODO (cặp A ↔ B; số nhóm lẻ thì ring 3 nhóm A → B → C → A — Lab Coach công bố)
- **Nhóm mình test bài của:** TODO
- **Problem family:** Traffic-light state + ego relevance tại giao lộ nhiều đầu đèn
- **Nguồn ảnh:** `images`

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Nguyễn Lâm Bách | https://github.com/BachNguyen-hub | Điều phối & Spec owner — chốt phạm vi bài toán, downstream contract và guideline v1–v3 | `00_team.md`, `01_problem_statement.md`, `02_guideline.md` |
| Âu Xuân Mạnh | https://github.com/ManhAu1111 | CVAT owner — thiết kế ontology/schema, chuẩn bị bộ ảnh, kiểm thử setup và lưu tham chiếu task/export | `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json`, `sample_pack.csv`, `09_cvat_export_or_task_reference.txt` |
| Nguyễn Hữu Huy | https://github.com/hynu15 | Gold & Calibration owner — xây edge case, chốt gold, điều phối calibration và tổng hợp lịch sử thay đổi | `04_edge_cases/edge_case_cards.md`, `04_edge_cases/gold_decisions.csv`, `06_calibration_report.csv`, `08_revision_log.md` |
| Đào Ngọc Hiếu | Chưa cung cấp | QA & Blind-test owner — thiết kế QA, tổ chức handoff, chấm GTS và tổng hợp phản hồi peer | `05_qa_plan.md`, `07_blind_handoff/clarification_log.csv`, `07_blind_handoff/transfer_score.csv`, `07_blind_handoff/gts_summary.md`, `07_blind_handoff/peer_feedback.md` |

Gợi ý chia vai (nhóm 2–3 người thì gộp): **spec owner** (`01`, `02`), **CVAT owner** (`03_*`, `sample_pack.csv`,
`09`), **gold owner** (`04_edge_cases/`), **QA owner** (`05`, `06`, `07_blind_handoff/`). Mỗi file một người sửa
chính để tránh xung đột git. Calibration thì mọi người cùng label.

## Phần việc chung và nguyên tắc phối hợp

- Cả 4 thành viên cùng research, góp ý ontology/guideline và review chéo trước mỗi gate; người phụ trách file là người tổng hợp và commit.
- Cả 4 thành viên tự tạo task và label độc lập ở vòng calibration, sau đó gửi export cho Nguyễn Hữu Huy tổng hợp.
- Nguyễn Hữu Huy là người duy nhất chạy `make freeze`; các thành viên còn lại review gold trước khi freeze nhưng không sửa gold sau freeze.
- Trong blind window, Đào Ngọc Hiếu điều phối quy trình và ghi log; các thành viên không giải thích rule bằng miệng cho peer.
- Khi chấm blind test, cả nhóm cùng kiểm bằng chứng và phân tích nguyên nhân; Đào Ngọc Hiếu tổng hợp kết quả, Nguyễn Lâm Bách cập nhật guideline v3, Nguyễn Hữu Huy ghi revision log.
- Âu Xuân Mạnh hỗ trợ kỹ thuật CVAT/pack/import/export xuyên suốt; mỗi thành viên vẫn chịu trách nhiệm vận hành task CVAT của chính mình.
