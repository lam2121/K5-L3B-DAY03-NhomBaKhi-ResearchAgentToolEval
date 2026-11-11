# Báo cáo cá nhân — Đặng Nguyễn Nhật Minh (MSSV: 2A202604118)

## 1. Thông tin thành viên

- Họ tên: Đặng Nguyễn Nhật Minh
- MSSV: 2A202604118
- Lớp: K5 - L3B
- Nhóm: Nhóm Bá Khí (`NhomBaKhi`)
- GitHub: MN-HD
- Vai trò: Data & Live Chat Engineer

## 2. Phần việc phụ trách

Phụ trách thiết kế bộ dữ liệu đánh giá mở rộng của nhóm và kiểm thử Research Agent thông qua các phiên hội thoại thực tế nhiều lượt.

Các nhiệm vụ chính:

- Xây dựng 10 evaluation cases trong `starter_v0/data/eval_group.json`, gồm 5 single-turn và 5 multi-turn.
- Thiết kế các tình huống kiểm tra định tuyến tool, truyền arguments, xử lý thông tin thiếu, duy trì ngữ cảnh hội thoại và tuân thủ giới hạn an toàn.
- Thực hiện ít nhất 3 phiên live chat với agent phiên bản v3 và lưu transcript JSON làm bằng chứng.
- Chạy group evaluation, phân tích các trường hợp FAIL, điều chỉnh các kỳ vọng đánh giá chưa phù hợp và kiểm chứng lại kết quả.
- Đồng bộ và push các file evaluation, transcript và run JSON lên GitHub.

## 3. Đóng góp cụ thể

### 3.1. Xây dựng bộ 10 evaluation cases

Thiết kế bộ 10 test cases mở rộng nhằm kiểm tra những tình huống mà bộ `eval_base.json` chưa bao phủ đầy đủ.

Bộ test gồm:
- 5 single-turn cases sử dụng trường `query`.
- 5 multi-turn cases sử dụng trường `turns`.

Các tình huống tập trung vào:
- `topic_radar`: So sánh tin tức báo chí và thảo luận trên X theo chủ đề, thời gian và giới hạn kết quả.
- `evidence_finder`: Tìm nguồn bằng chứng cho một claim và kiểm soát việc trích xuất nội dung sâu.
- `source_deduplicator`: Loại bỏ nguồn trùng lặp bằng URL và tiêu đề, đồng thời tuân thủ yêu cầu không gọi extraction API.
- `format`: Định dạng dữ liệu đã thu thập thành research brief hoặc bullet points mà không tìm kiếm lại.
- `rss_feed_reader`: Kiểm tra định tuyến trực tiếp tới RSS feed khi người dùng cung cấp URL.
- `paper_text`: Kiểm tra việc duy trì arXiv ID và giới hạn số trang qua nhiều lượt hội thoại.

Các case được tổ chức theo đúng schema của evaluator, có `failure_type`, `expect` và `metadata.what_it_tests`.

### 3.2. Thực hiện live chat và lưu transcript

Thực hiện 3 phiên hội thoại thực tế với Gemini provider và system prompt v3.

**Phiên 1 — Normal research**

Yêu cầu agent tìm tin tức AI trong ngày và chuẩn bị research brief với 5 nguồn.

Kết quả: Agent gọi `lookup` để tìm kiếm tin tức, sau đó gọi `format` để tổng hợp thành bản tin có phân loại chủ đề và liên kết nguồn.

Transcript:
`v3_gemini_20261110T193456801873.transcript.json`

**Phiên 2 — Missing information và multi-turn context**

Lượt 1: Yêu cầu agent tóm tắt 5 tweet mới nhất nhưng không cung cấp tài khoản.

Lượt 2: Bổ sung tài khoản Elon Musk.

Kết quả: Agent gọi `clarify` để hỏi tài khoản, ghi nhớ số lượng tweet cần lấy, sau đó gọi `timeline(screenname="elonmusk", limit=5)` và `format`.

Transcript:
`v3_gemini_20261110T194637712792.transcript.json`

**Phiên 3 — Safety và confirmation**

Lượt 1: Yêu cầu agent đăng research briefing lên Telegram.

Lượt 2: Từ chối xác nhận bằng câu "No. Do not send it."

Kết quả: Agent gọi `clarify(response_type="yes_no")` trước khi thực hiện hành động. Khi người dùng từ chối, agent hủy thao tác và không gọi `send`.

Transcript:
`v3_gemini_20261110T194822685087.transcript.json`

### 3.3. Chạy và cải thiện group evaluation

Thực hiện group evaluation trên Gemini với prompt và tool declarations phiên bản v3.

**Lần đánh giá đầu tiên:**
- 10 cases được thực thi đầy đủ.
- 8/10 cases PASS, đạt 80%.
- Không có provider error.
- Phát hiện hai trường hợp đánh giá quá nghiêm ngặt:
  - `G01`: Evaluator yêu cầu model truyền `include_social=true`, trong khi tool đã có giá trị mặc định là `true`.
  - `GM03`: Evaluator so sánh dấu câu cuối claim một cách tuyệt đối, mặc dù nội dung claim được bảo toàn.

**Cách xử lý:**

- Điều chỉnh expected arguments của `G01` để chỉ kiểm tra các tham số cốt lõi, không ép model truyền tham số mặc định.
- Chuẩn hóa claim trong `GM03` để evaluator không nhầm khác biệt dấu câu với lỗi ghi nhớ nội dung.

**Lần đánh giá cuối cùng:**

- Tổng số cases: 10.
- Cases PASS: 10.
- Case accuracy: 100%.
- Tool routing accuracy: 100%.
- Argument accuracy: 100%.
- Multi-turn accuracy: 100%.
- Provider errors: 0.

Run JSON:
`v3_B_group_gemini_20261111T103806407842.json`

### 3.4. Kiểm thử và quản lý mã nguồn

Sau khi đồng bộ code mới nhất từ nhóm, thực hiện toàn bộ unit test:

`python -m pytest -q`

Kết quả: **159 passed in 1.17s**, không có test thất bại.

Các đóng góp được commit và push lên nhánh `main`:

- `1633c37`: Add 10 group evaluation cases.
- `1fc7bbc`: Add three successful live chat transcripts.
- `5bdda64`: Finalize group evaluation and record Gemini results.

## 4. Bằng chứng

**Evaluation dataset:**
- `starter_v0/data/eval_group.json`

**Live chat transcripts:**
- `starter_v0/transcripts/v3_gemini_20261110T193456801873.transcript.json`
- `starter_v0/transcripts/v3_gemini_20261110T194637712792.transcript.json`
- `starter_v0/transcripts/v3_gemini_20261110T194822685087.transcript.json`

**Evaluation runs:**
- `starter_v0/runs/v3_B_group_gemini_20261111T102925310969.json` — lần đánh giá đầu tiên, 8/10 (80%).
- `starter_v0/runs/v3_B_group_gemini_20261111T103806407842.json` — lần đánh giá cuối cùng, 10/10 (100%).

**GitHub commits:**
- `1633c37`
- `1fc7bbc`
- `5bdda64`

Repository:
https://github.com/lam2121/K5-L3B-DAY03-NhomBaKhi-ResearchAgentToolEval

## 5. Tự đánh giá

### Những gì đã hoàn thành

- Hoàn thành bộ 10 evaluation cases theo đúng yêu cầu 5 single-turn và 5 multi-turn.
- Thực hiện thành công 3 phiên live chat với các tình huống khác nhau.
- Phân tích và xử lý hai false failures trong group evaluation.
- Đạt 10/10 cases PASS trong lần kiểm chứng cuối cùng.
- Lưu và push đầy đủ dataset, transcript và kết quả đánh giá lên GitHub.

### Khó khăn và cách xử lý

Trong quá trình kiểm thử, gặp các vấn đề cấu hình API riêng biệt cho Gemini, Tavily và RapidAPI. Đã xử lý bằng cách cấu hình đúng API key cho từng provider/tool và kiểm tra lại kết nối trước khi chạy live chat.

Ngoài ra, lần đánh giá đầu tiên có hai trường hợp FAIL do evaluator kiểm tra quá cứng các giá trị mặc định và dấu câu. Sau khi đối chiếu expected arguments với hành vi thực tế của tool, đã điều chỉnh expected values để đánh giá đúng mục tiêu của test case.

### Công cụ AI đã sử dụng và cách kiểm chứng

Sử dụng ChatGPT để:
- Hỗ trợ đọc hiểu cấu trúc repository và yêu cầu của từng checkpoint.
- Hỗ trợ thiết kế evaluation cases và kiểm tra cấu trúc JSON.
- Phân tích log JSON, xác định nguyên nhân các trường hợp FAIL.
- Hướng dẫn xử lý lỗi cấu hình API và Git.
- Hỗ trợ diễn giải kết quả unit test và group evaluation.

Các kết quả do AI hỗ trợ đều được đối chiếu với mã nguồn thực tế, tool schemas, terminal output, JSON evaluation runs và lịch sử commit trên GitHub. Không sử dụng AI để tạo giả kết quả chạy hoặc transcript.

### Bài học kinh nghiệm

Qua nhiệm vụ Data & Live Chat Engineer, tôi hiểu rõ hơn sự khác biệt giữa việc kiểm tra khả năng định tuyến của model và kiểm tra logic thực thi của tool.

Tôi cũng học được tầm quan trọng của việc thiết kế evaluation case đúng mục tiêu: chỉ kiểm tra những tham số thực sự cần thiết, không đánh giá sai một hành vi đúng chỉ vì model bỏ qua giá trị mặc định hoặc khác biệt hình thức nhỏ.

Các phiên live chat giúp tôi kiểm chứng hành vi thực tế của agent, đặc biệt là khả năng hỏi lại khi thiếu thông tin, duy trì ngữ cảnh nhiều lượt và yêu cầu xác nhận trước khi thực hiện hành động có tác động bên ngoài.