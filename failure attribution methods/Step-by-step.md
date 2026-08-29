# 1. Step-by-step

## 1.1. Cách hoạt động

Phương pháp này cũng được [TraceElephant](<TraceElephant dataset.md>) dùng làm prompting baseline; ở setting full observability, prefix log có thêm metadata, input context và tool log.

Step-by-step là một phương pháp phán đoán failure attribution trong [[Who&When dataset]]. LLM nhận truy vấn và failure log theo từng bước tăng dần. Ở bước \(i\), LLM nhận lịch sử từ bước đầu đến bước \(i\), rồi đánh giá liệu hành động mới nhất có chứa lỗi có thể cản trở quá trình giải bài hay không. Trong thiết lập có ground truth, prompt nhận thêm đáp án đúng của truy vấn.

Quy trình dừng ở lỗi đầu tiên mà LLM phát hiện:

1. Bắt đầu từ bước đầu của failure log.
2. Cung cấp cho LLM truy vấn và log từ bước đầu đến bước hiện tại.
3. Nếu LLM xác nhận hành động tại bước hiện tại có lỗi, trả về tác tử thực hiện hành động đó và số bước hiện tại.
4. Nếu không, chuyển sang bước kế tiếp; nếu đã hết log mà không phát hiện lỗi, trả về không tìm thấy lỗi.

Khác với All-at-once, phương pháp này không đưa toàn bộ log vào một lần. Nó kiểm tra tuần tự và có thể kết thúc ngay khi phát hiện lỗi; chi phí vì thế phụ thuộc vào vị trí bước lỗi quyết định và độ dài failure log.
