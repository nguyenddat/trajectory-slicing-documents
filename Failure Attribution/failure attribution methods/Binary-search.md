# 1. Binary Search

## 1.1. Cách hoạt động

Phương pháp này cũng được [TraceElephant](<TraceElephant dataset.md>) dùng làm prompting baseline; ở setting full observability, failure log có thêm metadata, input context và tool log.

Binary Search là một phương pháp phán đoán failure attribution trong [[Who&When dataset]]. Phương pháp tìm bước lỗi quyết định bằng cách thu hẹp dần failure log cần xét. LLM nhận truy vấn và failure log hiện đang được xét; trong thiết lập có ground truth, prompt nhận thêm đáp án đúng của truy vấn.

Quy trình bắt đầu với toàn bộ failure log và duy trì hai mốc đầu–cuối của phạm vi tìm kiếm:

1. Ở lượt đầu, cung cấp truy vấn và **toàn bộ failure log** cho LLM, rồi hỏi lỗi nằm ở nửa trên hay nửa dưới của log.
2. Giữ lại nửa mà LLM chọn và đưa toàn bộ nửa đó vào LLM ở lượt tiếp theo.
3. Tiếp tục chia đôi đoạn log đang được giữ, hỏi LLM chọn nửa trên hay nửa dưới, rồi chỉ giữ nửa được chọn.
4. Lặp lại đến khi đoạn log chỉ còn một bước. Bước đó là bước lỗi được dự đoán; tác tử thực hiện hành động ở bước này được trả về là tác tử chịu trách nhiệm.

Do mỗi lượt loại bỏ khoảng một nửa không gian tìm kiếm, Binary Search cần \(\lceil \log_2 n \rceil\) lượt khi failure log có \(n\) bước. Khác với All-at-once, full failure log chỉ được dùng để đưa ra quyết định chia đầu tiên; các lượt sau chỉ xử lý nửa log mà LLM đã chọn. Khác với Step-by-step, phương pháp không kiểm tra lần lượt từng bước.
