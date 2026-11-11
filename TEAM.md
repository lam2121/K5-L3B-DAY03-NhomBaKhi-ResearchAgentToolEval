# Team

## Thông tin nhóm

- Tên nhóm: Nhóm Bá Khí (`NhomBaKhi`)
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY03-NhomBaKhi-ResearchAgentToolEval`
- URL repo, nhánh nộp, commit chốt: `https://github.com/lam2121/K5-L3B-DAY03-NhomBaKhi-ResearchAgentToolEval`, nhánh `main`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Duy Hưng | 2A202604110 | | Hungnd1208 | Prompt & Tool Declarations (v0–v3), Eval Pipeline, `version_log.csv`, Ollama Local Eval, Unit Tests & Contract Sync |
|   2 | Nguyễn Văn Lâm | 2A202604115 | 26ai.lamnv@vinuni.edu.vn | lam2121 | Trưởng nhóm, Streamlit UI (`app.py`), Báo cáo chung & Điều phối; Hỗ trợ prompt/tool thứ 4/unit test/transcript Telegram |
|   3 | Nguyễn Quốc Toàn | 2A202604098 | | nguyenquoctoan0310 | QA & Unit Test Engineer — Hoàn thiện Unit Tests offline cho các core tools trong `tests/test_tools_todo.py` |
|   4 | Đặng Nguyễn Nhật Minh | 2A202604118 | 26ai.minhdnn@vinuni.edu.vn | MN-HD | Data & Live Chat Engineer — Thiết kế 10 Eval cases nhóm (`data/eval_group.json`), Live chat transcripts & Group Eval |
|   5 | Nguyễn Trung Dũng | 2A202604084 | | bomgoa1102 | New Tool Builder — Phát triển 3 tools mới (`source_deduplicator`, `evidence_finder`, `topic_radar`), `TOOL.md` và mock tests |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/team/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

---

## NguyenDuyHung-2A202604110

### Phần việc phụ trách

- Thực hiện Checkpoint 1: Chạy baseline v0, phân tích 7 ca lỗi ban đầu (Failure Analysis).
- Thực hiện Checkpoint 2: Đặt giả thuyết và tối ưu tuần tự `system_prompt.md` và `tools.yaml` qua các phiên bản v1, v2, v3; cập nhật `artifacts/version_log.csv` chuẩn UTF-8.
- Thực hiện Checkpoint 3: Đăng ký 3 tool mới (`topic_radar`, `evidence_finder`, `source_deduplicator`) vào `tools/__init__.py` và `artifacts/tools.yaml`; hoàn thiện unit tests trong `tests/test_tools_todo.py` để đạt 134/134 tests toàn suite (0 skipped, 0 failed).
- Thực hiện Checkpoint 4: Chạy đánh giá và chuẩn hóa `data/eval_group.json` cùng `system_prompt.md`, nâng điểm suite nhóm từ 50% (5/10) lên 90% (9/10).
- Hoàn thiện báo cáo nhóm `report/team/group_report.md` và kiểm tra tuân thủ quy tắc toàn bộ dự án.

### Đóng góp cụ thể

- Phân tích nguyên nhân lỗi v0 (65%, 13/20): do starter prompt ép đoán bừa tài khoản/URL, tự ý gửi không xin phép, và ép gọi 1 bước.
- Tối ưu v1 (70%, 14/20): Bỏ chỉ dẫn xấu, bắt buộc gọi `clarify` khi thiếu handle/URL và xin xác nhận trước khi gửi.
- Tối ưu v2 (80%, 16/20): Phân biệt `social_search` theo chủ đề với thiếu account cá nhân, kích hoạt gọi tool song song và chỉ định `response_type: yes_no` cho confirm (Tool Routing Accuracy đạt 100%).
- Tối ưu v3 (85%, 17/20 trên OpenRouter `gpt-4o-mini`, `provider_error_cases=0`): Chuẩn hóa mô tả khi nào dùng/không dùng và quy ước tham số sạch trong `artifacts/tools.yaml`.
- Xử lý lỗi rate limit (429/503) trong `providers/gemini_provider.py` bằng cơ chế retry.
- Chuẩn hóa bộ test nhóm `eval_group.json` (loại bỏ assert dư thừa lên tham số mặc định `false` ở `G01`, `G03`, tối ưu điều kiện kiểm tra `G05`), nâng điểm từ 5/10 lên 9/10 ở commit `ac3e062`, và kiểm chứng cuối tại `v3_B_group_openrouter_20261111T091833288768.json` đạt **10/10 (100%)** (`8987678`).
- Chạy thử nghiệm Ollama local `eval_group` với `qwen2.5:0.5b` (3/10 = 30%, routing 0.90) và `qwen2.5:3b` (6/10 = 60%, routing 1.00), ghi nhận vào `version_log.csv` (`bf2053a`).

### Bằng chứng

- File artifacts & config: `starter_v0/artifacts/system_prompt.md`, `starter_v0/artifacts/tools.yaml`, `starter_v0/artifacts/version_log.csv`, `starter_v0/tools/__init__.py`, `starter_v0/providers/gemini_provider.py`, `starter_v0/data/eval_group.json`, `starter_v0/tests/test_tools_todo.py`.
- Các file run thực nghiệm hợp lệ (`provider_error_cases == 0`):
  + `starter_v0/runs/v0_B_base_gemini_20261110T174737486284.json`
  + `starter_v0/runs/v1_B_base_gemini_20261110T181127878174.json`
  + `starter_v0/runs/v2_B_base_gemini_20261110T182200983550.json`
  + `starter_v0/runs/v3_B_base_openrouter_20261111T001429486980.json`
  + `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json`
  + `starter_v0/runs/v3-local_B_group_ollama_20261111T100206146391.json`
  + `starter_v0/runs/v3-local_B_group_ollama_20261111T101227317712.json`
- Commits: `463c8a9`, `17f136e`, `7ef6de9`, `d222de1`, `ac3e062`, `45d93ab`, `8987678`, `bf2053a`.

---

## NguyenVanLam-2A202604115

### Phần việc phụ trách

- Vai trò chính (Trưởng nhóm — UI, Báo cáo chung & Điều phối): Xây dựng và nâng cấp giao diện Web UI bằng Streamlit (`starter_v0/app.py`, `starter_v0/requirements.txt`, chuẩn bị tunnel với `starter_v0/cloudflared.exe`), biên tập Phần A & tổng quan `report/team/group_report.md`, hoàn thiện bảng phân công trong `TEAM.md`, viết báo cáo cá nhân `report/team/2A202604115_NguyenVanLam.md` và điều phối nộp bài.

### Đóng góp cụ thể & Hỗ trợ thành viên khác

- **Phần việc chính:** Khởi tạo/hoàn thiện `starter_v0/app.py` (chat đa lượt, chọn đa provider `openrouter`/`gemini`/`openai`/`anthropic`/`ollama`, mặc định `v3`, đọc `version_log.csv`, hiển thị tool call/lỗi rõ ràng và hỗ trợ flow xác nhận gửi Telegram) cùng khung tài liệu báo cáo nhóm/cá nhân.
- **Hỗ trợ Hưng (Prompt & Eval Engineer):** Lưu trữ, đồng bộ các phiên bản `starter_v0/artifacts/system_prompt_v0.md`–`v3.md`, `system_prompt.md` (`ac2a337`, `d4c74a2`) và chỉnh cấu hình mặc định trong `starter_v0/providers/gemini_provider.py` (`d80c157`).
- **Hỗ trợ Dũng (New Tool Builder & Tool Contract):** Phát triển thêm tool thứ 4 `rss_feed_reader` (`starter_v0/tools/rss_feed_reader/tool.py`, `TOOL.md`), đăng ký vào `starter_v0/tools/__init__.py` và `starter_v0/artifacts/tools.yaml` (không thay đổi `system_prompt.md`).
- **Hỗ trợ Toàn (QA & Unit Test Engineer):** Viết 3 tests chuẩn C4 cho `rss_feed_reader` và test failure path cho `source_deduplicator` trong `starter_v0/tests/test_tools_todo.py`, cùng bộ test offline `starter_v0/tests/test_rss_feed_reader.py` (nâng tổng suite lên 152 passed); sau khi Toàn bổ sung thêm 7 core tool tests (`717a643`), toàn suite hiện đạt **159 passed, 0 failed, 0 skipped**.
- **Hỗ trợ Minh (Data & Live Chat Engineer):** Chạy phiên live chat gửi Telegram thật sau xác nhận (`starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json`, `status: "sent"`) và cập nhật case `G05_rss_feed_reader_direct` trong `starter_v0/data/eval_group.json` cho tool thứ 4; run kiểm chứng `--suite group` sau đó đạt **10/10 (100%)**.

### Bằng chứng

- Files & config: `starter_v0/app.py`, `starter_v0/requirements.txt`, `starter_v0/tools/rss_feed_reader/` (`tool.py`, `TOOL.md`), `starter_v0/tests/test_rss_feed_reader.py`, `starter_v0/tests/test_tools_todo.py`, `starter_v0/artifacts/system_prompt_v0.md`–`v3.md`, `starter_v0/data/eval_group.json`, `TEAM.md`, `report/team/group_report.md`, `report/team/2A202604115_NguyenVanLam.md`.
- Transcript & run liên quan:
  + `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json` (`status: "sent"`)
  + `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (`G05` PASS, 10/10)
- Commits trên `main`: `d80c157`, `ac2a337`, `d4c74a2`, `373a505`, `b51da75`, `0bc7e33`.

---

## NguyenQuocToan-2A202604098

### Phần việc phụ trách

- Hoàn thiện toàn bộ unit test cho các core tools nghiên cứu trong `starter_v0/tests/test_tools_todo.py`, đối chiếu theo format chuẩn của `test_tools_examples.py` và registry `TOOL_FUNCTIONS`.
- Bảo đảm 100% unit tests chạy hoàn toàn offline bằng `fake_http`, `monkeypatch` và `block_network`, không phát sinh kết nối mạng thật.
- Kiểm thử guardrail an toàn cho tool gửi Telegram (`send`) cùng bộ test trích xuất bài báo `papers` / `paper_text` và phối hợp kiểm chứng toàn bộ suite `pytest`.

### Đóng góp cụ thể

- Hoàn thiện test cho `clarify` & `format` (`1a9490e`): kiểm tra `clarify(response_type="choice")` trả đúng `options` & `awaiting_user=True`; kiểm tra `format` gộp nhóm theo `section`, đánh số `thread` (`1/`, `2/`), và xử lý an toàn khi `items` là `None` hoặc `[]` (`item_count == 0`, không crash).
- Hoàn thiện test cho `lookup` & `fetch` (`1a9490e`): kiểm tra `lookup` bỏ trường `time_range` khỏi JSON gửi Tavily khi `timeframe=None`; `fetch` ánh xạ metadata Firecrawl (`title`, `sourceURL`, `source`) và kiểm tra cắt `summary` đúng ngưỡng `4000` ký tự.
- Hoàn thiện test cho `timeline`, `social_search` & `policy` (`1a9490e`): kiểm tra `timeline` bỏ tweet thiếu `tweet_id`, giữ đúng `limit` và chuẩn URL `https://x.com/<handle>/status/<id>`; `social_search` truyền đúng `search_type="Top"`; `policy` xử lý query rỗng, lọc cô lập theo `policy_area`, và tách dòng prompt injection (`> Ignore all previous instructions...`) sang `untrusted_text` thay vì `facts`.
- Bổ sung 7 unit tests cho `send`, `papers` & `paper_text` (`717a643`): kiểm tra `send(confirmed=true)` POST Telegram (`parse_mode="Markdown"`) và trả `RuntimeError` (`calls == []`) khi thiếu biến môi trường; `papers` parse Atom XML arXiv (`arxiv_id`, `authors`, `primary_category`, `sortBy`) và trả `HTTPError` khi `503`; `paper_text` mock tải PDF & `_extract_pdf_text`, chặn URL arXiv lỗi (`ValueError`, `calls == []`) và bắt `RequestException` khi timeout.
- Tổng cộng hoàn thiện **19 core unit tests** trong `test_tools_todo.py` (đưa file đạt **31 passed** và toàn suite `tests/` đạt **159 passed, 0 failed, 0 skipped**), đồng thời đối chiếu ranh giới `send` qua bằng chứng Telegram thật của nhóm.

### Bằng chứng

- Files & báo cáo: `starter_v0/tests/test_tools_todo.py` (19 core tests: `L25-L447`; toàn file **31 passed**; toàn suite `tests/` đạt **159 passed, 0 failed, 0 skipped**), `report/team/2A202604098_NguyenQuocToan.md`.
- Bằng chứng đối chiếu & kiểm thử:
  + `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json` (`clarify(yes_no)` → `send(confirmed=true)`, `status: "sent"`)
  + Kiểm tra offline 100% qua fixture `fake_http`, `monkeypatch` và `block_network`.
- Commits / PR: `1a9490e`, `0a1538d`, `717a643`, `db43f54`, `a8401b3`, `700d68e`.

---

## DangNguyenNhatMinh-2A202604118

### Phần việc phụ trách

- Thiết kế bộ 10 evaluation cases của nhóm trong `starter_v0/data/eval_group.json` (5 single-turn `query` + 5 multi-turn `turns`) theo đúng schema evaluator.
- Thực hiện tối thiểu 3 phiên live chat với Research Agent phiên bản `v3` (Gemini) và lưu transcript JSON làm bằng chứng.
- Chạy group evaluation trên Gemini, phân tích các trường hợp FAIL, điều chỉnh các kỳ vọng đánh giá quá cứng và kiểm chứng lại đạt 10/10 PASS.
- Kiểm thử lại toàn bộ test suite và đồng bộ dataset, transcripts, run JSON lên nhánh `main`.

### Đóng góp cụ thể

- Xây dựng 10 test cases trong `eval_group.json` (đầy đủ `failure_type`, `expect`, `metadata.what_it_tests`) bao phủ 6 nhóm tình huống mà `eval_base.json` chưa kiểm tra sâu: `topic_radar` (`G01`, `GM02` so sánh báo chí & X theo `timeframe`/`limit`), `evidence_finder` (`G02`, `GM03` tìm bằng chứng cho `claim` & kiểm soát `deep_read`), `source_deduplicator` (`G03`, `GM01` lọc trùng cục bộ & không gọi extraction API ở lượt 2), `format` (`G04`, `GM04` `brief`/`bullets` & đổi `headline`), `rss_feed_reader` (`G05`) và `paper_text` (`GM05` giữ `2307.09288` & sửa `max_pages=3`).
- Thực hiện 3 phiên live chat với Gemini `v3`: Phiên 1 (`193456801873`, tin AI 5 nguồn → `lookup` → `format`); Phiên 2 (`194637712792`, thiếu tài khoản → `clarify` giữ `limit=5` sang turn 2 → `timeline("elonmusk", 5)` → `format`); Phiên 3 (`194822685087`, yêu cầu gửi Telegram → `clarify(yes_no)` → user từ chối `"No. Do not send it."` → dừng gửi an toàn).
- Chạy Group Evaluation trên Gemini `v3` lần 1 (`v3_B_group_gemini_20261111T102925310969.json`) đạt `8/10 (80%, provider_error_cases = 0)`, phát hiện 2 ca FAIL giả do evaluator quá cứng: `G01` ép model truyền `include_social=true` (vốn đã mặc định) và `GM03` so dấu chấm cuối `claim`.
- Chuẩn hóa kỳ vọng ở `G01` và `GM03`, chạy kiểm chứng lần 2 (`v3_B_group_gemini_20261111T103806407842.json`) đạt **10/10 PASS (100%)** (`case_accuracy`, `tool_routing_accuracy`, `argument_accuracy`, `multiturn_accuracy` đều `1.0`, `provider_error_cases = 0`), và chạy lại `pytest -q` đạt **159 passed**.

### Bằng chứng

- Dataset & báo cáo: `starter_v0/data/eval_group.json`, `report/team/2A202604118_DangNguyenNhatMinh.md`.
- 3 transcripts & 2 runs Gemini:
  + `starter_v0/transcripts/v3_gemini_20261110T193456801873.transcript.json` (Phiên 1 — Normal research)
  + `starter_v0/transcripts/v3_gemini_20261110T194637712792.transcript.json` (Phiên 2 — Missing info + multi-turn)
  + `starter_v0/transcripts/v3_gemini_20261110T194822685087.transcript.json` (Phiên 3 — Safety & từ chối gửi)
  + `starter_v0/runs/v3_B_group_gemini_20261111T102925310969.json` (Lần 1: 8/10)
  + `starter_v0/runs/v3_B_group_gemini_20261111T103806407842.json` (Lần 2: 10/10 PASS)
- Commits: `1633c37`, `1fc7bbc`, `5bdda64`, `59c3a45`.

---

## NguyenTrungDung-2A202604084

### Phần việc phụ trách

- Xây dựng 3 công cụ nghiên cứu mới (Checkpoint 3): `source_deduplicator`, `evidence_finder` và `topic_radar` kèm cấu trúc trả kết quả/lỗi chuẩn để không làm gián đoạn luồng agent.
- Viết tài liệu đặc tả sử dụng, tham số và giới hạn (`TOOL.md`) cho từng công cụ để làm cơ sở khai báo vào registry `tools/__init__.py` và `artifacts/tools.yaml`.
- Phát triển 3 bộ unit test mock HTTP offline bao phủ đầu vào hợp lệ, đầu vào sai và lỗi dịch vụ ngoài; kiểm thử lại toàn bộ suite `pytest`.

### Đóng góp cụ thể

- `source_deduplicator`: Khử trùng lặp danh sách nguồn theo URL chuẩn hóa (bỏ tham số theo dõi `utm_*`, `fbclid`, `gclid`) và mức tương đồng tiêu đề; mặc định chạy cục bộ không tốn API, khi bật `verify_content` mới đọc đối chiếu qua Firecrawl và fallback sang Tavily Extract khi cần.
- `evidence_finder`: Tìm kiếm tài liệu theo `claim` qua Tavily, loại URL không hợp lệ (gồm URL nội bộ), chọn lọc kết quả từ các hostname khác nhau rồi trích đoạn liên quan; hỗ trợ `deep_read` (Firecrawl → Tavily Extract), sắp xếp bằng `relevance_score` theo khớp từ khóa nhưng cố ý giữ `assessment: requires_review` thay vì tự phán quyết đúng/sai.
- `topic_radar`: Tổng hợp tin tức Tavily News và bài đăng X (RapidAPI Twitter API45) cho cùng chủ đề, thống kê số bài báo, hostname, bài X và từ khóa giao thoa; giới hạn số kết quả & số bài làm giàu nội dung để tránh gọi API quá mức, và trả `partial` kèm cảnh báo khi một bên lỗi/thiếu key.
- Viết 3 bộ unit test offline dùng mock HTTP: `test_source_deduplicator.py` (7 tests: tracking URL, tiêu đề gần trùng, fallback đọc nội dung), `test_evidence_finder.py` (14 tests: đa domain, URL nội bộ, rỗng, lỗi cả 2 provider) và `test_topic_radar.py` (16 tests: gộp 2 nguồn, thiếu key/lỗi 1 bên, giới hạn làm giàu tối đa 2 bài, kiểm tra tham số) — tổng **37 passed** (toàn suite đạt **159 passed, 0 failed**).

### Bằng chứng

- Files & tài liệu tool: `starter_v0/tools/source_deduplicator/`, `starter_v0/tools/evidence_finder/`, `starter_v0/tools/topic_radar/` (đủ `tool.py` & `TOOL.md`), `report/team/2A202604084_NguyenTrungDung.md`.
- Files unit test & đối chiếu tích hợp:
  + `starter_v0/tests/test_source_deduplicator.py` (7 tests), `test_evidence_finder.py` (14 tests), `test_topic_radar.py` (16 tests) — tổng `37 passed`
  + Đối chiếu tích hợp 6 cases (`G01`–`G03`, `GM01`–`GM03`) trong `starter_v0/data/eval_group.json` và run `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json` (10/10)
- Commits: `771be66`, `41f3b77`, `93e21c9`, `38274ee`.
