# Báo cáo cá nhân — Nguyễn Văn Lâm (2A202604115)

## 1. Thông tin thành viên

- Họ tên: Nguyễn Văn Lâm
- MSSV: `2A202604115`
- Email: `26ai.lamnv@vinuni.edu.vn`
- GitHub: `lam2121`
- Vai trò chính (Trưởng nhóm): UI & Demo Lead (`starter_v0/app.py`, Cloudflare Tunnel), phụ trách tổng hợp Báo cáo chung (`report/team/group_report.md`), `TEAM.md`, báo cáo cá nhân và điều phối nộp bài.

## 2. Phần việc phụ trách

### 2.1. Công việc chính theo phân công nhóm
- Khởi tạo và nâng cấp giao diện Web UI bằng Streamlit (`starter_v0/app.py`, `starter_v0/requirements.txt`) để chat trực tiếp với Research Agent, hiển thị quá trình gọi tool/lỗi rõ ràng và hỗ trợ luồng xác nhận gửi Telegram.
- Chuẩn bị binary `starter_v0/cloudflared.exe` phục vụ mở Cloudflare Tunnel demo UI khi trình diễn.
- Khởi tạo và hoàn thiện phần cá nhân trong `TEAM.md`, cập nhật Phần A/Mục tổng quan trong `report/team/group_report.md` và báo cáo cá nhân `report/team/2A202604115_NguyenVanLam.md`; theo dõi, rà soát tiến độ chung và điều phối nộp bài của nhóm.

### 2.2. Phần hỗ trợ các thành viên khác
- **Hỗ trợ Hưng (Prompt & Eval Engineer):** Lưu trữ, đồng bộ lịch sử các phiên bản prompt (`starter_v0/artifacts/system_prompt_v0.md` → `v3.md`) vào `starter_v0/artifacts/system_prompt.md` và cấu hình model mặc định trong `starter_v0/providers/gemini_provider.py`.
- **Hỗ trợ Dũng (New Tool Builder & Tool Contract):** Phát triển thêm tool thứ 4 `rss_feed_reader` (`starter_v0/tools/rss_feed_reader/tool.py`, `TOOL.md`), đăng ký vào `starter_v0/tools/__init__.py` và khai báo schema vào `starter_v0/artifacts/tools.yaml` (giữ nguyên `starter_v0/artifacts/system_prompt.md`, không sửa prompt) để đạt tiêu chí Bonus ≥ 4 tool mới.
- **Hỗ trợ Toàn (QA & Unit Test Engineer):** Viết bổ sung bộ 3 unit tests theo chuẩn C4 cho `rss_feed_reader` và test failure path cho `source_deduplicator` trong `starter_v0/tests/test_tools_todo.py`, cùng bộ test offline mở rộng trong `starter_v0/tests/test_rss_feed_reader.py`.
- **Hỗ trợ Minh (Data & Live Chat Engineer):** Chạy phiên live chat sinh transcript gửi Telegram thực tế sau xác nhận người dùng (`starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json`) và cập nhật case `G05_rss_feed_reader_direct` vào `starter_v0/data/eval_group.json` (giữ đúng 10 cases: 5 single-turn + 5 multi-turn) cho tool thứ 4; sau đó Hưng đã chạy kiểm chứng lại `--suite group` ở commit `8987678`.

## 3. Đóng góp cụ thể

- **Xây dựng Streamlit UI (`starter_v0/app.py`, `starter_v0/requirements.txt`):**
  - Thiết kế giao diện hội thoại đa lượt kết nối trực tiếp `ResearchAgent` và `TOOL_FUNCTIONS`.
  - Hỗ trợ kiểm soát hành vi xác nhận `yes_no` trước khi gửi bản tin ra Telegram và hiển thị rõ các tool call / lỗi tool thay vì che lỗi.
- **Hỗ trợ luồng Live Chat & Gửi Telegram thật (`d4c74a2`):**
  - Thực hiện phiên chat đa lượt lưu tại `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json` (Turn 3–4 cho thấy agent gọi `clarify(response_type="yes_no")` hỏi xác nhận, sau khi user xác nhận mới gọi `send(confirmed=true)` và trả về `status: "sent"`).
- **Hỗ trợ đồng bộ Prompt & Provider (`d80c157`, `ac2a337`, `d4c74a2`):**
  - Lưu lại lịch sử `system_prompt_v0.md` đến `system_prompt_v3.md` phục vụ đối chiếu quá trình tối ưu của nhóm, đồng thời giữ nguyên `system_prompt.md` ở trạng thái chuẩn sau `ac3e062` / `45d93ab`.
- **Hỗ trợ Tool thứ 4 `rss_feed_reader` + Unit Tests + Eval Case (`G05`) (`373a505`, kiểm chứng tại `8987678`):**
  - Xây dựng `read_rss_feed(url, max_items, keyword)` đọc RSS 2.0 / RSS 1.0 (RDF) và Atom XML feed với kiểm tra URL an toàn (chặn `localhost`, `.local`, `.internal`, userinfo, IP private/loopback và redirect nội bộ), chặn `<!DOCTYPE` / `<!ENTITY`, làm sạch HTML description và luôn trả dict lỗi chuẩn qua `err("rss_feed_reader", exc)`.
  - Hoàn thiện 3 tests chuẩn C4 cho `rss_feed_reader` và test failure path cho `source_deduplicator` trong `tests/test_tools_todo.py`, cùng 9 ca test offline (`fake_http`) trong `tests/test_rss_feed_reader.py`, nâng tổng suite lên **152 passed, 0 failed, 0 skipped**.
  - Thay `G05_lookup_general_documentation` bằng `G05_rss_feed_reader_direct` trong `starter_v0/data/eval_group.json` (giữ đúng 5 single-turn + 5 multi-turn). Kết quả chạy kiểm chứng thực tế tại `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (commit `8987678`) đạt **10/10 PASS (100%)** (`provider_error_cases: 0`), trong đó `G05_rss_feed_reader_direct` gọi chính xác `rss_feed_reader(url="https://openai.com/news/rss.xml", max_items=4, keyword="safety")` mà không cần chỉnh sửa `system_prompt.md`.

## 4. Bằng chứng

- **File/commit/PR:**
  - Commits của bản thân trên nhánh `main`:
    + `d80c157`: Khởi tạo Streamlit UI (`starter_v0/app.py`), `requirements.txt`, chỉnh mặc định `providers/gemini_provider.py`, khung `TEAM.md` và `report/team/`.
    + `ac2a337`: Nâng cấp `starter_v0/app.py`, đồng bộ `system_prompt.md` và lưu `system_prompt_v1.md`–`v3.md`.
    + `d4c74a2`: Lưu `system_prompt_v0.md` và transcript gửi Telegram thực tế `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json`.
    + `373a505`: Bổ sung tool thứ 4 `rss_feed_reader` (`tool.py`, `TOOL.md`, `tools/__init__.py`, `artifacts/tools.yaml`), unit tests (`tests/test_tools_todo.py`, `tests/test_rss_feed_reader.py`), case `G05_rss_feed_reader_direct` trong `data/eval_group.json`, cập nhật `TEAM.md`, `group_report.md` và `2A202604115_NguyenVanLam.md`.
  - Commit kiểm chứng tiếp nối từ thành viên nhóm (`8987678`, `Hungnd1208`):
    + Chạy `--suite group` trên OpenRouter (`openai/gpt-4o-mini`) cho bộ case có `G05_rss_feed_reader_direct`: `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (**10/10 PASS = 100%**, `provider_error_cases = 0`).
  - Các file liên quan trên `main`:
    + `starter_v0/app.py`, `starter_v0/requirements.txt`
    + `starter_v0/tools/rss_feed_reader/tool.py`, `starter_v0/tools/rss_feed_reader/TOOL.md`
    + `starter_v0/tools/__init__.py`, `starter_v0/artifacts/tools.yaml`
    + `starter_v0/tests/test_tools_todo.py`, `starter_v0/tests/test_rss_feed_reader.py`
    + `starter_v0/data/eval_group.json`, `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json`
    + `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json`
    + `TEAM.md`, `report/team/group_report.md`, `report/team/2A202604115_NguyenVanLam.md`
- **AI/công cụ đã dùng và cách tự kiểm tra kết quả:**
  - Sử dụng trợ lý AI (Antigravity) hỗ trợ thiết kế mã nguồn Streamlit UI, parser XML + SSRF guard cho `rss_feed_reader`, chuẩn hóa test offline và biên tập tài liệu báo cáo.
  - Cách tự kiểm tra kết quả:
    + Chạy `.\venv\Scripts\python.exe -m pytest tests/ -v`: đạt **152 passed in 0.50s** (`0 failed, 0 skipped`), trong đó `test_tool_contract.py` đạt **75/75 passed** và chặn mọi kết nối mạng thật qua `block_network`.
    + Kiểm chứng `eval_group.json` bằng run thực tế `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json`: **10/10 PASS (100%)**, `case_accuracy = 1.0`, `tool_routing_accuracy = 1.0`, `argument_accuracy = 1.0`, `multiturn_accuracy = 1.0`, `provider_error_cases = 0`.
    + Kiểm tra `git diff 45d93ab 8987678 -- starter_v0/artifacts/system_prompt.md`: trống (`prompt_hash` giữ nguyên `90e5ccb74af1`).
    + Kiểm chứng luồng live chat & Telegram thật qua log JSON `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json` (`status: "sent"`).

## 5. Tự đánh giá

- **Điểm làm tốt:** Hoàn thành đầy đủ giao diện UI Streamlit, chứng minh được luồng live chat có xác nhận trước khi gửi Telegram thật, hoàn thiện tài liệu phân công/báo cáo theo đúng vai trò Trưởng nhóm và hỗ trợ hoàn thiện tool thứ 4 `rss_feed_reader` cùng unit tests offline (152 passed, 0 skipped, 0 failed) góp phần giúp run group eval mới nhất (`8987678`) đạt **10/10 (100%)** và đủ điều kiện cho tiêu chí Bonus ≥ 4 tool mới.
- **Hạn chế / Việc cần lưu ý:** Ở mức toàn nhóm (C8), vẫn cần nhắc các thành viên còn lại (Toàn, Minh, Dũng) bổ sung đầy đủ họ tên/MSSV vào `TEAM.md` và hoàn thiện file báo cáo cá nhân `report/team/MSSV_HoTen.md` trước khi chốt nộp bài cuối cùng.
