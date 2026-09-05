## Block comparison

**Block comparison** là một cách đo độ tương đồng từ vựng giữa hai khối văn bản kề nhau. Trong TextTiling, giá trị này được gọi là **lexical score** và được tính tại mỗi khoảng trống giữa hai token-sequences.

Tại khoảng trống `i`, thuật toán tạo:
- block trái gồm `k` token-sequences ngay trước `i`;
- block phải gồm `k` token-sequences ngay sau `i`.

Mỗi block được biểu diễn bằng một vector tần suất term. Điểm tương đồng là tích vô hướng đã chuẩn hóa của hai vector, tương đương cosine similarity:

$$
\operatorname{sim}(b_1,b_2)=
\frac{\sum_t f(t,b_1)f(t,b_2)}
{\sqrt{\sum_t f(t,b_1)^2}\sqrt{\sum_t f(t,b_2)^2}}
$$

Trong đó, $f(t,b)$ là số lần term $t$ xuất hiện trong block $b$. Điểm nằm trong khoảng từ 0 đến 1.
- Điểm cao: hai block có nhiều term chung, nên khả năng cao vẫn thuộc cùng một tiểu chủ đề.
- Điểm thấp: vốn từ giữa hai block thay đổi rõ rệt, nên khoảng trống ở giữa là một ứng viên ranh giới tiểu chủ đề.

Hai block được trượt dọc văn bản từng token-sequence, vì vậy các block ở những lần tính liên tiếp có chồng lấp. Chuỗi lexical score sau đó được làm trơn; các thung lũng sâu trong chuỗi này được dùng để phát hiện ranh giới.

## Vocabulary introduction không phải là similarity
Trong TextTiling, **vocabulary introduction** là một phương pháp gán **lexical score** thay thế cho block comparison, chứ không phải một cách đo độ tương đồng. Nó không so sánh block trái với block phải.

Tại gap $i$, phương pháp này xét hai token-sequences kề gap, tạo thành một interval dài $2w$ token. Score là tỷ lệ term xuất hiện **lần đầu tiên trong toàn văn bản** ở interval đó:

$$
\operatorname{score}(i)=
\frac{\operatorname{NumNewTerms}(b_1)+\operatorname{NumNewTerms}(b_2)}{2w},
$$

trong đó $b_1$ và $b_2$ lần lượt là token-sequence bên trái và bên phải; $\operatorname{NumNewTerms}(b)$ đếm các term trong $b$ chưa từng xuất hiện ở phần văn bản trước đó. Term lặp lại trong interval hoặc đã xuất hiện trước đó không đóng góp thêm vào score.

Hai phương pháp có cùng đầu ra là một chuỗi lexical score tại các gap và cùng được đưa vào bước xác định ranh giới, nhưng ý nghĩa điểm khác nhau:
- **Block comparison:** điểm cao nghĩa là hai phía có nhiều term chung.
- **Vocabulary introduction:** điểm cao nghĩa là quanh gap có nhiều term được giới thiệu lần đầu; nó không cho biết hai phía giống nhau đến đâu.
