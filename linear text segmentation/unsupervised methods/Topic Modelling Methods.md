# Topic Modelling Methods

Nội dung này thuộc [[linear text segmentation/Linear Text Segmentation - Unsupervised Methods|Unsupervised Methods]]

## Cách hoạt động

Topic modelling methods là một nhánh unsupervised của linear text segmentation. Khác với [[Count-based Methods|Count-based Methods]] vốn so sánh trực tiếp sự thay đổi từ vựng giữa các vùng văn bản, hướng này giả định rằng văn bản được sinh ra từ các `topic` ẩn và cố suy ra các topic đó từ chính dữ liệu.

Ý tưởng cốt lõi là: 
- Mỗi `topic` được biểu diễn như một phân phối xác suất trên từ vựng ( `topic` không phải là một nhãn ngôn ngữ tự nhiên như `sports` hay `politics`, mà là một phân phối trong đó một số từ có xác suất cao hơn các từ khác). Ví dụ, nếu một topic cho xác suất cao với các từ như `team`, `goal`, `match`, `player`, người đọc có thể diễn giải nó như một topic về thể thao. 
- Tên topic thường là nhãn do con người đặt sau khi nhìn vào các từ có xác suất cao nhất, chứ không nhất thiết là thứ mô hình tự sinh ra sẵn dưới dạng ngôn ngữ tự nhiên.

Điểm quan trọng là mô hình không chỉ đếm từ ở mức bề mặt rồi so sánh trực tiếp hai vùng văn bản kề nhau. Thay vào đó, nó cố giải thích các mẫu đồng xuất hiện của từ bằng một lớp biến ẩn là `topic`. Từ rất nhiều lần các từ cùng xuất hiện trong corpus, mô hình học ra rằng có những nhóm từ thường đi cùng nhau và có thể được xem như do cùng một topic chi phối.

Trong survey, hướng này chủ yếu gắn với các mô hình như LDA. Một số phương pháp khác như TextTiling, TopicTiling.

Ưu điểm của topic modelling methods là chúng cho một cách nhìn có cấu trúc hơn so với count-based methods. Chúng không chỉ phát hiện nơi coherence giảm mà còn gắn mỗi đoạn với một topic ẩn, nên việc diễn giải nội dung của từng segment trở nên tự nhiên hơn. Survey cũng lưu ý rằng hướng này có lợi thế ở chỗ phân loại chủ đề ở mức đoạn có thể xuất hiện như một sản phẩm phụ sau khi phân đoạn, và nó cũng phù hợp hơn với các bài toán phân đoạn phân cấp.
