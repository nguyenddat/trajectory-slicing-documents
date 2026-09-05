# Disentangled Contrastive Learning (DCL)

## Cách hoạt động

DCL là phương pháp contrastive do [AEGIS](<Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>) đề xuất cho failure attribution trên [Aegis dataset](<Aegis dataset.md>). Nó xem failed trajectory là một bag of turns và học tách tín hiệu về agent khỏi tín hiệu về error mode.

**Đầu vào.** Ở suy luận, DCL nhận toàn bộ failed trajectory, tách thành các turn. Khi train, nó còn nhận gold agent/mode labels, trajectory thành công tương ứng của cùng task, và hai prototype bank khởi tạo từ mô tả văn bản của agent và error mode.

**Quy trình suy luận và học.**

1. Encoder mã hóa từng turn; MIL attention chọn Top-\(K\) evidence turn có tín hiệu lỗi nổi bật từ toàn trajectory.
2. Các evidence turn được đối chiếu với prototype bank agent và error mode để tạo phân phối độc lập \(p_A\) (*who*) và \(p_E\) (*why*).
3. Pair head kết hợp hai phân phối thành xác suất cặp agent–mode \(p_P\).
4. Trong huấn luyện, mỗi evidence turn được kéo gần turn “clean” tương ứng của trajectory thành công và prototype đúng; nó bị đẩy xa prototype sai cùng hard negative từ bag khác. Loss đồng thời gồm BCE đa nhãn ở mức agent/mode/pair, supervised contrastive loss và hierarchical consistency loss.
5. Consistency loss phạt mọi cặp có xác suất lớn hơn xác suất của chính agent hoặc mode thành phần.

**Điều kiện dừng và đầu ra.** DCL xử lý toàn bag một lần, chọn một số evidence turn cố định rồi trả về các dự đoán đa nhãn agent, error mode và cặp của chúng; nó không duyệt tuần tự log để tìm một bước dừng.

**Đặc điểm vận hành cần thiết.** Turn thành công ghép cặp chỉ cần trong huấn luyện để tạo positive anchor; suy luận không cần trajectory thành công. Bài dùng encoder all-MiniLM-L6-v2 nhẹ hơn nhiều LLM SFT và đặt DCL như một cách học biểu diễn cho tín hiệu lỗi thưa, compositional.
