## Evaluation Metrics for TextTiling

Theo bài TextTiling gốc, mỗi **paragraph gap** là một vị trí ứng viên. Khi đánh giá với người đọc, một gap là **true boundary** nếu ít nhất 3 trong 7 người đọc đánh dấu chuyển chủ đề tại đó; các gap còn lại là non-boundary. Với một hệ thống, gọi $TP$, $FP$, và $FN$ lần lượt là số boundary đúng được chọn, boundary thừa, và boundary đúng bị bỏ sót.

### Precision

$$
\operatorname{Precision}=\frac{TP}{TP+FP}.
$$

Precision là tỷ lệ boundary do thuật toán chọn mà trùng với true/majority boundary. Precision thấp nghĩa là nhiều **false positive** (insertion error): thuật toán chia đoạn quá mức hoặc đặt ranh giới sai.

### Recall

$$
\operatorname{Recall}=\frac{TP}{TP+FN}.
$$

Recall là tỷ lệ true boundary mà thuật toán tìm được. Recall thấp nghĩa là nhiều **false negative** (deletion error): thuật toán bỏ lỡ chuyển tiểu chủ đề.

TextTiling báo cáo precision và recall ở hai mức cutoff: cutoff liberal thường tăng recall nhưng giảm precision; cutoff conservative thường tăng precision nhưng giảm recall. Bài gốc không dùng F1 làm metric chính.

### Kappa agreement ($\kappa$)

$$
\kappa=\frac{P(A)-P(E)}{1-P(E)}.
$$

Trong đó $P(A)$ là tỷ lệ đồng ý quan sát được giữa hai bộ nhãn boundary/non-boundary, còn $P(E)$ là tỷ lệ đồng ý kỳ vọng do ngẫu nhiên. Kappa được dùng để:

- đánh giá mức đồng thuận giữa người đọc và quyết định nhóm;
- báo cáo mức đồng thuận giữa kết quả TextTiling và phán quyết của người đọc.

Metric này quan trọng vì boundary chỉ chiếm một phần các paragraph gap; accuracy thô có thể cao đơn giản do luôn dự đoán non-boundary. Trong thí nghiệm của bài, các người đọc được so với quyết định nhóm và giá trị kappa trung bình là $0.647$.

### Lưu ý về cách khớp boundary

Đánh giá phân đoạn tiểu chủ đề chính dùng paragraph gap được đa số người đọc chọn, nên precision/recall ở đây là phép khớp boundary theo đúng vị trí gap. Thí nghiệm phụ về phân biệt các bài báo nối tiếp lại chấp nhận một boundary nằm trong phạm vi ba câu so với vị trí đúng; đây là quy tắc khớp riêng cho thiết lập đó, không thay đổi định nghĩa precision hoặc recall.
