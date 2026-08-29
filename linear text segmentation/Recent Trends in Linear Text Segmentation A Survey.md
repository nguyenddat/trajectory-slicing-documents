
# 1. Vấn đề đặt ra

Linear text segmentation, hay topic segmentation, là bài toán xác định ranh giới nơi chủ đề thay đổi trong một văn bản. 

=> Tính chất `linear`: các chủ đề được xem như xuất hiện nối tiếp nhau theo trật tự trong tài liệu; thay vì mô hình hóa cấu trúc phân cấp hay các tiểu chủ đề.

Bài toán này khác với topic classification. Mục tiêu không phải là gán nhãn chủ đề cho văn bản hay cho từng đoạn, mà là xác định chính xác vị trí nơi một chủ đề kết thúc và chủ đề khác bắt đầu.

# 2. Các hướng tiếp cận

## 2.1. Basic Units

Theo tác giả, các hệ thống thường làm việc ở mức 
- Từ
- Câu 
- Pseudo-sentence, 
- Đoạn văn
-  `speaker turn` (một số bối cảnh hội thoại nhiều người nói)

Trong các nghiên cứu sớm, đoạn văn thường được dùng vì chúng thường gắn với sự tổ chức chủ đề trong văn bản viết. Tuy nhiên, khi bài toán được mở rộng sang các miền như spoken language, multimedia, hoặc các dữ liệu không có cấu trúc đoạn rõ ràng, vai trò này giảm dần và được thay bằng từ, câu, hoặc pseudo-sentence. Pseudo-sentence từng được dùng để tránh lỗi tách câu, nhất là trong giai đoạn công cụ sentence tokenization còn chưa ổn định.

Với các dữ liệu hội thoại nhiều người nói như meeting transcript, `speaker turn` thường là lựa chọn tự nhiên hơn vì mỗi lượt nói thường đủ hoàn chỉnh để mang một ý giao tiếp tương đối mạch lạc. 