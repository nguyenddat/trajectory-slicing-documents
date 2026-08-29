# LLM-based Methods

Nội dung này thuộc [[linear text segmentation/Linear Text Segmentation - Unsupervised Methods|Unsupervised Methods]]

## Cách hoạt động

LLM-based methods là nhánh mới nhất trong các phương pháp unsupervised cho linear text segmentation mà survey đề cập. Thay vì dựa chủ yếu vào thống kê từ, topic ẩn, hay embedding cố định rồi gắn vào một thuật toán segmentation riêng, hướng này dùng trực tiếp Large Language Models như `ChatGPT` để xử lý bài toán thông qua prompting.

Ý tưởng chính là xem text segmentation như một bài toán `Natural Language Generation` hoặc `instruction following`. Thay vì chỉ đo độ giống nhau giữa hai vùng văn bản liên tiếp, hệ thống đưa văn bản vào LLM cùng với prompt yêu cầu mô hình xác định nơi chủ đề thay đổi hoặc sinh ra cấu trúc phân đoạn phù hợp.

Theo survey, các công trình đầu tiên trong hướng này dùng multi-billion parameter LLMs trong thiết lập `zero-shot`, tức là không huấn luyện thêm trực tiếp trên dữ liệu segmentation mà dựa vào prompt engineering để mô hình thực hiện nhiệm vụ. Trong cách nhìn của survey, các hệ như vậy vẫn được xếp vào unsupervised methods vì chúng không học có giám sát riêng cho bài toán từ dữ liệu gán nhãn segmentation.

Điểm mạnh nổi bật của hướng này là khả năng tận dụng hiểu biết ngữ nghĩa và diễn ngôn ở mức rộng hơn so với các phương pháp unsupervised trước đó. Survey ghi nhận rằng khi prompt được tối ưu cẩn thận, LLM-based methods có thể vượt qua các phương pháp unsupervised khác. Điều này cho thấy LLM là một hướng đầy hứa hẹn, đặc biệt khi muốn vượt qua những giới hạn của các cách tiếp cận dựa mạnh vào trùng lặp từ vựng hoặc các biểu diễn cục bộ.

Tuy vậy, survey cũng xem đây mới là một hướng ở giai đoạn đầu. Hiệu quả của phương pháp phụ thuộc nhiều vào prompt, vào hành vi của mô hình nền, và vào cách chuyển đầu ra sinh ngôn ngữ tự nhiên của LLM thành ranh giới phân đoạn cụ thể. Vì vậy, dù rất hứa hẹn, LLM-based methods vẫn được tác giả xem như một hướng cần được khám phá thêm, nhất là trong mối liên hệ với các hạn chế của tài nguyên và đánh giá trong lĩnh vực.
