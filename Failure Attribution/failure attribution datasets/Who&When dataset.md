# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Bài báo nằm trong bối cảnh phát triển hệ LLM multi-agent, nơi kết quả đánh giá cần được nối với việc xác định thành phần phải cải thiện. Bối cảnh chung về vòng lặp đánh giá–cải thiện và giới hạn của benchmark/LLM-as-a-judge đã được trình bày tại [Bối cảnh chung](<Bối cảnh chung.md>).

Khoảng trống mà nghiên cứu này tập trung là: sau khi một hệ multi-agent thất bại trên một kịch bản đánh giá, việc chuyển từ kết quả thất bại sang kết luận **thành phần nào trực tiếp gây ra thất bại** vẫn chủ yếu do con người làm. Các benchmark chi tiết hơn chỉ bổ sung metric để tham chiếu; chúng không tự động ánh xạ kết quả benchmark tới thành phần cần cải thiện. Do đó, câu hỏi mà bài báo nêu ra là: **thành phần nào của hệ agentic cần được cải thiện?**

Tác giả đề xuất và phát biểu một bài toán nghiên cứu mới, *automated failure attribution*: khi hệ thống thất bại trong một kịch bản đánh giá, tự động xác định thành phần chịu trách nhiệm mà không cần con người can thiệp. Phần problem formulation của hướng nghiên cứu này được trình bày tại [[Failure Attribution - Formulation]].

## 1.2. Vì sao các hướng trước chưa giải quyết bài toán

Theo bài báo, các benchmark chi tiết hơn không giải quyết được failure attribution vì metric vẫn chỉ là điểm tham chiếu và việc quy kết từ kết quả benchmark vẫn là thủ công. Các nghiên cứu LLM-as-a-judge có thể dùng LLM để giảm công sức đánh giá, nhưng trong agentic systems, failure attribution vẫn bị để lại như một bước thủ công. Các reward model và nghiên cứu đánh giá tiến trình chủ yếu đánh giá đầu ra hoặc các bước suy luận của một LLM đơn lẻ, thay vì xác định lỗi trong hệ agentic phức tạp.

Vì vậy, khoảng trống là thiếu một cơ chế tự động quy kết thất bại trong hệ multi-agent, thay vì chỉ thiếu thêm một thước đo kết quả cuối hoặc bộ phát hiện bước sai.

# 2. The Who&When Dataset

## 2.1. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

Who&When gồm 184 tác vụ gán nhãn trên log thất bại của 127 hệ LLM multi-agent. Truy vấn mẫu lấy từ validation set của hai benchmark GAIA và AssistantBench. Tác giả tạo/thu log bằng hai framework hệ tác tử:

- **AG2 / CaptainAgent:** CaptainAgent tự động xây dựng một đội tác tử cho từng truy vấn, gồm tên tác tử, prompt và công cụ phù hợp với tác vụ; sau đó tối ưu cấu hình theo vòng lặp. Tác giả lấy cấu hình multi-agent cuối cùng cùng lịch sử chạy tương ứng.
- **AutoGen / Magentic-One:** Magentic-One là hệ multi-agent được thiết kế thủ công, gồm năm tác tử có các năng lực khác nhau. Tác giả chạy hệ này trên validation set AssistantBench và trên một mẫu ngẫu nhiên 30 truy vấn GAIA để thu log thất bại.

Với cả hai nguồn, Who&When chỉ giữ các lần chạy không giải được truy vấn, vì mục tiêu dataset là thu thập những sai sót của tác tử dẫn đến thất bại khi giải các vấn đề thực tế. Mỗi mẫu gồm truy vấn, toàn bộ failure log, thông tin hệ thống (prompt, công cụ và tên tác tử đối với hệ sinh bằng thuật toán), cùng nhãn và giải thích bằng ngôn ngữ tự nhiên.

## 2.2. Các bước sinh và gán nhãn

1. **Chọn truy vấn và tạo/chạy hệ tác tử.** Tác giả dùng truy vấn từ GAIA và AssistantBench. Với CaptainAgent, hệ tự tạo đội tác tử theo từng truy vấn; với Magentic-One, tác giả chạy hệ thiết kế thủ công trên các tập truy vấn đã nêu.
2. **Thu và lọc failure log.** Tác giả lấy lịch sử chạy của hệ, nhưng chỉ giữ những lần không giải được truy vấn. Với Magentic-One, phần GAIA ngoài mẫu 30 truy vấn bị loại vì log ngữ cảnh dài làm gán nhãn quá phức tạp.
3. **Tạo mẫu gán nhãn.** Mỗi failure log được ghép với truy vấn và thông tin hệ thống cần thiết; nhãn cần xác định tác tử chịu trách nhiệm, bước lỗi quyết định và lý do lỗi.
4. **Gán nhãn độc lập.** Ba chuyên gia về AI agent được chia các log, dùng cùng một hướng dẫn chuẩn để gán nhãn ba thành phần trên, đồng thời đánh dấu nhãn chắc chắn hoặc còn không chắc chắn.
5. **Thảo luận các nhãn không chắc chắn.** Các chuyên gia thảo luận để đạt đồng thuận; tác giả không chỉ áp dụng nguyên tắc đa số mà yêu cầu những người tham gia được thuyết phục về kết luận.
6. **Kiểm tra chéo và gán nhãn lại nếu cần.** Mỗi chuyên gia rà soát nhãn của chuyên gia khác. Nếu phát hiện khác biệt về chuẩn gán nhãn, họ tiếp tục thảo luận và gán nhãn lại cho tới khi thống nhất.

## 2.3. Mục đích và đánh đổi của thiết kế

Tác giả đặc biệt chỉ ra giới hạn của gán nhãn thủ công. Người gán nhãn phải đọc log phức tạp, theo dõi logic giải bài của từng tác tử và đánh giá liệu một hành động có làm chệch toàn bộ quá trình hay không; khi dùng công cụ web, họ còn có thể phải kiểm tra lịch sử duyệt và các trang đã truy cập. Một lần chạy có thể có nhiều tác tử mắc lỗi, còn mức độ nghiêm trọng của lỗi đôi khi tinh tế và mang tính chủ quan.

Các phê bình này lý giải hai lựa chọn thiết kế của tác giả: loại phần lớn GAIA của Magentic-One do log quá dài, và dùng ba vòng gán nhãn có thảo luận/kiểm tra chéo thay vì một nhãn đơn lẻ. Bài báo báo cáo ba người gán nhãn đã dùng lần lượt 30,9; 30,2; và 23,2 giờ; tỷ lệ nhãn không chắc chắn ở vòng II là 15%–30%, và vẫn có bất đồng trước khi thảo luận. Những số liệu này được tác giả dùng để cho thấy failure attribution thủ công rất tốn thời gian và khó.

# 3. Đánh giá các phương pháp và insight

## 3.1. Đánh đổi giữa ba phương pháp

Đánh đổi ổn định giữa ba phương pháp:

- **All-at-once:** cho kết quả agent-level cao nhất và ổn định nhất. Vì nhìn toàn bộ failure log trong một lượt, phương pháp có đủ bối cảnh để quy trách nhiệm giữa nhiều tác tử; đồng thời đây là phương pháp có chi phí input thấp nhất vì chỉ cần một lượt suy luận. Điểm yếu là định vị chính xác bước lỗi trong log dài.
- **Step-by-step:** cho kết quả step-level cao hơn, do LLM đánh giá từng hành động với ngữ cảnh tăng dần. Đổi lại, chi phí rất cao: ở mỗi bước mới, LLM lại nhận toàn bộ lịch sử từ đầu đến bước đó. Khi trace dài hoặc lỗi xuất hiện muộn, phần lịch sử bị đưa vào lặp lại nhiều lần.
- **Binary Search:** là phương án nằm giữa hai phương pháp trên. Nó không có toàn bộ bối cảnh ở mọi lượt như All-at-once, cũng không duyệt và nạp lại toàn bộ tiền tố qua từng bước như Step-by-step; thay vào đó nó thu hẹp dần log bằng các lần chia đôi. Trong thí nghiệm của tác giả, hiệu năng và chi phí của Binary Search thường ở giữa hai phương pháp còn lại.

## 3.2. Insight đáng chú ý

1. **Bối cảnh rộng và độ phân giải cao phục vụ hai mục tiêu khác nhau.** Toàn bộ log hữu ích khi cần xác định tác tử chịu trách nhiệm; còn việc xử lý tăng dần phù hợp hơn khi cần xác định chính xác bước lỗi. Tác giả liên hệ điểm yếu của All-at-once ở step-level với khó khăn tìm một chi tiết trong ngữ cảnh dài.

2. **Ground truth hỗ trợ phán đoán, nhưng không thay thế đánh giá trajectory.** Khi được cung cấp đáp án đúng, cả ba phương pháp đều tốt hơn. Tuy nhiên, tác giả lưu ý đáp án cuối không phải nhãn đúng–sai cho từng hành động; nó chỉ là tín hiệu để nhận ra hệ đang đi theo hướng không thể đạt đáp án đúng.

3. **Log dài là giới hạn chung.** Tác giả nhấn mạnh đây là khó khăn trọng yếu của failure attribution trong trace dài.

4. **Kết quả ở cấp thống kê hữu ích hơn kết quả từng instance.**

5. **Có thể kết hợp ưu điểm, nhưng phải trả chi phí.** Phương pháp lai dùng All-at-once để chọn tác tử rồi Step-by-step để tìm bước lỗi trong hành động của tác tử đó. Cách này cải thiện dự đoán bước lỗi, nhưng phải chạy tuần tự hai phương pháp nên có chi phí tính toán cao nhất.

6. **Model reasoning mạnh hơn không tự động giải quyết bài toán.**

# 4. Limitations và Future Works

## 4.1. Limitations

Bài báo không có section limitations riêng; các hạn chế dưới đây là những điều tác giả nêu trong kết quả và kết luận.

1. **Automated failure attribution ở cấp từng instance vẫn chưa dùng được trong thực tế.** Tác giả cho thấy việc xác định tác tử chịu trách nhiệm đã khó, còn định vị đúng bước lỗi quyết định khó hơn nhiều; một số thiết lập còn có kết quả thấp hơn random baseline. Ngay cả các reasoning model mạnh cũng không cải thiện một cách nhất quán.

2. **Trace dài là giới hạn trung tâm.** Khi ngữ cảnh dài hơn, hiệu quả của cả ba phương pháp giảm, đặc biệt ở step-level. Điều này giới hạn khả năng áp dụng các phương pháp hiện tại cho failure log dài.

3. **Nhãn ground truth có chi phí và độ bất định cao.** Như đã trình bày ở Mục 2.3, tác giả phải dùng nhiều vòng gán nhãn chuyên gia và vẫn quan sát nhãn không chắc chắn cùng bất đồng ban đầu. Đây là khó khăn của việc tạo nhãn lỗi quyết định đáng tin cậy.

4. **Kết quả mạnh hơn ở cấp thống kê so với cấp từng instance.** Các phương pháp có thể cho tín hiệu hữu ích khi tổng hợp nhiều log, nhưng điều đó chưa thay thế được chẩn đoán chính xác cho một lần chạy cụ thể.

5. **So sánh với OpenAI o1 có một thay đổi prompt.** Tác giả cho biết prompt gốc bị gắn cờ bởi chính sách sử dụng của OpenAI nên họ sửa nhỏ prompt cho o1, dù vẫn giữ ý định ban đầu. Đây là một lưu ý khi diễn giải so sánh reasoning model.

## 4.2. Future Works

Tác giả không đưa ra roadmap kỹ thuật cụ thể. Hướng tương lai mà họ nêu là cần tiếp tục nghiên cứu automated failure attribution, vì kết quả hiện tại cho thấy bài toán phức tạp và cấp thiết.

Who&When được giới thiệu như một nguồn lực nền tảng để thúc đẩy hướng nghiên cứu này: benchmark cung cấp failure log, nhãn tác tử/bước lỗi quyết định và giải thích bằng ngôn ngữ tự nhiên. Ngoài chẩn đoán từng instance, kết quả ở cấp thống kê cũng được tác giả xem là cơ sở có thể hành động hơn để ưu tiên tinh chỉnh các thành phần của hệ multi-agent.
