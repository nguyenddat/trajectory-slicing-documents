Nội dung này thuộc [[Text Segmentation/Recent Trends in Linear Text Segmentation A Survey|Recent Trends in Linear Text Segmentation A Survey]]
# Unsupervised Methods

Unsupervised methods là các cách tiếp cận không dựa vào dữ liệu gán nhãn ranh giới chủ đề để huấn luyện trực tiếp. Thay vào đó, chúng khai thác các tín hiệu như độ giống nhau giữa các đoạn văn bản liền kề, cấu trúc chủ đề tiềm ẩn, hoặc biểu diễn ngữ nghĩa từ các mô hình ngôn ngữ để suy ra nơi chủ đề thay đổi.

## Count-based Methods

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

## Topic Modelling Methods

Topic modelling methods là một nhánh unsupervised của linear text segmentation. Khác với [[Unsupervised Methods#Count-based Methods|Count-based Methods]] vốn so sánh trực tiếp sự thay đổi từ vựng giữa các vùng văn bản, hướng này giả định rằng văn bản được sinh ra từ các `topic` ẩn và cố suy ra các topic đó từ chính dữ liệu.

Ý tưởng cốt lõi là: 
- Mỗi `topic` được biểu diễn như một phân phối xác suất trên từ vựng ( `topic` không phải là một nhãn ngôn ngữ tự nhiên như `sports` hay `politics`, mà là một phân phối trong đó một số từ có xác suất cao hơn các từ khác). Ví dụ, nếu một topic cho xác suất cao với các từ như `team`, `goal`, `match`, `player`, người đọc có thể diễn giải nó như một topic về thể thao. 
- Tên topic thường là nhãn do con người đặt sau khi nhìn vào các từ có xác suất cao nhất, chứ không nhất thiết là thứ mô hình tự sinh ra sẵn dưới dạng ngôn ngữ tự nhiên.

Điểm quan trọng là mô hình không chỉ đếm từ ở mức bề mặt rồi so sánh trực tiếp hai vùng văn bản kề nhau. Thay vào đó, nó cố giải thích các mẫu đồng xuất hiện của từ bằng một lớp biến ẩn là `topic`. Từ rất nhiều lần các từ cùng xuất hiện trong corpus, mô hình học ra rằng có những nhóm từ thường đi cùng nhau và có thể được xem như do cùng một topic chi phối.

Trong survey, hướng này chủ yếu gắn với các mô hình như LDA. Một số phương pháp khác như TextTiling, TopicTiling.

Ưu điểm của topic modelling methods là chúng cho một cách nhìn có cấu trúc hơn so với count-based methods. Chúng không chỉ phát hiện nơi coherence giảm mà còn gắn mỗi đoạn với một topic ẩn, nên việc diễn giải nội dung của từng segment trở nên tự nhiên hơn. Survey cũng lưu ý rằng hướng này có lợi thế ở chỗ phân loại chủ đề ở mức đoạn có thể xuất hiện như một sản phẩm phụ sau khi phân đoạn, và nó cũng phù hợp hơn với các bài toán phân đoạn phân cấp.

## Embeddings-based Methods

Embeddings-based methods là nhánh unsupervised chuyển trọng tâm từ thống kê đếm từ sang biểu diễn ngữ nghĩa liên tục của từ hoặc câu. 

Ý tưởng chính là: thay vì xem hai vùng văn bản giống nhau chỉ khi chúng lặp lại cùng một từ, ta biểu diễn chúng bằng `embedding` rồi đo mức độ gần nhau trong không gian vector.

Ở giai đoạn đầu, hướng này chủ yếu dùng `word embeddings`. Các phương pháp như vậy đánh giá coherence giữa các câu liên tiếp bằng quan hệ giữa embedding của các từ cấu thành chúng. Survey nêu `GraphSeg` như một ví dụ tiêu biểu.

Về sau, khi neural language models phát triển hơn, trọng tâm chuyển dần từ `word-based methods` sang `sentence-based methods`. Thay vì làm việc trực tiếp với từng từ riêng lẻ, hệ thống tạo `sentence embeddings` hoặc biểu diễn cho `speaker turn` từ các mô hình transformer-based language models như `BERT`, rồi dùng các biểu diễn này trong các quy trình segmentation quen thuộc.

Điểm quan trọng ở đây là pipeline tổng thể không nhất thiết phải hoàn toàn mới. Một số hệ vẫn giữ tinh thần của các phương pháp cũ như TextTiling, nhưng thay biểu diễn bag-of-words bằng dense semantic representations. Nghĩa là phần quyết định ranh giới vẫn dựa trên sự thay đổi coherence giữa các đơn vị liên tiếp, nhưng coherence lúc này được đo bằng độ gần ngữ nghĩa thay vì chỉ bằng trùng lặp từ vựng.

Theo survey, đây là một bước chuyển quan trọng của lĩnh vực: từ các phương pháp dựa trên thống kê từ vựng sang các phương pháp dựa trên biểu diễn ngữ nghĩa mạnh hơn. Sự dịch chuyển từ `word embeddings` sang `sentence embeddings` dựa trên transformer cũng mở đường cho các hệ supervised và LLM-based về sau.

## LLM-based Methods

LLM-based methods là nhánh mới nhất trong các phương pháp unsupervised cho linear text segmentation mà survey đề cập. Thay vì dựa chủ yếu vào thống kê từ, topic ẩn, hay embedding cố định rồi gắn vào một thuật toán segmentation riêng, hướng này dùng trực tiếp Large Language Models như `ChatGPT` để xử lý bài toán thông qua prompting.

Ý tưởng chính là xem text segmentation như một bài toán `Natural Language Generation` hoặc `instruction following`. Thay vì chỉ đo độ giống nhau giữa hai vùng văn bản liên tiếp, hệ thống đưa văn bản vào LLM cùng với prompt yêu cầu mô hình xác định nơi chủ đề thay đổi hoặc sinh ra cấu trúc phân đoạn phù hợp.

Theo survey, các công trình đầu tiên trong hướng này dùng multi-billion parameter LLMs trong thiết lập `zero-shot`, tức là không huấn luyện thêm trực tiếp trên dữ liệu segmentation mà dựa vào prompt engineering để mô hình thực hiện nhiệm vụ. Trong cách nhìn của survey, các hệ như vậy vẫn được xếp vào unsupervised methods vì chúng không học có giám sát riêng cho bài toán từ dữ liệu gán nhãn segmentation.

Điểm mạnh nổi bật của hướng này là khả năng tận dụng hiểu biết ngữ nghĩa và diễn ngôn ở mức rộng hơn so với các phương pháp unsupervised trước đó. Survey ghi nhận rằng khi prompt được tối ưu cẩn thận, LLM-based methods có thể vượt qua các phương pháp unsupervised khác. Điều này cho thấy LLM là một hướng đầy hứa hẹn, đặc biệt khi muốn vượt qua những giới hạn của các cách tiếp cận dựa mạnh vào trùng lặp từ vựng hoặc các biểu diễn cục bộ.

Tuy vậy, survey cũng xem đây mới là một hướng ở giai đoạn đầu. Hiệu quả của phương pháp phụ thuộc nhiều vào prompt, vào hành vi của mô hình nền, và vào cách chuyển đầu ra sinh ngôn ngữ tự nhiên của LLM thành ranh giới phân đoạn cụ thể. Vì vậy, dù rất hứa hẹn, LLM-based methods vẫn được tác giả xem như một hướng cần được khám phá thêm, nhất là trong mối liên hệ với các hạn chế của tài nguyên và đánh giá trong lĩnh vực.
