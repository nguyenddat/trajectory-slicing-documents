# Count-based Methods

Phương pháp này thuộc nhóm [[Linear Text Segmentation - Unsupervised Methods]].

## Cách hoạt động

Count-based methods là nhóm phương pháp phân đoạn chủ đề dựa chủ yếu vào thống kê xuất hiện của từ. 

Ý tưởng cốt lõi của chúng là: nếu hai đoạn văn bản liền kề nói về cùng một chủ đề, phân bố từ vựng của chúng sẽ khá giống nhau; nếu chủ đề đổi, sự giống nhau đó sẽ giảm xuống rõ rệt.

Count-based methods không cố “hiểu nghĩa” sâu của văn bản, mà dựa vào sự thay đổi trong mẫu phân bố từ.  Phương pháp kinh điển nhất là TextTiling. Nó trượt hai cửa sổ qua chuỗi câu:
  1. Lấy một khối câu bên trái và một khối câu bên phải.
  2. Biểu diễn mỗi khối bằng bag-of-words.
  3. Tính độ tương tự, thường là cosine similarity.
  4. Di chuyển cửa sổ dọc văn bản và ghi lại đường cong độ tương tự.
  5. Những “đáy” sâu trong đường cong là ứng viên ranh giới chủ đề.

Trực giác ở đây khá mạnh: khi chủ đề đang ổn định, các từ ở hai bên cửa sổ giống nhau nhiều; khi văn bản chuyển sang chủ đề khác, từ vựng thay đổi nên độ tương tự tụt xuống. Ví dụ đoạn trước nói về football, team, goal, match, còn đoạn sau nói về election, vote, party, candidate, thì tín hiệu đổi chủ đề sẽ khá rõ.

Sau TextTiling, có các cải tiến như dùng TF-IDF thay vì đếm thô. Một biến thể nổi tiếng khác là C99. 

Ngoài nhóm “so sánh hai cửa sổ”, còn có nhóm dùng dynamic programming, HMM, hoặc language model đếm từ. Ở đây ý tưởng là: mỗi chủ đề có một phân bố từ riêng, và bài toán trở thành tìm cách chia văn bản thành các đoạn sao cho xác suất sinh ra văn bản dưới các chủ đề đó là hợp lý nhất. Khác với TextTiling vốn khá heuristic, nhóm này có khung tối ưu hóa rõ hơn. Tuy nhiên bản chất vẫn là dựa vào thống kê từ và xác suất từ, nên vẫn được xếp vào count-based.

Điểm mạnh của count-based methods:
  - Đơn giản, dễ hiểu, dễ cài đặt.
  - Không cần dữ liệu gán nhãn.
  - Hợp với thời kỳ dữ liệu nhỏ hoặc tài nguyên tính toán hạn chế.
  - Giải thích được vì sao hệ thống đặt ranh giới ở đâu.

Điểm yếu của chúng cũng khá rõ:
  - Phụ thuộc mạnh vào trùng lặp từ bề mặt.
  - Khó nhận ra hai đoạn cùng chủ đề nhưng dùng từ khác nhau.
  - Nhạy với nhiễu từ vựng, độ dài đoạn, và lựa chọn cửa sổ.
  - Không nắm bắt ngữ nghĩa sâu như embedding hay transformer.

Nói ngắn gọn, count-based methods xem phân đoạn chủ đề như bài toán phát hiện nơi mà thống kê từ vựng thay đổi đáng kể. Chúng là nền tảng lịch sử rất quan trọng vì đặt ra trực giác coherence cục bộ, nhưng về sau bị vượt qua bởi các phương pháp dùng topic modeling, embeddings, và transformer do các phương pháp mới biểu diễn ngữ nghĩa tốt hơn.