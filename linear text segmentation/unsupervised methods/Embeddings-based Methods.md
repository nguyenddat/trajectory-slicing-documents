# Embeddings-based Methods

Nội dung này thuộc [[linear text segmentation/Linear Text Segmentation - Unsupervised Methods|Unsupervised Methods]]

## Cách hoạt động

Embeddings-based methods là nhánh unsupervised chuyển trọng tâm từ thống kê đếm từ sang biểu diễn ngữ nghĩa liên tục của từ hoặc câu. 

Ý tưởng chính là: thay vì xem hai vùng văn bản giống nhau chỉ khi chúng lặp lại cùng một từ, ta biểu diễn chúng bằng `embedding` rồi đo mức độ gần nhau trong không gian vector.

Ở giai đoạn đầu, hướng này chủ yếu dùng `word embeddings`. Các phương pháp như vậy đánh giá coherence giữa các câu liên tiếp bằng quan hệ giữa embedding của các từ cấu thành chúng. Survey nêu `GraphSeg` như một ví dụ tiêu biểu.

Về sau, khi neural language models phát triển hơn, trọng tâm chuyển dần từ `word-based methods` sang `sentence-based methods`. Thay vì làm việc trực tiếp với từng từ riêng lẻ, hệ thống tạo `sentence embeddings` hoặc biểu diễn cho `speaker turn` từ các mô hình transformer-based language models như `BERT`, rồi dùng các biểu diễn này trong các quy trình segmentation quen thuộc.

Điểm quan trọng ở đây là pipeline tổng thể không nhất thiết phải hoàn toàn mới. Một số hệ vẫn giữ tinh thần của các phương pháp cũ như TextTiling, nhưng thay biểu diễn bag-of-words bằng dense semantic representations. Nghĩa là phần quyết định ranh giới vẫn dựa trên sự thay đổi coherence giữa các đơn vị liên tiếp, nhưng coherence lúc này được đo bằng độ gần ngữ nghĩa thay vì chỉ bằng trùng lặp từ vựng.

Theo survey, đây là một bước chuyển quan trọng của lĩnh vực: từ các phương pháp dựa trên thống kê từ vựng sang các phương pháp dựa trên biểu diễn ngữ nghĩa mạnh hơn. Sự dịch chuyển từ `word embeddings` sang `sentence embeddings` dựa trên transformer cũng mở đường cho các hệ supervised và LLM-based về sau.
