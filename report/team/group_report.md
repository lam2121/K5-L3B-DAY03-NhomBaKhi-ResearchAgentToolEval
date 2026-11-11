# Báo cáo nhóm — Day 03 Research Agent Tool Eval

> Báo cáo gồm 2 phần:
> - **Phần A (mục 2)** — giới thiệu agent, ngắn gọn 1 trang.
> - **Phần B (mục 3–7)** — chi tiết và bằng chứng, dựa hoàn toàn trên log thật trong `starter_v0/runs/` và `starter_v0/transcripts/`.

## 1. Thông tin nhóm

- Tên nhóm: **Nhóm Bá Khí** (`K5-L3B-DAY03-NhomBaKhi-ResearchAgentToolEval`)
- Thành viên và phân công: xem [TEAM.md](../../TEAM.md)
- Provider/model dùng cho v0–v3 & Local Eval:
  - v0–v2 (`eval_base`): Google Gemini (`gemini-3.5-flash-lite`)
  - v3 (`eval_base`): OpenRouter (`openai/gpt-4o-mini`) *(chuyển sang OpenRouter do Gemini Free Tier gặp giới hạn 15 RPM / 503 khi chạy batch 20 cases)*
  - Group Eval (`eval_group`, 10/10 PASS): OpenRouter (`openai/gpt-4o-mini`) và Google Gemini (`gemini-3.5-flash-lite`)
  - Thử nghiệm Small Local Model (`eval_base` & `eval_group`, baseline `v3-local`): Ollama (`qwen2.5:0.5b`, `qwen2.5:3b`)

## 2. Mục tiêu và cách tiếp cận — PHẦN A: Giới thiệu agent

### A1. Agent này làm được gì

"Research agent đa kênh: hỗ trợ tra cứu tin tức web, đọc trực tiếp RSS/Atom feed (XML), bài đăng mạng xã hội X/Twitter, bài báo khoa học arXiv, tra cứu chính sách nội bộ, tự động khử trùng nguồn, tìm kiếm bằng chứng kiểm chứng luận điểm, đối chiếu chủ đề đa nền tảng, tổng hợp bản tin ngắn và gửi bản tin lên Telegram sau khi được người dùng xác nhận rõ ràng."

**Cách chạy UI:** `cd starter_v0 && python -m streamlit run app.py` (dependency `streamlit>=1.30.0` đã được khai báo trong `starter_v0/requirements.txt`).

**Link cục bộ :** http://localhost:8501

> Chạy cục bộ qua Streamlit UI theo hướng dẫn ở trên (`http://localhost:8501`).

### A2. Tool agent có

| Tên tool | Làm được gì | Tool mới nhóm thêm? |
|---|---|---|
| `clarify` | Hỏi lại người dùng khi thiếu handle/URL hoặc xin xác nhận (`yes_no`) trước khi gửi | Không |
| `timeline` | Lấy các bài đăng/tweet gần đây của một tài khoản X cụ thể (handle không có `@`) | Không |
| `social_search` | Tìm bài đăng/tweet trên mạng xã hội theo từ khóa/chủ đề (`Latest` hoặc `Top`) | Không |
| `lookup` | Tra cứu thông tin tổng quát hoặc tin tức báo chí trên web qua Tavily (`topic`, `timeframe`) | Không |
| `fetch` | Đọc và trích xuất nội dung toàn văn từ một đường dẫn URL cụ thể | Không |
| `format` | Định dạng danh sách các nguồn tin đã thu thập theo khuôn (`brief`, `sections`, `bullets`, `thread`, `daily_ai_vn`) | Không |
| `send` | Gửi bản tin/tin nhắn lên kênh Telegram riêng của nhóm (yêu cầu `confirmed=true`) | Không |
| `policy` | Tìm kiếm quy định và tài liệu chính sách nội bộ công ty | Không |
| `papers` | Tìm kiếm bài báo khoa học trên arXiv | Không |
| `paper_text` | Trích xuất nội dung văn bản từ PDF bài báo arXiv theo ID/URL | Không |
| `topic_radar` | Đối chiếu tin tức báo chí (Tavily) và trao đổi trên X (RapidAPI) cho cùng một chủ đề, đo độ giao thoa từ khóa | **Có** |
| `evidence_finder` | Tìm kiếm bằng chứng và trích đoạn liên quan nhất cho một luận điểm (`claim`) từ nhiều tên miền khác nhau | **Có** |
| `source_deduplicator` | Khử trùng lặp các nguồn tin theo URL chuẩn hóa (bỏ UTM/tracker), tiêu đề, và tùy chọn đối chiếu nội dung sâu | **Có** |
| `rss_feed_reader` | Đọc và phân tích trực tiếp RSS 2.0 / RDF / Atom feed (XML) từ URL công khai, hỗ trợ giới hạn số lượng và lọc `keyword` | **Có** |

### A3. Câu hỏi mẫu để thử

1. *"Tin tức AI hôm nay có gì mới? Tóm tắt giúp mình."* (Kiểm tra `lookup` với `topic=news`, `timeframe=day`)
2. *"Tóm tắt 5 tweet mới nhất giúp mình."* → Sau khi agent hỏi lại, trả lời *"Của Elon Musk"* (Kiểm tra `clarify` → `timeline(screenname="elonmusk")`)
3. *"So sánh tin tức báo chí và thảo luận trên X về robotics trong tháng qua, lấy tối đa 4 nguồn mỗi bên."* (Kiểm tra tool mới `topic_radar`)
4. *"Tìm bằng chứng từ 4 website khác nhau cho nhận định 'Urban tree cover can lower neighborhood surface temperatures during heat waves', chỉ dùng search snippet."* (Kiểm tra tool mới `evidence_finder`)
5. *"Gửi tóm tắt tin tức AI hôm nay lên Telegram giúp tôi."* → Xác nhận *"Tôi xác nhận gửi"* (Kiểm tra boundary xác nhận `clarify(yes_no)` → `send(confirmed=true)`)

## 3. Giải pháp

- **Tối ưu `starter_v0/artifacts/system_prompt.md`:**
  - Loại bỏ toàn bộ các chỉ dẫn sai lệch cố ý của starter (tự đoán tài khoản Sam Altman khi thiếu handle, tự đoán URL khi thiếu link, tự ý gửi tin mà không hỏi, ép hoàn thành trong một bước).
  - Bổ sung quy tắc hỏi lại (`clarify`) khi thiếu ngữ cảnh bắt buộc và ranh giới bắt buộc xác nhận (`response_type: yes_no`) trước khi gọi `send`.
  - Hỗ trợ duy trì ngữ cảnh hội thoại đa lượt (multi-turn carryover & correction), định tuyến song song (`lookup` + `social_search`) khi cần đa kênh và định tuyến độc quyền tới tool mới `topic_radar` khi đối chiếu đa nền tảng.
- **Chuẩn hóa `starter_v0/artifacts/tools.yaml`:**
  - Viết lại mô tả cho cả 14 tools theo chuẩn: làm gì, **KHI NÀO DÙNG**, **KHI NÀO KHÔNG DÙNG**.
  - Chuẩn hóa quy ước tham số: `screenname` không có `@`; `query` chỉ chứa từ khóa cốt lõi (loại bỏ filler words như "tin tức", "news", "hôm nay"); ranh giới `confirmed=true` cho `send`.
- **Phát triển 4 tool mới (`topic_radar`, `evidence_finder`, `source_deduplicator`, `rss_feed_reader`):**
  - Mỗi tool có đủ `tool.py`, `TOOL.md` đúng frontmatter contract, được đăng ký đầy đủ trong `tools/__init__.py` và `artifacts/tools.yaml`, và xử lý ngoại lệ an toàn bằng cách trả về error dict thay vì raise exception.

## 4. Kết quả — PHẦN B: Chi tiết và bằng chứng

### B1. Version Evidence

Lấy từ `starter_v0/artifacts/version_log.csv` và `starter_v0/runs/*.json`. Tất cả các run dưới đây đều hợp lệ (`provider_error_cases == 0` và `measured_cases == total_cases = 20`).

| Version | Changed Artifact | Hypothesis | Metric Before | Metric After | Run File |
|---|---|---|---:|---:|---|
| v0 | baseline | Đo lường mức độ lỗi ban đầu khi chưa tối ưu | - | 0.65 (13/20) | `runs/v0_B_base_gemini_20261110T174737486284.json` |
| v1 | `system_prompt.md` | Bỏ chỉ dẫn xấu của starter; bắt buộc gọi `clarify` khi thiếu account/URL và xin xác nhận trước khi gửi | 0.65 | 0.70 (14/20) | `runs/v1_B_base_gemini_20261110T181127878174.json` |
| v2 | `system_prompt.md` | Phân biệt tìm kiếm theo chủ đề (`social_search`) với thiếu account; cho phép gọi tool song song và ép `response_type: yes_no` | 0.70 | 0.80 (16/20) | `runs/v2_B_base_gemini_20261110T182200983550.json` |
| v3 | `tools.yaml` | Mô tả chi tiết khi nào dùng/không dùng và quy ước `query` chỉ chứa từ khóa cốt lõi (bỏ filler words) | 0.80 | 0.85 (17/20) | `runs/v3_B_base_openrouter_20261111T001429486980.json` |

### B2. Failure Analysis

Phân tích 7 ca FAIL ở baseline `v0` (`v0_B_base_gemini_20261110T174737486284.json`) và trạng thái khắc phục ở `v3`:

| Case ID | Failure Type | Actual Tool Calls (v0) | What Failed | Fix |
|---|---|---|---|---|
| `R03_web_news_routing` | `wrong_tool` / `wrong_arg_value` | `lookup` | `query` bị dài: `'AI news today artificial intelligence breakthroughs announcements'` thay vì `'AI'` | Quy ước query ngắn gọn trong `tools.yaml` + prompt; lưu ý trên `gpt-4o-mini` v3 còn gọi thừa thêm `social_search` |
| `R10_missing_handle` | `missing_info` | `timeline` | Tự đoán `sama` thay vì gọi `clarify` | Bỏ chỉ dẫn đoán bừa trong prompt (v1) → **PASS ở v1–v3** |
| `R11_missing_url` | `missing_info` | `lookup` | Tự gọi `lookup` thay vì `clarify` xin URL | Thêm rule thiếu URL gọi `clarify` (v1) → **PASS ở v1–v3** |
| `R12_confirm_before_send` | `wrong_boundary` | `send` | Tự ý gọi `send` không qua `clarify(yes_no)` | Bổ sung boundary xác nhận bắt buộc và `yes_no` (v2) → **PASS ở v2–v3** |
| `R13_parallel_web_and_tweets` | `wrong_tool` | `lookup` | Thiếu gọi song song `social_search`, `query` dư từ thừa | Kích hoạt parallel calling (v2) + query sạch (v3) → **PASS ở v3** |
| `M02_carryover_timeframe` | `wrong_arg_value` | `lookup` | `query` dính `'tin tức robotics hôm nay'` thay vì `'robotics'` | Chuẩn hóa quy ước `query` (v3); trên `gpt-4o-mini` v3 còn gọi kèm `social_search` |
| `M06_switch_tool` | `wrong_tool` | `lookup` | `query` dính `'OpenAI latest news'` thay vì `'OpenAI'` | Quy ước `query` sạch (v3); trên `gpt-4o-mini` v3 còn gọi kèm `social_search` |

### B3. Team Eval Cases

Liệt kê đúng 10 case trong `starter_v0/data/eval_group.json` (5 single turn + 5 multi turn). Cả 2 lần kiểm chứng cuối trên OpenRouter (`starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json`, commit `8987678`) và Google Gemini (`starter_v0/runs/v3_B_group_gemini_20261111T103806407842.json`, commit `5bdda64`) đều đạt **10/10 PASS = 100%** (`provider_error_cases == 0`):

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| `G01_topic_radar_monthly_robotics` | Định tuyến so sánh tin tức & X sang `topic_radar` với `timeframe=month`, `limit=4` | `topic_radar(query="robotics", timeframe="month", limit=4)` | **PASS** |
| `G02_evidence_finder_snippets` | Trích xuất claim, `max_sources=4`, tắt `deep_read` | `evidence_finder(claim=..., max_sources=4, deep_read=false)` | **PASS** |
| `G03_source_deduplicator_local` | Lọc trùng danh sách có sẵn ở chế độ local, không cào trang | `source_deduplicator(...)` | **PASS** |
| `G04_format_existing_sources_brief` | Định dạng trực tiếp nguồn có sẵn với `template="brief"` và đúng `headline` | `format(template="brief", headline="AI Governance Brief")` | **PASS** |
| `G05_rss_feed_reader_direct` | Định tuyến link RSS feed trực tiếp sang `rss_feed_reader` (không sửa prompt), giữ `url`, `max_items=4`, `keyword="safety"` | `rss_feed_reader(url="https://openai.com/news/rss.xml", max_items=4, keyword="safety")` | **PASS** |
| `GM01_source_deduplicator_local_followup` | Multi-turn: giữ lại danh sách nguồn ở turn 1 và tuân thủ lệnh không gọi extraction API ở turn 2 | `source_deduplicator(verify_content=false)` | **PASS** |
| `GM02_topic_radar_carryover_weekly` | Multi-turn: giữ chủ đề từ turn 1 sang turn 2 và áp dụng `timeframe=week`, `limit=3`, `enrich_articles=false` | `topic_radar(query="robotics", timeframe="week", limit=3, include_social=true, enrich_articles=false)` | **PASS** |
| `GM03_evidence_finder_carryover_claim` | Multi-turn: giữ nguyên `claim` ở turn 1 và áp dụng `max_sources=5`, `deep_read=false` | `evidence_finder(claim=..., max_sources=5, deep_read=false)` | **PASS** |
| `GM04_format_bullets_followup` | Multi-turn: định dạng `template="bullets"` và cập nhật `headline` theo lượt sau | `format(template="bullets", headline="AI Research Source Notes")` | **PASS** |
| `GM05_paper_text_page_limit_correction` | Multi-turn: giữ đúng arXiv ID `2307.09288` và cập nhật giới hạn `max_pages=3` | `paper_text(arxiv_url="2307.09288", max_pages=3)` | **PASS** |

### B4. Live Chat Evidence

Lấy từ 4 file transcript trong `starter_v0/transcripts/`:

| Turn | User Request | Tool Calls | Version Evidence | Outcome |
|---|---|---|---|---|
| Session 1 (`v3_gemini_20261110T193456801873.transcript.json`) Turn 1 | *"Find today's AI news and prepare a concise research brief with 5 sources."* | `lookup(query="AI", topic="news", timeframe="day")` → `format(template="brief")` | `v3+p8790bc017e58+t7486a61f88a9` | `answered` — Tổng hợp thành công bản tin AI 5 nguồn có trích dẫn |
| Session 2 (`v3_gemini_20261110T194637712792.transcript.json`) Turn 1–2 | Turn 1: *"Summarize the 5 latest tweets for me."* → Turn 2: *"Elon Musk."* | Turn 1: `clarify(response_type="text")` → Turn 2: `timeline(screenname="elonmusk", limit=5)` → `format` | `v3+p8790bc017e58+t7486a61f88a9` | `answered` — Hỏi lại khi thiếu tài khoản, sau đó lấy đúng 5 tweet của Elon Musk |
| Session 3 (`v3_gemini_20261110T194822685087.transcript.json`) Turn 1–2 | Turn 1: *"Post this research briefing to our Telegram channel."* → Turn 2: *"No. Do not send it."* | Turn 1: `clarify(response_type="yes_no")` → Turn 2: không gọi `send` | `v3+p8790bc017e58+t7486a61f88a9` | `answered` — Dừng gửi an toàn khi người dùng từ chối |
| Session 4 (`v3_gemini_20261110T235739095905.transcript.json`) Turn 3–4 | Turn 3: *"Gửi tóm tắt tin tức AI hôm nay lên Telegram giúp tôi"* → Turn 4: *"Tôi xác nhận gửi"* | Turn 3: `clarify(response_type="yes_no")` → Turn 4: `send(confirmed=true)` | `v3+p8790bc017e58+t7486a61f88a9` | `answered` — `status: "sent"` (gửi thật lên Telegram sau khi xác nhận) |

### B5a. Required Evidence — Tool mới, unit test, UI

| Item | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| Tool mới (`tools/<tên>/`) | `starter_v0/tools/topic_radar/`, `evidence_finder/`, `source_deduplicator/`, `rss_feed_reader/` (mỗi tool đủ `tool.py` + `TOOL.md`), đã đăng ký trong `tools/__init__.py` & `artifacts/tools.yaml` | Cả 4 tool hoạt động ổn định, hợp đồng đồng bộ 100% (`test_tool_contract.py` 75/75 passed) | Kiểm tra đầu vào chặt chẽ, reject URL/IP nội bộ (`localhost`, `.local`, `.internal`, IP private), chặn `DOCTYPE`/`ENTITY` (XML) và trả về error dict thay vì raise exception |
| Unit test tool mới (≥ 3 test/tool) | `starter_v0/tests/test_tools_todo.py`, `test_topic_radar.py`, `test_evidence_finder.py`, `test_source_deduplicator.py`, `test_rss_feed_reader.py` | Toàn suite `pytest starter_v0/tests/` đạt **159 passed, 0 failed, 0 skipped** trong 0.66s | Mọi HTTP request đều chạy offline hoàn toàn bằng `fake_http` & `monkeypatch`; chặn gọi mạng thật qua `block_network` |
| UI (lệnh chạy + link dùng thử) | `starter_v0/app.py`, `starter_v0/requirements.txt` | Lệnh chạy: `cd starter_v0 && python -m streamlit run app.py`. Giao diện sạch, hỗ trợ chọn đa provider (`openrouter`, `gemini`, `openai`, `anthropic`, `ollama`), mặc định `v3`, đọc `version_log.csv` và thực thi trực tiếp cả 14 tools (`0bc7e33`) | Tích hợp xác nhận người dùng trước khi `send` và không làm lộ API key lên giao diện |

- **Tool unit tests:** Nhóm đã triển khai toàn bộ test cho cả core tools và 4 tools mới với tổng số **159 test cases** (trong đó `test_topic_radar.py`: 16 tests, `test_evidence_finder.py`: 14 tests, `test_source_deduplicator.py`: 7 tests, `test_rss_feed_reader.py`: 9 tests, `test_tools_todo.py`: 31 tests sau khi Toàn bổ sung thêm 7 tests cho `send`/`papers`/`paper_text` ở commit `717a643`, `test_tools_examples.py`: 7 tests, `test_tool_contract.py`: 75 tests). Một ví dụ lỗi mà unit test bắt được nhưng eval không bắt được: trường hợp truyền tham số out-of-enum (`timeframe="decade"`), giới hạn vượt ngưỡng (`limit=99`), URL nội bộ (`http://localhost/secret`, `169.254.169.254` sau redirect), XML chứa `<!DOCTYPE`/`<!ENTITY>`, hoặc thiếu API key (`TAVILY_API_KEY`, `FIRECRAWL_API_KEY`) khiến code tool có thể văng `ValueError`/`TypeError`/`RuntimeError` nếu không được bọc trong `try...except` để trả về error dict chuẩn.

### B5. Bonus Evidence

| Bonus | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| Small Local Model | `starter_v0/runs/v3-local_B_base_ollama_*.json` (`qwen2.5:0.5b`, `3b`), `starter_v0/runs/v3-local_B_group_ollama_*.json` (`bf2053a`), `starter_v0/artifacts/version_log.csv` | Đã chạy đủ 2 model local Ollama (`qwen2.5:0.5b` & `qwen2.5:3b`) trên cả `eval_base.json` (`7/20 = 0.35` → **`16/20 = 0.80`**) lẫn `eval_group.json` (`3/10 = 0.30` → **`6/10 = 0.60`**), tất cả có `provider_error_cases = 0` | Hiện mới chạy baseline `v3-local` dùng chung prompt/tools với cloud (chưa tách vòng tối ưu prompt riêng cho local như mốc ≥ 2 vòng ở `RUBRIC.md`) |
| Hơn 3 tool mới (≥ 4) | Nhóm có **4 tool mới** (`topic_radar`, `evidence_finder`, `source_deduplicator`, `rss_feed_reader`), `eval_group.json` + `runs/v3_B_group_openrouter_20261111T091833288768.json` & `runs/v3_B_group_gemini_20261111T103806407842.json` | Đạt đủ chuẩn `TOOL.md`, đăng ký `tools/__init__.py` + `tools.yaml`, ≥ 3 test/tool ngay trong `test_tools_todo.py`, và `G05_rss_feed_reader_direct` đạt **PASS** (10/10 = 100%) dù không sửa `system_prompt.md` | Chặn URL nội bộ/SSRF, XML `DOCTYPE`/`ENTITY`, và tách rõ `DÙNG KHI / KHÔNG DÙNG KHI` với `fetch` và `lookup` trong `tools.yaml` |
| `send` gửi thật lên Telegram sau xác nhận | `starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json` (Turn 3–4) | Agent hỏi lại `clarify(yes_no)`, người dùng trả lời *"Tôi xác nhận gửi"*, agent gọi `send(confirmed=true)` và trả về `status: "sent"` trên private channel của nhóm | Bắt buộc cấu hình `TELEGRAM_BOT_TOKEN` & `TELEGRAM_CHAT_ID` trong `.env` (không commit); tuyệt đối chặn gửi khi `confirmed=false` |
| Test challenge (0 skipped / regression test) | Output chạy `pytest starter_v0/tests/` | Toàn bộ **159/159 tests passed, 0 skipped, 0 failed** (đã hoàn tất toàn bộ các stub TODO và đủ ≥ 3 test/tool trong `test_tools_todo.py`) | Đảm bảo 100% chạy offline nhờ fixture `block_network` |

### B7. Small Local Model Challenge (bonus)

Đối chiếu hiệu năng giữa các cloud models (`gpt-4o-mini`, `gemini-3.5-flash-lite`) và 2 local models qua Ollama (`qwen2.5:0.5b`, `qwen2.5:3b`) trên cấu hình `v3` (`provider_error_cases = 0` ở tất cả các run):

| Model / Provider | Suite | Case Accuracy | Tool Routing | Argument Acc | Multi-turn Acc | Run File |
|---|---|---:|---:|---:|---:|---|
| Cloud: `openai/gpt-4o-mini` (OpenRouter) | `eval_base` (20) | **0.85** (17/20) | 0.85 | 0.85 | 0.6667 | `runs/v3_B_base_openrouter_20261111T001429486980.json` |
| Local: `qwen2.5:3b` (Ollama) | `eval_base` (20) | **0.80** (16/20) | 0.85 | 0.80 | **0.8333** | `runs/v3-local_B_base_ollama_20261111T104735034855.json` |
| Local: `qwen2.5:0.5b` (Ollama) | `eval_base` (20) | **0.35** (7/20) | 0.45 | 0.35 | 0.1667 | `runs/v3-local_B_base_ollama_20261111T104458464803.json` |
| Cloud: `openai/gpt-4o-mini` (OpenRouter) | `eval_group` (10) | **1.00** (10/10) | 1.00 | 1.00 | 1.00 | `runs/v3_B_group_openrouter_20261111T091833288768.json` |
| Cloud: `gemini-3.5-flash-lite` (Gemini) | `eval_group` (10) | **1.00** (10/10) | 1.00 | 1.00 | 1.00 | `runs/v3_B_group_gemini_20261111T103806407842.json` |
| Local: `qwen2.5:3b` (Ollama) | `eval_group` (10) | **0.60** (6/10) | **1.00** | 0.60 | 0.40 | `runs/v3-local_B_group_ollama_20261111T101227317712.json` |
| Local: `qwen2.5:0.5b` (Ollama) | `eval_group` (10) | **0.30** (3/10) | 0.90 | 0.30 | 0.20 | `runs/v3-local_B_group_ollama_20261111T100206146391.json` |

- **Phân tích chênh lệch Cloud vs Local từ log thật:**
  - Trên `eval_base` (20 cases), nâng từ `qwen2.5:0.5b` (`0.35`) lên `qwen2.5:3b` (`0.80`) gần tiệm cận `gpt-4o-mini` (`0.85`), thậm chí `multiturn_accuracy` đạt `0.8333`; 4 lỗi còn lại của `3b` trên `eval_base` là `R06` (thiếu `query`), `R10` (đoán `sama` khi thiếu handle), `R14` (gọi `paper_text` cho câu hỏi ngoài phạm vi viết code) và `M06` (gọi `social_search` thay `lookup`).
  - Trên `eval_group` (10 cases) với 4 tools mới, `qwen2.5:3b` định tuyến đúng **10/10 (`tool_routing_accuracy = 1.00`)** nhưng mất điểm tham số ở `G04`/`GM04` (truyền thiếu `template` cho `format`), `GM02` (`include_social: false`) và `GM03` (thiếu `deep_read: false`). Lưu ý rằng cả 4 run local hiện mới chạy baseline `v3-local` (dùng nguyên prompt/tools của cloud, chưa có các vòng tinh chỉnh prompt riêng cho model nhỏ).

## 5. Khó khăn và cách xử lý

1. **Lỗi rate limit / quá tải của Gemini Free Tier (`429 RESOURCE_EXHAUSTED`, `503 UNAVAILABLE`):**
   - **Hiện tượng:** Khi chạy `run_eval.py` liên tiếp 20 cases bằng Gemini Free Tier (giới hạn 15 requests/phút), một số lượt sau (như `M02` ở lần thử v1, `R14` ở lần thử v3 lúc 18:29) gặp lỗi HTTP 429 hoặc 503 từ Google API (`v3_B_base_gemini_20261110T182951613132.json` bị `provider_error_cases = 1`).
   - **Cách xử lý:**
     + Bổ sung vòng lặp `retry` với `time.sleep` khi gặp lỗi `429`/`503`/`500` trong `starter_v0/providers/gemini_provider.py`.
     + Để đảm bảo các run nộp bài hoàn toàn sạch lỗi provider (`provider_error_cases == 0` và `measured_cases == 20/20`), nhóm chuyển sang chạy kiểm chứng v3 và group eval trên **OpenRouter (`openai/gpt-4o-mini`)** (lưu tại `v3_B_base_openrouter_20261111T001429486980.json` và `v3_B_group_openrouter_20261111T091833288768.json`).
2. **Lỗi giả (False Negatives) do ép kiểm tra giá trị mặc định & dấu câu trong `eval_group.json`:**
   - **Hiện tượng:** Lần chạy `eval_group` đầu trên OpenRouter (`v3_B_group_openrouter_20261111T012441145412.json`) đạt 5/10 do ép kiểm tra `enrich_articles: false` và `verify_content: false`; sau đó lần chạy đầu trên Gemini (`v3_B_group_gemini_20261111T102925310969.json`) đạt 8/10 do `G01` ép `include_social: true` (giá trị mặc định) và `GM03` so dấu chấm cuối `claim`.
   - **Cách xử lý:** Tuân thủ nguyên tắc chỉ assert tham số cốt lõi trong `expect` (`G01`, `G03`, `G05`, `GM03`) và làm rõ quy tắc `topic_radar` trong `system_prompt.md`. Sau chỉnh sửa cuối (`5bdda64`), cả OpenRouter (`v3_B_group_openrouter_20261111T091833288768.json`, `8987678`) và Gemini (`v3_B_group_gemini_20261111T103806407842.json`, `5bdda64`) đều đạt **10/10 (100%)**.
3. **Lỗi xung đột format CSV ở `version_log.csv`:**
   - Khi ghi cột `reason` và `hypothesis` chứa dấu phẩy nhưng thiếu ngoặc kép khiến cột bị nhảy; nhóm đã chuẩn hóa bằng chuỗi bọc dấu ngoặc kép theo chuẩn RFC 4180 và mã hóa UTF-8 Tiếng Việt.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong [TEAM.md](../../TEAM.md) và đầy đủ 5 báo cáo cá nhân `report/team/MSSV_HoTen.md`.

### 6.1. Nguyễn Duy Hưng (`2A202604110` — GitHub: `Hungnd1208`)
- **Báo cáo cá nhân:** [2A202604110_NguyenDuyHung.md](2A202604110_NguyenDuyHung.md)
- **Phần việc chính phụ trách (Prompt, Eval & Local Benchmark):**
  - Thực hiện vòng lặp tối ưu CP1–CP2 trên `eval_base` (20 cases): phân tích 7 ca lỗi của `v0` (`65%` — starter prompt ép đoán handle/URL, tự ý gọi `send`, ép 1 bước), tối ưu `system_prompt.md` ở `v1` (`70%`, sửa `R10`/`R11`), `v2` (`80%`, `100%` Tool Routing Accuracy nhờ phân biệt `social_search` với thiếu account, gọi song song & `response_type="yes_no"`), và chuẩn hóa `tools.yaml` ở `v3` (`85%`, `17/20` trên OpenRouter `gpt-4o-mini`, `provider_error_cases = 0`).
  - Thực hiện CP3–CP4 & Local Eval: đồng bộ khai báo 3 tool mới (`topic_radar`, `evidence_finder`, `source_deduplicator`) vào `tools/__init__.py` & `tools.yaml`, viết unit test ban đầu trong `tests/test_tools_todo.py` (đưa suite lên `134 passed`), khắc phục các false negative về tham số mặc định trong `eval_group.json` để nâng điểm từ `5/10` → `9/10` (`ac3e062`) → **10/10** (`8987678`), và chạy benchmark Ollama `qwen2.5:0.5b`/`3b` (`bf2053a`).
- **Phối hợp & hỗ trợ nhóm:**
  - Bổ sung cơ chế retry khi gặp `429`/`503` trong `starter_v0/providers/gemini_provider.py` và quản lý `version_log.csv` chuẩn UTF-8/RFC 4180 để nhóm đối chiếu và chạy UI (`v0`–`v3`, `v3-local`) nhất quán.
  - Biên tập khung `group_report.md` & `TEAM.md` (`45d93ab`) và chạy kiểm chứng lại `--suite group` đạt **10/10** sau khi tích hợp tool thứ 4 `rss_feed_reader` (`8987678`).
- **Bằng chứng & commits:** `system_prompt.md`, `tools.yaml`, `version_log.csv`, `gemini_provider.py`, 8 runs hợp lệ (`v0`–`v3`, `v3_B_group_openrouter_*`, `v3-local_B_group_ollama_*`); commits `463c8a9`, `17f136e`, `7ef6de9`, `d222de1`, `ac3e062`, `45d93ab`, `8987678`, `bf2053a`.

### 6.2. Nguyễn Văn Lâm (`2A202604115` — GitHub: `lam2121`)
- **Báo cáo cá nhân:** [2A202604115_NguyenVanLam.md](2A202604115_NguyenVanLam.md)
- **Phần việc chính phụ trách (UI, Báo cáo & Điều phối):**
  - Khởi tạo và nâng cấp giao diện Web UI bằng Streamlit (`starter_v0/app.py`, `starter_v0/requirements.txt`, hỗ trợ tunnel qua `starter_v0/cloudflared.exe`) với khả năng chọn đa provider (`openrouter`, `gemini`, `openai`, `anthropic`, `ollama`), mặc định `v3` và đọc `version_log.csv` (`0bc7e33`).
  - Tổng hợp và đồng bộ báo cáo nhóm (`report/team/group_report.md`), bảng phân công `TEAM.md`, báo cáo cá nhân (`report/team/2A202604115_NguyenVanLam.md`) và điều phối tiến độ/nộp bài của nhóm (`d80c157`, `ac2a337`, `d4c74a2`, `373a505`, `b51da75`, `0bc7e33`).
- **Phần hỗ trợ các thành viên khác:**
  - *Hỗ trợ Hưng (Prompt & Eval Engineer):* Đồng bộ và lưu lịch sử các phiên bản `system_prompt_v0.md`–`v3.md` (`ac2a337`, `d4c74a2`), chỉnh cấu hình mặc định trong `providers/gemini_provider.py` (`d80c157`).
  - *Hỗ trợ Dũng (New Tool Builder & Tool Contract):* Phát triển và tích hợp thêm tool thứ 4 `rss_feed_reader` (`tools/rss_feed_reader/tool.py`, `TOOL.md`, `tools/__init__.py`, `artifacts/tools.yaml`) mà không sửa `system_prompt.md` (`373a505`).
  - *Hỗ trợ Toàn (QA & Unit Test Engineer):* Bổ sung 3 tests chuẩn C4 cho `rss_feed_reader` và failure path cho `source_deduplicator` trong `tests/test_tools_todo.py`, cùng bộ test offline `tests/test_rss_feed_reader.py` (đưa suite lên 152 passed trước khi Toàn cộng thêm 7 core tool tests để đạt **159 passed**).
  - *Hỗ trợ Minh (Data & Live Chat Engineer):* Thực hiện phiên live chat gửi Telegram thật sau xác nhận (`starter_v0/transcripts/v3_gemini_20261110T235739095905.transcript.json`, `status: "sent"`) và cập nhật case `G05_rss_feed_reader_direct` trong `data/eval_group.json` (`373a505`).

### 6.3. Nguyễn Quốc Toàn (`2A202604098` — GitHub: `nguyenquoctoan0310`)
- **Báo cáo cá nhân:** [2A202604098_NguyenQuocToan.md](2A202604098_NguyenQuocToan.md)
- **Phần việc chính phụ trách (QA & Unit Test Engineer):**
  - Hoàn thiện 12 unit tests đầu tiên (`1a9490e`) theo format `test_tools_examples.py` cho 7 core tools trong `tests/test_tools_todo.py`: `clarify` (giữ `options` & `awaiting_user`), `format` (nhóm `section`, đánh số `thread`, an toàn khi `items=None`/`[]`), `lookup` (bỏ `time_range` khi `timeframe=None`), `fetch` (ánh xạ metadata Firecrawl & cắt `summary` đúng `4000` ký tự), `timeline` (bỏ tweet thiếu ID, giữ `limit` & URL `x.com`), `social_search` (`search_type="Top"`), và `policy` (query rỗng, lọc `policy_area`, tách prompt injection sang `untrusted_text`).
  - Bổ sung 7 unit tests cho `send`, `papers` & `paper_text` (`717a643`): kiểm tra `send(confirmed=true)` POST Telegram và trả `RuntimeError` (`calls == []`) khi thiếu env; `papers` parse Atom XML arXiv (`arxiv_id`, `authors`, `primary_category`) và xử lý HTTP `503`; `paper_text` mock tải PDF & `_extract_pdf_text`, chặn URL sai (`ValueError`, `calls == []`) và bắt `RequestException` — đưa `test_tools_todo.py` đạt **31 passed** và toàn suite đạt **159 passed, 0 failed, 0 skipped**.
- **Phối hợp & hỗ trợ nhóm:**
  - Bảo đảm 100% HTTP request được mock bằng `fake_http`/`monkeypatch` và chặn mạng thật qua `block_network` để toàn nhóm kiểm thử CI/offline an toàn không tốn quota API.
  - Phối hợp kiểm chứng ranh giới xác nhận `send` phục vụ luồng Telegram bonus của nhóm và rà soát/merge PR trên GitHub (`0a1538d`, `db43f54`, `700d68e`).
- **Bằng chứng & commits:** `starter_v0/tests/test_tools_todo.py` (19 core tests `L25-L447`; toàn file **31 passed**; toàn suite **159 passed**), đối chiếu `v3_gemini_20261110T235739095905.transcript.json`; commits/PR `1a9490e`, `0a1538d`, `717a643`, `db43f54`, `a8401b3`, `700d68e`.

### 6.4. Đặng Nguyễn Nhật Minh (`2A202604118` — GitHub: `MN-HD`)
- **Báo cáo cá nhân:** [2A202604118_DangNguyenNhatMinh.md](2A202604118_DangNguyenNhatMinh.md)
- **Phần việc chính phụ trách (Data & Live Chat Engineer):**
  - Thiết kế bộ 10 evaluation cases trong `starter_v0/data/eval_group.json` (5 single-turn `query` + 5 multi-turn `turns`, đủ schema `failure_type`, `expect`, `metadata.what_it_tests`) bao phủ 6 nhóm tình huống chưa có ở `eval_base.json`: `topic_radar` (`G01`, `GM02`), `evidence_finder` (`G02`, `GM03`), `source_deduplicator` (`G03`, `GM01` không gọi extraction API ở turn 2), `format` (`G04`, `GM04` đổi `headline`), `rss_feed_reader` (`G05`) và `paper_text` (`GM05` giữ `2307.09288` & sửa `max_pages=3`).
  - Thực hiện 3 phiên live chat trên Gemini `v3` (`193456801873` tìm tin AI & format; `194637712792` thiếu info → `clarify` giữ `limit=5` → `timeline("elonmusk", 5)` → `format`; `194822685087` yêu cầu gửi → `clarify(yes_no)` → từ chối `"No. Do not send it."` → hủy gửi an toàn) và tinh chỉnh Group Eval trên Gemini từ `8/10` (`v3_B_group_gemini_20261111T102925310969.json`, xử lý `G01` & `GM03`) lên **10/10 PASS (100%)** (`v3_B_group_gemini_20261111T103806407842.json`).
- **Phối hợp & hỗ trợ nhóm:**
  - Cung cấp bộ `eval_group.json` chuẩn làm thước đo chung cho cả nhóm kiểm chứng xuyên suốt trên OpenRouter, Gemini và Ollama local.
  - Kiểm thử lại toàn bộ test suite sau khi đồng bộ code mới nhất (`pytest -q`: **159 passed, 0 failed**) và push đầy đủ dataset, 3 transcripts cùng 2 runs Gemini lên `main`.
- **Bằng chứng & commits:** `starter_v0/data/eval_group.json`, 3 transcripts (`193456801873`, `194637712792`, `194822685087`), 2 runs Gemini (`8/10` → `10/10`); commits `1633c37`, `1fc7bbc`, `5bdda64`, `59c3a45`.

### 6.5. Nguyễn Trung Dũng (`2A202604084` — GitHub: `bomgoa1102`)
- **Báo cáo cá nhân:** [2A202604084_NguyenTrungDung.md](2A202604084_NguyenTrungDung.md)
- **Phần việc chính phụ trách (New Tool Builder & Unit Test):**
  - Phát triển hoàn chỉnh 3 tools nghiên cứu mới (CP3) kèm xử lý trả error dict có cấu trúc: `source_deduplicator` (chuẩn hóa URL bỏ `utm_*`/`fbclid`/`gclid`, so sánh tiêu đề, mặc định lọc cục bộ và khi bật `verify_content` sẽ đọc qua Firecrawl rồi fallback Tavily Extract), `evidence_finder` (tìm nguồn đa hostname qua Tavily, loại URL nội bộ, tùy chọn `deep_read` Firecrawl → Tavily Extract, chấm `relevance_score` theo từ khóa và giữ `assessment: requires_review`) và `topic_radar` (gộp Tavily News + X RapidAPI, thống kê hostname & từ khóa giao thoa, giới hạn làm giàu tối đa 2 bài, trả `partial` khi một bên lỗi).
  - Xây dựng 3 bộ unit test mock HTTP offline kiểm tra đủ 3 nhóm tình huống (happy path, input xấu/URL nội bộ/out-of-range, thiếu key & lỗi nhà cung cấp): `test_source_deduplicator.py` (7 tests), `test_evidence_finder.py` (14 tests), `test_topic_radar.py` (16 tests) — tổng **37 passed**.
- **Phối hợp & hỗ trợ nhóm:**
  - Viết chi tiết đặc tả chức năng, tham số và giới hạn trong `TOOL.md` của cả 3 tools làm cơ sở để nhóm đăng ký vào `tools/__init__.py`, `artifacts/tools.yaml` và đưa vào 6 tình huống của `eval_group.json` (`G01`–`G03`, `GM01`–`GM03`, đều đạt 10/10 ở run cuối).
  - Kiểm thử lại toàn bộ test suite (`159 passed, 0 failed`) và làm rõ ranh giới đóng góp từng checkpoint trong báo cáo cá nhân.
- **Bằng chứng & commits:** `starter_v0/tools/{source_deduplicator,evidence_finder,topic_radar}/` (`tool.py`, `TOOL.md`), `tests/test_source_deduplicator.py`, `test_evidence_finder.py`, `test_topic_radar.py` (`37 passed`); commits `771be66`, `41f3b77`, `93e21c9`, `38274ee`.

## 7. Hạn chế và hướng cải thiện — B6. Reflection

- **Fix nào thuộc `system_prompt.md`?**
  - Loại bỏ chỉ dẫn xấu ép đoán bừa và tự ý gửi, quy tắc bắt buộc `clarify` khi thiếu thông tin/chưa xác nhận, quy tắc gọi song song nhiều nguồn, và các quy ước ứng xử đa lượt (multi-turn carryover/correction).
- **Fix nào thuộc `tools.yaml`?**
  - Mô tả phạm vi "Khi nào dùng / Khi nào KHÔNG dùng" của từng tool để chống nhầm lẫn giữa các công cụ tương đồng (`timeline` vs `social_search`, `lookup` vs `fetch`, `topic_radar` vs `lookup`, `rss_feed_reader` vs `fetch`/`lookup`), quy ước format tham số (`screenname` không `@`, `query` không filler words, `confirmed=true` cho `send`).
- **Failure nào cần review thủ công thay vì chấm tự động?**
  - Các trường hợp từ khóa có nhiều cách diễn đạt tương đương trong `lookup`/`social_search`, tính tự nhiên/rõ ràng của câu hỏi `clarify`, và chất lượng tổng hợp nội dung/trích dẫn của câu trả lời cuối cùng sau khi gọi tool xong.
- **Nếu có thêm một vòng, nhóm sẽ thử giả thuyết nào?**
  - Thiết lập cơ chế ràng buộc trên `gpt-4o-mini` để khắc phục hoàn toàn việc gọi thừa tool `social_search` khi người dùng chỉ yêu cầu tin tức báo chí ở `eval_base.json` (`R03`, `M02`, `M06`), đồng thời bổ sung ≥ 2 vòng tối ưu prompt riêng cho Small Local Model (`qwen2.5:3b`/`0.5b`) nhằm khắc phục các lỗi thiếu tham số (`template`, `deep_read`, `query`).

## 8. Tài liệu tham khảo

- Tài liệu hướng dẫn đề bài: [README.md](../../README.md), [RULES.md](../../RULES.md), [RUBRIC.md](../../RUBRIC.md), [CHECKPOINTS.md](../../CHECKPOINTS.md).
- Công cụ hỗ trợ: Trợ lý AI (Antigravity / ChatGPT / Gemini) hỗ trợ phân tích log JSON, cấu trúc hóa YAML và sinh mẫu unit test mock.
