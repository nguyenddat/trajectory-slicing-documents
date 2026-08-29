# Hướng dẫn cho agent

Đây là project dùng để đọc và ghi chú tài liệu nghiên cứu.

## Tài liệu nguồn

- Các file nghiên cứu gốc được lưu trong thư mục `raw/`.
- Không chỉnh sửa, di chuyển, hoặc xoá tài liệu trong `raw/` trừ khi người dùng yêu cầu rõ ràng.

## Khi đọc và ghi chú nghiên cứu

Bước tiên quyết là xác định rõ vấn đề và bài toán nghiên cứu mà tác giả đặt ra để giải quyết. Cần thực sự tập trung vào vấn đề này: phân biệt điều tác giả muốn giải quyết với phương pháp hoặc đóng góp mà họ dùng để giải quyết nó. Không bắt đầu bằng việc tóm tắt phương pháp hay kết quả khi chưa làm rõ điểm này.

Mỗi ghi chú nghiên cứu phải luôn mở đầu bằng đúng section:

```markdown
# 1. Vấn đề đặt ra
```

### Quy ước đánh số header

Dùng đánh số phân cấp cho các header trong note:

- Cấp 1: `# 1. Tiêu đề`, `# 2. Tiêu đề`.
- Cấp 2: `## 1.1. Tiêu đề`, `## 1.2. Tiêu đề`, `## 2.1. Tiêu đề`.
- Cấp 3: `### 1.1.1. Tiêu đề`.

Số thứ tự phải phản ánh đúng quan hệ cha–con, bắt đầu lại theo section cha và không bỏ cấp.

Section này cần trả lời dựa trên tài liệu nguồn:

- Bối cảnh hoặc khoảng trống nào thúc đẩy nghiên cứu.
- Vấn đề cụ thể tác giả muốn giải quyết.
- Bài toán nghiên cứu được định nghĩa như thế nào (đầu vào, đầu ra, mục tiêu và các ràng buộc quan trọng nếu có).
- Những phê bình hoặc giới hạn mà tác giả nêu đối với các nghiên cứu trước; làm rõ vì sao chúng chưa giải quyết đầy đủ vấn đề và khoảng trống mà nghiên cứu hiện tại nhắm tới.

Khi tác giả nhận xét, so sánh hoặc chỉ ra ưu/nhược điểm của một nghiên cứu đã có note riêng, hãy liên kết trực tiếp tên nghiên cứu được nhắc tới đến note đó ngay tại nhận định. Ví dụ, khi note TRAIL nêu giới hạn mà tác giả quy cho MAST, viết liên kết tới note MAST thay vì chỉ ghi tên MAST hoặc chỉ liên kết đến tài liệu nguồn. Điều này áp dụng cho cả phê bình tích cực lẫn tiêu cực; không tạo liên kết để ngụ ý một nhận định mà nguồn không nêu.

Khi phân tích vấn đề đặt ra, cần đối chiếu với bối cảnh nghiên cứu chung. Nếu nghiên cứu hiện tại đề cập đến cùng bối cảnh, hãy thêm ref của nghiên cứu vào phần bối cảnh chung đó và không lặp lại bối cảnh này trong file note riêng. Nếu nghiên cứu có điểm mới so với bối cảnh chung, chỉ viết riêng điểm mới đó trong file note; không đưa điểm mới vào file bối cảnh nghiên cứu chung.

Chỉ sau section này mới phân tích các nội dung khác như phương pháp, dữ liệu, thí nghiệm, kết quả, giới hạn, hoặc nhận xét.

## Nghiên cứu phát triển và dataset

Việc xây dựng dataset thường không phải là nội dung chính của nghiên cứu phát triển, nên không cần đi quá sâu vào các chi tiết triển khai. Tuy vậy, ghi chú vẫn phải nắm rõ:

- **Đặc trưng cơ bản của dataset:** đơn vị dữ liệu, quy mô, nguồn/split và các tổ hợp chạy hay cấu hình hệ thống tạo dữ liệu. Chỉ nêu số liệu hay tổ hợp cần để người đọc biết dataset bao phủ cái gì; nếu tài liệu nguồn mâu thuẫn giữa các số tổng hợp, ghi rõ mâu thuẫn thay vì tự chọn một số.
- Dataset được thiết kế như thế nào ở mức cần thiết để hiểu nghiên cứu.
- Mục đích và đánh đổi đằng sau thiết kế đó.

Ví dụ, nếu mục tiêu là mở rộng dataset ở quy mô lớn thì cần nhận diện vai trò của LLM hoặc tự động hoá; nếu mục tiêu là chiều sâu và chất lượng dữ liệu thì cần nhận diện phần đóng góp thủ công, chuyên môn, hoặc quy trình kiểm định. Tập trung giải thích **vì sao** họ chọn cách thiết kế dữ liệu này, thay vì liệt kê tỉ mỉ từng bước xây dựng.

Khi trình bày dataset trong một file note, dùng thứ tự sau (ở mức chi tiết cần thiết để hiểu nghiên cứu):

1. **Đặc trưng cơ bản:** nêu đơn vị dữ liệu, quy mô, nguồn/split và các tổ hợp chạy hoặc cấu hình tạo dữ liệu.
2. **Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu:** nêu benchmark/tập dữ liệu nguồn, framework hoặc hệ thống được chạy, và cách các mẫu hay log được tạo/thu thập.
3. **Các bước sinh và gán nhãn:** trình bày theo các bước được đánh số, chỉ giữ những thao tác quyết định bản chất và chất lượng của dữ liệu.
4. **Mục đích và đánh đổi của thiết kế:** giải thích mục tiêu phía sau lựa chọn nguồn dữ liệu, tự động hoá, lọc mẫu và quy trình kiểm định.

Đặc biệt, phải nhận diện và viết rõ các ý kiến phản biện, giới hạn hoặc khó khăn mà tác giả nêu. Phân biệt chúng với nhận định của agent: chỉ quy các phê bình cho tác giả khi tài liệu nguồn thực sự nêu hoặc dùng bằng chứng để lập luận cho chúng; nếu những phê bình này giải thích một lựa chọn thiết kế dữ liệu, hãy nối rõ hai ý đó trong phần mục đích và đánh đổi.

## Phương pháp failure attribution

Mỗi phương pháp failure attribution được ghi trong file riêng. File mở đầu bằng tên phương pháp, sau đó có section `## Cách hoạt động` và liên kết đến nghiên cứu/dataset đã dùng phương pháp đó.

Phần cách hoạt động phải trình bày ngắn gọn theo thứ tự sau:

1. **Đầu vào:** truy vấn, failure log (toàn bộ, tiền tố hay đoạn log tùy phương pháp) và các tín hiệu tùy chọn như ground truth.
2. **Quy trình suy luận:** mô tả các bước được đánh số theo đúng thứ tự thực thi, gồm cách phương pháp mở rộng, thu hẹp hoặc duyệt failure log.
3. **Điều kiện dừng và đầu ra:** nêu khi nào phương pháp kết thúc và nó trả về tác tử, bước lỗi hoặc kết quả khác nào.
4. **Đặc điểm vận hành cần thiết:** chỉ nêu chi phí, số lượt suy luận hoặc khác biệt với các phương pháp khác khi nghiên cứu nguồn nói rõ và điều đó giúp hiểu cách phương pháp hoạt động.

Ưu tiên hành vi thực thi trong code, pseudocode hoặc prompt được dùng trong nghiên cứu hơn mô tả khái quát. Không suy ra cách phương pháp chia, duyệt hay cập nhật failure log ngoài những gì nguồn thể hiện. Không đưa kết quả thí nghiệm vào file phương pháp trừ khi người dùng yêu cầu riêng.

## Cấu trúc note của một nghiên cứu

Note của một nghiên cứu trình bày theo thứ tự sau khi các nội dung này có mặt và phù hợp với loại nghiên cứu:

1. **Vấn đề đặt ra:** luôn là section đầu tiên và tuân theo các yêu cầu ở trên. Liên kết sang bối cảnh chung hoặc file khái niệm riêng thay vì lặp lại phần đã được trình bày ở đó.
2. **Dataset hoặc thiết kế dữ liệu:** với nghiên cứu phát triển/dataset, áp dụng đúng cấu trúc đặc trưng cơ bản → nguồn dữ liệu → các bước sinh/gán nhãn → mục đích và đánh đổi. Với nghiên cứu không xây dựng dataset, chỉ mô tả dữ liệu ở mức cần thiết để hiểu thí nghiệm.
3. **Đánh giá phương pháp và insight:** nêu thiết lập đánh giá ở mức cần thiết, sau đó tổng hợp các xu hướng, đánh đổi và insight mà tác giả thực sự rút ra. Khi kết quả sẽ được chạy lại trong project, ưu tiên các kết luận định tính có tính ổn định (ví dụ: ưu/nhược điểm, chi phí, tác động của điều kiện thử nghiệm) thay vì chép các con số cụ thể từ bài báo; chỉ ghi con số khi người dùng yêu cầu hoặc nó thiết yếu cho lập luận.
4. **Limitations và Future Works:** tách rõ hạn chế đã được tác giả nêu trong bài với hướng tương lai mà tác giả thực sự đề xuất. Nếu bài báo không có section riêng hoặc không nêu roadmap kỹ thuật, phải nói rõ điều đó; không biến suy luận của agent thành future work của tác giả.

Các phương pháp hoặc khái niệm dùng chung được tách sang file riêng và liên kết từ note nghiên cứu. Mỗi section cần ngắn gọn, dùng heading phân cấp, đoạn văn ngắn hoặc danh sách khi có nhiều ý độc lập. Chỉ viết những nhận định có thể đối chiếu với tài liệu nguồn; khi nêu phê bình, giới hạn hoặc insight, phải cho thấy đó là kết luận của tác giả, không phải suy diễn của agent.
