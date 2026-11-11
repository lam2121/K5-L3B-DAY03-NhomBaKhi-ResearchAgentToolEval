# Báo cáo cá nhân — Nguyễn Duy Hưng (MSSV: 2A202604110)

## 1. Thông tin thành viên

- Họ tên: Nguyễn Duy Hưng
- MSSV: 2A202604110
- Email: <điền email của Hưng>
- GitHub: Hungnd1208
- Lớp: K5 - L3B
- Nhóm: Nhóm Bá Khí (`NhomBaKhi`)

## 2. Phần việc phụ trách

- **Checkpoint 1:** Chạy baseline v0 bằng provider thật, lưu run hợp lệ và phân tích chi tiết 7 ca lỗi ban đầu (`wrong_tool`, `missing_info`, `wrong_boundary`, `wrong_arg_value`).
- **Checkpoint 2:** Vòng lặp tối ưu dựa trên bằng chứng từ v1 → v3 thông qua tinh chỉnh `artifacts/system_prompt.md` và `artifacts/tools.yaml`; quản lý và chuẩn hóa `artifacts/version_log.csv` định dạng UTF-8 Tiếng Việt.
- **Checkpoint 3:** Đồng bộ khai báo 3 tool mới (`topic_radar`, `evidence_finder`, `source_deduplicator`) vào `tools/__init__.py` và `artifacts/tools.yaml`; hoàn thiện unit tests cho các tool mới trong `tests/test_tools_todo.py` (tại mốc CP3 đưa toàn suite `pytest` đạt **134 passed, 0 failed, 0 skipped**; sau khi nhóm tích hợp đủ 4 tool và đồng bộ `main`, toàn suite đạt **159 passed, 0 failed, 0 skipped**).
- **Checkpoint 4:** Kiểm tra, chạy và chuẩn hóa bộ 10 eval cases nhóm (`data/eval_group.json`) cùng `system_prompt.md` để khắc phục các false failures từ tham số mặc định, nâng kết quả `--suite group` từ 50% (5/10) lên 90% (9/10) ở mốc `ac3e062` và **100% (10/10)** ở bản kiểm chứng cuối (`8987678`); đồng thời chạy thử nghiệm local qua Ollama (`qwen2.5:0.5b` 3/10, `qwen2.5:3b` 6/10) và cập nhật `version_log.csv` (`bf2053a`).
- **Checkpoint 5:** Hoàn thiện báo cáo nhóm `report/team/group_report.md`, `TEAM.md` và rà soát tính tuân thủ quy tắc toàn bộ dự án.

## 3. Đóng góp cụ thể

- **Baseline v0 (`65%`, 13/20 cases):** Xác định nguyên nhân lỗi từ starter prompt (cố ý ép đoán tài khoản Sam Altman/URL, tự ý gọi `send` không qua xác nhận, ép xử lý trong một bước).
- **Version v1 (`70%`, 14/20 cases):** Loại bỏ các chỉ dẫn sai lệch của starter; bổ sung luật gọi `clarify(response_type="text")` khi thiếu handle/URL và `clarify(response_type="yes_no")` khi người dùng yêu cầu gửi/đăng bài. Sửa thành công `R10`, `R11`.
- **Version v2 (`80%`, 16/20 cases):** Phân định rõ giữa tìm kiếm chủ đề (`social_search`) với thiếu tài khoản cá nhân; kích hoạt gọi công cụ song song (`lookup` + `social_search`) và chỉ định rõ `response_type: "yes_no"` cho hành động gửi. Đạt **100% Tool Routing Accuracy**.
- **Version v3 (`85%`, 17/20 cases trên OpenRouter `gpt-4o-mini`, `provider_error_cases=0`):** Viết lại toàn diện `artifacts/tools.yaml` với mô tả rõ *"Khi nào dùng / Khi nào KHÔNG dùng"* và quy ước trích xuất tham số sạch (bỏ filler words trong `query`, handle không chứa `@`, boundary xác nhận rõ ràng cho `send`).
- **Xử lý kỹ thuật provider:** Bổ sung cơ chế retry tự động khi gặp lỗi rate limit `429 RESOURCE_EXHAUSTED` / `503 UNAVAILABLE` trong `providers/gemini_provider.py`, và kiểm chứng trên OpenRouter để đảm bảo các run nộp bài đều có `provider_error_cases == 0`.
- **Unit Test & Contract Sync:** Cập nhật `tools/__init__.py` và viết đầy đủ các test offline (dùng `fake_http` & `monkeypatch`) cho `topic_radar`, `evidence_finder`, `source_deduplicator` (happy path, bad input, failure path trả error dict) tại `tests/test_tools_todo.py`. Tại mốc CP3 toàn suite đạt **134/134 passed, 0 skipped**; sau khi nhóm hoàn thiện tích hợp thêm `rss_feed_reader` và bổ sung test, toàn suite trên `main` đạt **159/159 passed, 0 skipped**.
- **Tối ưu Eval Group (`50%` → `90%` → `100%`):** Phát hiện lỗi false negative trong `eval_group.json` (do test case ép kiểm tra giá trị mặc định `false` cho `enrich_articles` và `verify_content` dù model không cần truyền; ép exact query ở `G05` phiên bản cũ trước khi nhóm tích hợp `rss_feed_reader`). Tinh chỉnh lại `eval_group.json` và bổ sung quy tắc định tuyến `topic_radar` trong `system_prompt.md`, giúp `--suite group` tăng từ 50% lên 90% (commit lịch sử `ac3e062`) và ở bản kiểm chứng cuối đạt **10/10 PASS (100%)** tại lần chạy `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (commit `8987678`).
- **Benchmark Local Ollama Eval (`bf2053a`):** Chạy kiểm chứng bộ `eval_group` trên Ollama local để đối chiếu năng lực tool-calling giữa các model nhỏ và OpenRouter (`qwen2.5:0.5b` đạt **3/10 - 30%**, routing `0.90`; `qwen2.5:3b` đạt **6/10 - 60%**, routing `1.00`, `provider_error_cases == 0`), đồng thời cập nhật kết quả vào `artifacts/version_log.csv`.
## 4. Bằng chứng

- **Files/Artifacts đã tạo & cập nhật:**
  - `starter_v0/artifacts/system_prompt.md`
  - `starter_v0/artifacts/tools.yaml`
  - `starter_v0/artifacts/version_log.csv`
  - `starter_v0/providers/gemini_provider.py`
  - `starter_v0/tools/__init__.py`
  - `starter_v0/tests/test_tools_todo.py`
  - `starter_v0/data/eval_group.json`
  - `TEAM.md`, `report/team/group_report.md`
- **Commits kỹ thuật trên GitHub:**
  - `463c8a9` (`prompt1`), `17f136e` (`prompt`), `7ef6de9` (`prompt v3`), `d222de1` (`cp3 cp4`), `ac3e062` (`fix eval`), `45d93ab` (`xong team, group report`), `8987678` (`final`), `bf2053a` (`chay local eval`).

  - **Run JSON hợp lệ (`provider_error_cases == 0`):**
  - `starter_v0/runs/v0_B_base_gemini_20261110T174737486284.json` (13/20 - 65%)
  - `starter_v0/runs/v1_B_base_gemini_20261110T181127878174.json` (14/20 - 70%)
  - `starter_v0/runs/v2_B_base_gemini_20261110T182200983550.json` (16/20 - 80%)
  - `starter_v0/runs/v3_B_base_openrouter_20261111T001429486980.json` (17/20 - 85%)
  - `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (10/10 - 100%)
  - `starter_v0/runs/v3-local_B_group_ollama_20261111T100206146391.json` (3/10 - 30%, `qwen2.5:0.5b`, routing `0.90`)
  - `starter_v0/runs/v3-local_B_group_ollama_20261111T101227317712.json` (6/10 - 60%, `qwen2.5:3b`, routing `1.00`)
- **AI/công cụ đã dùng và cách tự kiểm tra kết quả:**
  - Công cụ AI: Sử dụng trợ lý AI Antigravity để hỗ trợ đọc phân tích lỗi JSON log, đề xuất giả thuyết tách biến v1–v3, viết cấu trúc YAML/Markdown và tạo unit tests với mock `fake_http`.
  - Tự kiểm tra: Chạy kiểm thử toàn bộ test suite qua `pytest tests/` từ thư mục `starter_v0/` (mốc CP3 đạt **134 passed**, contract **70 passed**; trạng thái đồng bộ trên `main` hiện đạt **159 passed, 0 failed, 0 skipped**, `test_tool_contract.py` **75 passed**), trích xuất thống kê trực tiếp từ các file `runs/*.json` bằng script `starter_v0/scripts/parse_runs.py` và PowerShell, rà soát `git status` & `git grep` đảm bảo không rò rỉ API key hay sửa đổi `eval_base.json`.

## 5. Tự đánh giá

- Thực hiện đúng và đầy đủ vòng lặp khoa học dựa trên bằng chứng (Evidence-driven optimization) ở CP1–CP4.
- Mọi kết luận và metric trong báo cáo đều đối chiếu trực tiếp từ run JSON có `provider_error_cases == 0` và output thực tế của `pytest`.
- Toàn bộ test suite không còn case nào `skipped` hay `failed`, đảm bảo tuân thủ nghiêm ngặt quy định liêm chính và bảo mật.
