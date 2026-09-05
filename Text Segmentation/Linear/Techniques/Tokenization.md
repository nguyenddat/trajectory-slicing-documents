# Tokenization

Tokenization là bước tiền xử lý biến văn bản đầu vào thành các thuật ngữ và các đơn vị có kích thước đồng nhất để những bước sau có thể so sánh độ tương đồng từ vựng một cách ổn định.

## Quy trình

1. Xác định phần thân văn bản và bỏ qua header hoặc thông tin phụ.
2. Tách thân văn bản thành các token, rồi chuyển chúng về chữ thường.
3. Lọc stop words — các từ chức năng hoặc xuất hiện rất thường xuyên.
4. Đưa các token còn lại về dạng gốc bằng phân tích hình thái, để các biến thể danh từ và động từ được quy về cùng một term.
5. Chia tuần tự văn bản thành các **token-sequences** có độ dài cố định `w`.

## Token-sequences

Token-sequence là một nhóm gồm đúng `w` token liên tiếp. Chúng được xem là các **pseudo-sentences**, dùng thay cho câu cú pháp thật vì mọi đơn vị đều có cùng kích thước.

Việc chuẩn hóa kích thước này tránh so sánh thiếu công bằng giữa câu dài và câu ngắn, vốn có thể tạo ra các điểm tương đồng từ vựng khó chuẩn hóa. Token-sequences không chồng lấp: mỗi token chỉ thuộc một token-sequence.

Sự chồng lấp chỉ xuất hiện ở bước tính điểm sau đó, khi các block gồm nhiều token-sequences được trượt đi từng đơn vị để so sánh các vùng văn bản kề nhau.