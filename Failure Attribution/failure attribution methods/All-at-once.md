# 1. All-at-once

## 1.1. Cách hoạt động

Phương pháp này cũng được [TraceElephant](<TraceElephant dataset.md>) dùng làm prompting baseline; ở setting full observability, full log có thêm metadata, input context và tool log.

All-at-once là một phương pháp phán đoán failure attribution trong [[Who&When dataset]]. Một LLM đóng vai trò đánh giá nhận **toàn bộ failure log trong một lần nhập**, cùng truy vấn mà hệ multi-agent đang cố giải. Trong thiết lập có ground truth, prompt nhận thêm đáp án đúng của truy vấn.

Từ ngữ cảnh đầy đủ này, LLM đưa ra một phán đoán duy nhất gồm:

- tên tác tử mà LLM cho là trực tiếp chịu trách nhiệm cho lời giải sai;
- số bước mà tác tử đó lần đầu mắc lỗi;
- lý do cho phán đoán.

Vì nhận toàn bộ log và chỉ suy luận một lần, phương pháp không lần lượt kiểm tra từng bước hay chia nhỏ trace. Theo phân tích chi phí của bài báo, All-at-once có chi phí input thấp nhất trong ba phương pháp vì chỉ cần một lượt suy luận.
