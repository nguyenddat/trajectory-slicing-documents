# Bag-of-Words

**Bag-of-Words (BoW)** biểu diễn một đơn vị văn bản bằng số lần các term trong một từ vựng xác định xuất hiện trong đơn vị đó. Mỗi văn bản vì vậy được chuyển thành một vector có cùng số chiều với từ vựng. Mô hình chỉ giữ thông tin về sự có mặt hoặc tần suất của term; thứ tự, vị trí và cấu trúc cú pháp của chúng bị bỏ qua.

## Cách tạo biểu diễn
1. Tiền xử lý văn bản và token hóa thành các term nhất quán (ví dụ: chuyển chữ thường, lọc stop word, và đưa biến thể từ về dạng gốc khi phù hợp).
2. Lập từ vựng $V=\{t_1,t_2,\ldots,t_{|V|}\}$ từ tập văn bản hoặc corpus đang xét.
3. Với mỗi đơn vị văn bản $d$, đếm số lần mỗi term $t_j$ xuất hiện để tạo vector:

$$
\mathbf{x}_d=[f(t_1,d),f(t_2,d),\ldots,f(t_{|V|},d)],
$$

trong đó $f(t_j,d)$ là tần suất của $t_j$ trong $d$. Nếu chỉ quan tâm term có xuất hiện hay không, thay $f(t_j,d)$ bằng $1$ hoặc $0$ để được **binary BoW**.

Vector BoW thường rất thưa: mỗi văn bản chỉ chứa một phần nhỏ của toàn bộ từ vựng. Khi so sánh các văn bản có độ dài khác nhau, có thể chuẩn hóa vector hoặc thay tần suất thô bằng trọng số TF-IDF; TF-IDF vẫn dùng cùng không gian term nhưng giảm ảnh hưởng của những term phổ biến trong nhiều văn bản.

## Ví dụ
Với từ vựng $V=[\texttt{agent},\texttt{error},\texttt{log},\texttt{tool}]$ và hai block đã tiền xử lý:
- $b_1$: `agent log tool error` $\rightarrow [1,1,1,1]$
- $b_2$: `agent error error` $\rightarrow [1,2,0,0]$
Hai vector có các phần tử khác $0$ chung tại `agent` và `error`; vì vậy chúng có một mức tương đồng từ vựng nhất định. Tuy nhiên, BoW không phân biệt `agent error` với `error agent`, cũng không biết các từ đồng nghĩa như `failure` và `error` có nghĩa gần nhau.