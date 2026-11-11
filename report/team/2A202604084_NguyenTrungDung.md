# Báo cáo cá nhân - Nguyễn Trung Dũng

## 1. Thông tin cá nhân

Họ tên: Nguyễn Trung Dũng
MSSV: 2A202604084
Lớp: K5 - L3B
Nhóm: Nhóm Bá Khí (NhomBaKhi)
GitHub: bomgoa1102
Vai trò: Phát triển công cụ mới (New Tool Builder), viết tài liệu sử dụng và unit test cho các công cụ mở rộng của Research Agent.

## 2. Phần việc theo các checkpoint

CP1 - Setup và baseline v0: Phần khởi tạo repo, cấu hình provider và chạy baseline do thành viên phụ trách prompt/eval thực hiện. Tôi sử dụng kết quả baseline để nắm tình trạng ban đầu của agent, không nhận đây là phần mình trực tiếp chạy. Run v0 lưu tại `starter_v0/runs/v0_B_base_gemini_20261110T174737486284.json` có 13/20 case đạt, không có lỗi provider. Các lỗi chính liên quan đến chọn sai tool, thiếu thông tin đầu vào, sai tham số và xử lý chưa đúng bước xác nhận.

CP2 - Tối ưu prompt và tool declaration: Việc sửa `system_prompt.md`, `tools.yaml` qua v1-v3 và quản lý `version_log.csv` do thành viên khác phụ trách. Tôi tập trung làm rõ chức năng, tham số và trường hợp sử dụng của các tool mới trong `TOOL.md` để nhóm có cơ sở khai báo chúng vào registry. Các kết quả base eval đã lưu lần lượt là 14/20 ở v1, 16/20 ở v2 và 17/20 ở v3 OpenRouter. Đây là kết quả chung của nhóm, không phải số liệu tôi tự chạy.

CP3 - Tool mới và unit test: Đây là công việc chính của tôi. Tôi xây dựng ba công cụ `source_deduplicator`, `evidence_finder`, `topic_radar`; mỗi công cụ có `tool.py`, `TOOL.md` và file kiểm thử sử dụng mock. Mục tiêu là bổ sung những thao tác phục vụ nghiên cứu nguồn tin mà các tool có sẵn chưa giải quyết trực tiếp: bỏ nguồn trùng, tìm tài liệu liên quan đến một nhận định và đối chiếu tin báo chí với thảo luận trên X. Các tool được viết để trả kết quả có cấu trúc; khi đầu vào sai hoặc API lỗi thì trả về thông tin lỗi thay vì làm agent dừng đột ngột.

CP4 - Eval nhóm 10 case: Tôi không phụ trách soạn toàn bộ bộ eval hay chạy provider, nhưng các tool tôi viết được đưa vào sáu tình huống của `starter_v0/data/eval_group.json`: G01 và GM02 dùng `topic_radar`; G02 và GM03 dùng `evidence_finder`; G03 và GM01 dùng `source_deduplicator`. Bộ dữ liệu nhóm gồm 5 case một lượt và 5 case nhiều lượt. Run `v3_B_group_openrouter_20261111T091833288768.json` đạt 10/10 case, với `provider_error_cases = 0` và `measured_cases = total_cases = 10`. Đây là kết quả kiểm tra cách agent chọn tool và truyền tham số trên bộ case nhóm; nó không thay thế cho việc kiểm thử code từng tool.

CP5 - UI, live chat và báo cáo: Phần Streamlit UI, transcript live chat, gửi Telegram và báo cáo nhóm do các thành viên được phân công đảm nhiệm. Phần của tôi trong checkpoint này là cung cấp mã nguồn, tài liệu, unit test và bằng chứng commit của ba tool để nhóm tổng hợp, đồng thời hoàn thành báo cáo cá nhân này. Tôi không ghi nhận UI, transcript hay các run của thành viên khác là đóng góp kỹ thuật của mình.

## 3. Công việc đã thực hiện và cách xử lý

Với `source_deduplicator`, tôi muốn xử lý tình huống agent tìm được nhiều kết quả nhưng thực ra chúng dẫn đến cùng một bài viết. Tool nhận danh sách nguồn và lọc trùng theo URL đã chuẩn hóa, bỏ các tham số theo dõi như `utm_*`, `fbclid`, `gclid`. Nếu URL khác nhau, tool còn so sánh mức tương đồng của tiêu đề. Khi bật `verify_content`, công cụ mới đọc nội dung để đối chiếu sâu hơn, ưu tiên Firecrawl và chuyển sang Tavily Extract nếu cần. Chế độ mặc định chỉ lọc cục bộ, không gọi API nên phù hợp khi không cần tốn thêm lượt truy cập. Kết quả trả lại danh sách đã lọc, số mục bị loại và kiểu kiểm tra.

Với `evidence_finder`, đầu vào là một nhận định cần tìm tài liệu liên quan. Tool tìm kiếm bằng Tavily, bỏ URL không hợp lệ, chọn kết quả từ các hostname khác nhau rồi lấy đoạn nội dung có liên quan. Nếu bật `deep_read`, tool thử đọc toàn trang bằng Firecrawl, sau đó có thể dùng Tavily Extract khi cách đầu không thành công. Tôi giữ trường `relevance_score` để hỗ trợ sắp xếp đoạn trích, nhưng điểm này chỉ dựa vào sự trùng khớp từ khóa, không thể xem là kết luận nhận định đúng hay sai. Output vì vậy để `assessment` ở mức `requires_review` thay vì tự động phán quyết.

Với `topic_radar`, tôi kết hợp kết quả tin tức từ Tavily News và bài đăng trên X thông qua RapidAPI Twitter API45. Tool chuẩn hóa các nguồn vào một danh sách, thống kê số bài báo, số hostname báo chí, số bài đăng X và các từ khóa xuất hiện ở cả hai nhóm kết quả. Nếu một nguồn gặp lỗi nhưng nguồn còn lại vẫn hoạt động, tool trả trạng thái `partial` cùng cảnh báo. Tôi cũng giới hạn số lượng kết quả và số bài được làm giàu nội dung để tránh gọi API quá nhiều. Kết quả này chỉ phản ánh mẫu dữ liệu lấy được, không dùng để khẳng định một chủ đề đang nổi tiếng trên toàn thị trường.

Trong quá trình viết test, tôi tập trung vào ba dạng tình huống: đầu vào hợp lệ, đầu vào sai và lỗi từ dịch vụ bên ngoài. Các bài test dùng mock HTTP, nên có thể kiểm tra logic mà không cần API key thật. Riêng `source_deduplicator` có kiểm tra loại URL chứa tracking, tiêu đề gần trùng và trường hợp cần fallback khi đọc nội dung. `evidence_finder` có kiểm tra việc giữ các domain khác nhau, URL nội bộ không hợp lệ, kết quả rỗng và lỗi của hai nhà cung cấp. `topic_radar` có kiểm tra kết hợp hai nguồn, khi một nguồn không có key hoặc bị lỗi, làm giàu tối đa hai bài và kiểm tra tham số đầu vào.

## 4. Bằng chứng và kết quả kiểm tra

Các commit kỹ thuật trong lịch sử repo gắn với tài khoản `bomgoa1102` gồm:

- `771be66` - thêm `source_deduplicator/tool.py` và `source_deduplicator/TOOL.md`.
- `41f3b77` - thêm `tests/test_source_deduplicator.py` với các trường hợp kiểm tra mock.
- `93e21c9` - thêm `topic_radar` và `evidence_finder`, mỗi tool có mã nguồn, tài liệu và file test riêng.

Những file chính để đối chiếu là `starter_v0/tools/source_deduplicator/`, `starter_v0/tools/evidence_finder/`, `starter_v0/tools/topic_radar/` và ba file tương ứng `starter_v0/tests/test_source_deduplicator.py`, `test_evidence_finder.py`, `test_topic_radar.py`. Phần đăng ký các tool vào `starter_v0/tools/__init__.py`, khai báo trong `starter_v0/artifacts/tools.yaml` và các test tích hợp trong `tests/test_tools_todo.py` đã được nhóm đồng bộ; tôi không nhận toàn bộ phần tích hợp này là commit của mình.

Tôi đối chiếu bộ source và chạy lại kiểm thử offline trên bản repo được cung cấp. Lệnh `python -m pytest -q tests/test_topic_radar.py tests/test_evidence_finder.py tests/test_source_deduplicator.py` cho kết quả 37 passed. Lệnh `python -m pytest -q` tại thư mục `starter_v0` cho kết quả 159 passed, không có test failed. Các kết quả này chứng minh unit test hiện chạy được trong môi trường kiểm tra, nhưng không có nghĩa toàn bộ API bên ngoài đã được kiểm thử trực tiếp.

Ở mức agent, bằng chứng liên quan là `starter_v0/data/eval_group.json` và run `starter_v0/runs/v3_B_group_openrouter_20261111T091833288768.json`. Các case của ba tool có trong bộ kiểm tra và run nhóm đạt 10/10 như đã nêu. Việc thiết kế, điều chỉnh bộ eval và chạy provider là công việc chung có phân công riêng; tôi chỉ dùng kết quả để cho thấy tool đã được kết nối vào luồng đánh giá.

## 5. Công cụ hỗ trợ, tự đánh giá và hạn chế

Trong quá trình hoàn thiện báo cáo cá nhân, tôi sử dụng ChatGPT để rà soát yêu cầu của bài, đối chiếu file mã nguồn, lịch sử commit, test và run JSON, đồng thời hỗ trợ diễn đạt báo cáo cho rõ ý. Những số liệu ghi ở đây lấy từ file trong repo và từ kết quả chạy kiểm thử, không tự đặt ra kết quả thực nghiệm.

Điểm tôi thấy làm được là bổ sung ba công cụ có nhiệm vụ khác nhau nhưng có thể kết hợp trong quy trình nghiên cứu nguồn tin: tìm tài liệu, đối chiếu thông tin từ nhiều kênh rồi loại nguồn trùng trước khi trình bày. Mỗi tool đều có tài liệu sử dụng, kiểm tra tham số và kiểm thử bằng mock cho cả trường hợp thành công lẫn lỗi. Việc xử lý lỗi theo dạng dữ liệu trả về cũng giúp agent dễ tiếp tục hội thoại hơn.

Hạn chế là các unit test chủ yếu chạy offline nên chưa chứng minh được chất lượng dữ liệu của Tavily, Firecrawl và RapidAPI trong mọi trường hợp thực tế. `evidence_finder` mới chấm mức liên quan theo từ khóa, còn `topic_radar` chỉ so sánh một mẫu tin và bài đăng; cả hai đều cần người dùng kiểm tra nguồn trước khi đưa ra kết luận. Nếu có thời gian phát triển tiếp, tôi sẽ ưu tiên thêm kiểm thử với dữ liệu thực có kiểm soát và cải thiện cách đánh giá chất lượng nguồn.

Báo cáo này chỉ ghi phần đóng góp liên quan đến ba tool và các bằng chứng đi kèm. Nội dung về kết quả prompt v0-v3, bộ eval chung, giao diện và live chat được nhắc đến để làm rõ mối liên hệ giữa các checkpoint, không nhằm nhận thay phần việc của thành viên khác.
