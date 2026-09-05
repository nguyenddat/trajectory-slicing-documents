## Giới thiệu

TextTiling là một [[Text Segmentation/Linear/Unsupervised Methods|phương pháp unsupervised]] để phân đoạn văn bản ở cấp đoạn văn, dựa trên việc phát hiện **chuyển đổi tiểu chủ đề** (_subtopic shift_). Phương pháp không học từ nhãn ranh giới; mục tiêu là chia văn bản thuyết minh thành các **phân đoạn tiểu chủ đề** (_subtopic segments_) gồm nhiều đoạn văn liên tiếp.

Trong văn bản kỹ thuật, cấu trúc tiểu chủ đề đôi khi được thể hiện rõ qua tiêu đề và tiêu đề phụ. Tuy nhiên, nhiều văn bản dài thiếu các dấu hiệu cấu trúc này, khiến việc tự động phát hiện ranh giới tiểu chủ đề trở nên hữu ích.
### Đóng góp chính
TextTiling khai thác **sự đồng xuất hiện và phân bố từ vựng** để phát hiện cấu trúc tiểu chủ đề qua ba bước:
1. [[Text Segmentation/Linear/Techniques/Tokenization|Tokenization]]: chuyển văn bản thành các thuật ngữ và các đơn vị văn bản có kích thước gần tương đương câu.
2. [[Text Segmentation/Linear/Techniques/Similarity|Similarity]]: tính điểm liên kết từ vựng giữa các đơn vị.
3. Xác định ranh giới tiểu chủ đề tại các **điểm lõm sâu nhất** (_depth valleys_) trên đường biểu diễn điểm số.

Về tổng thể, cấu trúc hóa văn bản ở cấp đoạn gồm hai nhiệm vụ:
4. Phát hiện các phân đoạn tiểu chủ đề.
5. Xác định và gán nhãn chủ đề cho từng phân đoạn.

Nghiên cứu này chỉ tập trung vào **phát hiện ranh giới phân đoạn**.

## Why Multi-paragraph Units?
Phần này giải thích vì sao nên chia văn bản thành các đơn vị gồm nhiều đoạn thay vì chỉ dựa vào đoạn văn có sẵn.

Ý chính:
- Đoạn văn không phản ánh chuyển chủ đề; đôi khi chỉ để dễ đọc/trình bày.
- Phân đoạn nhiều đoạn có thể tạo ra các “passage” tương ứng hơn với một tiểu chủ đề hoàn chỉnh.
- Ứng dụng cho hiển thị/hypertext: giúp chia tài liệu dài thành các phần có ý nghĩa, dễ điều hướng và đọc trên màn hình.
- Ứng dụng cho truy hồi thông tin: so khớp truy vấn với passage liên quan có thể hữu ích hơn so với toàn bộ tài liệu dài.
- Cũng có tiềm năng cho tóm tắt văn bản và sinh văn bản tự động.

Tác giả nhấn mạnh TextTiling phù hợp nhất với văn bản thuyết minh dài, ít cấu trúc sẵn như bài báo khoa học phổ thông hoặc báo cáo.

Lợi ích:
- Hiển thị văn bản trực tuyến và hypertext: TextTiling chia tài liệu dài thành các passage có ý nghĩa, giúp người dùng đọc, điều hướng và chuyển tài liệu sang hypertext hiệu quả hơn.
- Truy hồi thông tin: TextTiling giúp hệ thống so khớp truy vấn với passage liên quan thay vì toàn bộ tài liệu dài, từ đó dễ tìm và hiển thị phần phù hợp hơn.

Tác giả nhấn mạnh TextTiling phù hợp nhất với văn bản thuyết minh dài, ít cấu trúc sẵn như bài báo khoa học phổ thông hoặc báo cáo.

## Ý tưởng khởi nguồn
Trong cùng một tiểu chủ đề, một nhóm từ vựng có xu hướng xuất hiện tập trung; khi tiểu chủ đề thay đổi, nhóm từ đang hoạt động cũng thay đổi, làm độ tương đồng từ vựng giữa hai vùng kề nhau giảm xuống. Vì vậy, bài toán phân đoạn được chuyển thành việc tìm nơi một “cụm” từ vựng kết thúc và một cụm mới bắt đầu.

## Boundary Identification

Bước này nhận một chuỗi **lexical score** tại các gap giữa token-sequences và trả về các gap được chọn làm ranh giới tiểu chủ đề. Cách tính **depth score** là như nhau cho mọi phương pháp tạo lexical score.

### Đầu vào từ các bước trước

1. [[Text Segmentation/Linear/Techniques/Tokenization|Tokenization]] tạo các token-sequences và các gap giữa chúng.
2. [[Text Segmentation/Linear/Techniques/Similarity|Similarity]] tạo lexical score cho từng gap. Các cách tính lexical score khác cũng phải xuất ra cùng dạng: một score cho mỗi gap.

Từ [[Text Segmentation/Linear/Techniques/Similarity|Similarity]], đầu vào có thể được nhìn như một vector score theo các gap (tức hàng của một ma trận một chiều):

$$
\mathbf{S} =
\begin{array}{c|ccccc}
\text{gap } i & 1 & 2 & 3 & 4 & 5 \\ \hline
\text{lexical score } S(i) & 0.82 & 0.75 & 0.21 & 0.68 & 0.80
\end{array}.
$$

### 1. Làm mượt lexical-score plot

Trước khi tính depth score, TextTiling làm mượt dãy lexical score để loại các đáy nhỏ do nhiễu. Tại mỗi gap $g$, với một số chẵn nhỏ $s$, lấy trung bình score ở $s/2$ gap bên trái, score tại $g$, và $s/2$ gap bên phải:

$$
S'(g) = \frac{\sum_{j=g-s/2}^{g+s/2} S(j)}{s+1}.
$$

Lặp lại phép làm mượt này $n$ lần. Thiết lập mặc định trong bài là $s=2, n=1$, tức trung bình trượt ba điểm $(g-1, g, g+1)$ một lần. Làm mượt chỉ là một low-pass filter; tác giả lưu ý có thể dùng hàm lọc khác. Làm mượt quá mạnh có thể làm mờ chính vị trí chuyển tiểu chủ đề.

### 2. Tính depth score tại từng gap
Với mỗi gap $i$, tìm đỉnh cục bộ gần nhất ở hai phía trên dãy score đã làm mượt:
- $l$: đỉnh bên trái, được tìm bằng cách đi sang trái cho đến khi $S(l-1) < S(l)$;
- $r$: đỉnh bên phải, được tìm bằng cách đi sang phải cho đến khi $S(r+1) < S(r)$.

Depth score là tổng độ tụt từ hai đỉnh đó xuống gap $i$:

$$
D(i) = [S(l)-S(i)] + [S(r)-S(i)].
$$

Hai hiệu này không âm vì $l$ và $r$ là các peak được chọn; do đó không cần viết giá trị tuyệt đối. Ví dụ, với $i=3$ trong vector ở trên, nếu peak trái là $0.82$ và peak phải là $0.80$ thì:

$$
D(3)=(0.82-0.21)+(0.80-0.21)=1.20.
$$

Output trung gian là một dãy `gap → depth score`. Depth lớn nghĩa là gap nằm ở đáy sâu tương đối so với cả hai phía: tín hiệu từ vựng đã giảm ở phía trái và tăng trở lại ở phía phải, nên đây là ứng viên mạnh cho chuyển tiểu chủ đề.

### 3. Chọn ranh giới từ depth score
“Lớn” không phải một ngưỡng tuyệt đối dùng chung cho mọi tài liệu. TextTiling xét phân bố depth score trong chính tài liệu, với $\mu$ là trung bình và $\sigma$ là độ lệch chuẩn:
- **Liberal cutoff (LC):** chọn khi $D(i) > \mu-\sigma$; nhiều boundary hơn, thiên về recall.
- **Conservative cutoff (HC):** chọn khi $D(i) > \mu-\sigma/2$; ít boundary hơn, thiên về precision.

Các gap sau khi lọc được sắp theo depth score. Nếu văn bản có paragraph break, ranh giới có thể được dời về paragraph break gần nhất. Thuật toán cũng không cho phép hai ranh giới quá gần nhau: cần ít nhất ba token-sequences xen giữa.

### Trường hợp cần lưu ý
- Một đáy nhỏ có thể làm gián đoạn một đáy sâu hơn. Làm mượt giúp giảm lỗi này; đồng thời depth score thấp nếu một phía không có peak đủ cao.
- Một đáy dài và phẳng (_plateau_) biểu thị thay đổi từ vựng rất từ từ. Plateau dài có thể hợp lý khi chọn hai mép làm boundary; plateau ngắn buộc thuật toán chọn gần đúng và có thể cần thêm tín hiệu discourse.
- Không ưu tiên dùng độ dốc của thung lũng: một đáy sâu nhưng đổi từ từ thường biểu thị chuyển tiểu chủ đề tốt hơn một đáy nông, dốc do digression ngắn.

## Parameter Settings

Cấu hình mặc định mà bài dùng cho nhiều loại văn bản là:

| Ký hiệu | Giá trị mặc định | Vai trò |
|---|---:|---|
| $w$ | 20 | Số term trong một token-sequence. |
| $k$ | 10 | Số token-sequences ở mỗi block trái/phải khi dùng [[Text Segmentation/Linear/Techniques/Similarity|block comparison]]. |
| $s$ | 2 | Bề rộng làm mượt; tương ứng trung bình trượt ba gap. |
| $n$ | 1 | Số lượt làm mượt. |
| cutoff | LC hoặc HC | Quy tắc chọn boundary từ depth score: $D(i)>\mu-\sigma$ hoặc $D(i)>\mu-\sigma/2$. |

Các tham số phụ thuộc lẫn nhau. Tăng kích thước block $k$ làm phép so sánh bao quát ngữ cảnh rộng hơn; theo tác giả, điều này cần ít làm mượt hơn, tìm được ít boundary hơn và tạo phân đoạn thô hơn. Chỉ nên làm mượt vừa phải vì làm mượt quá mạnh có thể che mất vị trí chuyển tiểu chủ đề.

## Đánh giá

Các metric đánh giá được dùng trong bài, gồm precision, recall và kappa agreement, được tách tại [[Evaluation Metrics|Evaluation Metrics]].

## Analysis

### Kết quả và giới hạn được tác giả nêu

- **Block comparison mạnh hơn vocabulary introduction trong phần lớn văn bản thử nghiệm.** Khi so sánh ở cùng mức recall của người đọc, tác giả nhận thấy block comparison thường cho precision cao hơn. Tuy nhiên, vocabulary introduction tốt hơn ở một số văn bản mà block comparison phát hiện rất ít boundary; hai cách chấm score có tính bổ sung trong các trường hợp khó.
- **Ngưỡng chọn boundary kiểm soát trade-off chứ không giải quyết tính mơ hồ của ranh giới.** LC gán nhiều boundary hơn nên recall tăng và precision giảm; HC cho chiều ngược lại. Vì vậy cutoff cần được chọn theo chi phí tương đối của bỏ sót và chia quá mức, không phải theo một “độ sâu đúng” cố định.
- **Mức đồng thuận của người đọc là một giới hạn của gold label.** Một đoạn tóm tắt cuối văn bản có thể là boundary hợp lý theo thay đổi từ vựng nhưng không được đa số người đọc tách ra. Kết quả đó bị tính là lỗi, cho thấy precision/recall với nhãn người đọc không hoàn toàn tương đương với chất lượng cấu trúc ngữ nghĩa.
- **TextTiling phụ thuộc vào giả định “chuyển tiểu chủ đề đi kèm đổi vốn từ”.** Đoạn tóm tắt tái sử dụng từ vựng của phần trước, trích dẫn lời nói, các giai thoại ngắn, hay chuyển chủ đề mà tín hiệu chính là đại từ đã bị stop list loại bỏ đều có thể làm lexical signal yếu hoặc sai. Tác giả ghi nhận các văn bản có phong cách khác biệt và nhiều lời thoại là trường hợp hoạt động kém.
- **Nhạy với tham số và độ hạt.** Không có một cấu hình tối ưu cho mọi văn bản: thay đổi $w$, $k$, $s$, $n$ cải thiện một số văn bản nhưng giảm chất lượng ở văn bản khác. Điều này phù hợp với bản chất local của score: block lớn cho ngữ cảnh rộng và ít boundary hơn, còn block nhỏ nhạy với chuyển đổi cục bộ hơn.

### 6.5. Detecting Breaks between Consecutive Documents

Phần 6.5 kiểm tra một bài toán khác: nối nhiều bài báo độc lập thành một luồng rồi dùng TextTiling để tìm **ranh giới giữa các bài báo**, thay vì ranh giới tiểu chủ đề bên trong một bài báo. Tác giả nhấn mạnh đây là phép kiểm tra có ích để so sánh với các nghiên cứu trước, nhưng **vi phạm giả định trung tâm** của TextTiling.

TextTiling xếp hạng valley theo thay đổi từ vựng **tương đối bên trong một tài liệu**. Thí nghiệm 6.5 lại mặc định ranh giới giữa tài liệu phải quan trọng hơn mọi chuyển tiểu chủ đề nội tại. Hai điều này có thể mâu thuẫn: một bài báo rất mạch lạc có thể có một subtopic shift nội bộ sâu hơn thay đổi từ vựng giữa hai bài báo có chủ đề tương tự. Khi đó thuật toán đánh dấu subtopic shift là false positive theo nhãn document boundary, dù đó là một boundary hợp lý cho bài toán gốc.

Thiết lập của tác giả dùng 44 bài *Wall Street Journal* năm 1989, bỏ các bài ngắn hơn 10 câu, tổng cộng 691 đoạn. Block comparison chạy với tham số mặc định; một dự đoán được tính đúng nếu cách ranh giới bài báo thật không quá ba câu. Kết quả theo số boundary được giữ lại sau khi xếp hạng depth score:

| Boundary giữ lại ($B$) | Đúng ($C$) | Precision | Recall |
|---:|---:|---:|---:|
| 10 | 8 | .80 | .19 |
| 20 | 16 | .80 | .37 |
| 30 | 22 | .73 | .51 |
| 40 | 27 | .68 | .63 |
| 43 | 29 | .67 | .67 |
| 50 | 31 | .62 | .72 |
| 60 | 36 | .60 | .83 |
| 70 | 41 | .59 | .95 |

Điểm đáng chú ý không chỉ là break-even tại $B=43$ với precision = recall = .67. Các boundary depth cao nhất gần như luôn là exact hit; khi hạ ngưỡng, các dự đoán đúng còn lại thường lệch một đến ba câu. Đồng thời, phần lớn false positive có depth cao thực ra là các chuyển tiểu chủ đề mạnh bên trong bài báo. Đây là bằng chứng trực tiếp cho thấy kết quả phụ thuộc vào định nghĩa boundary được chấm, không chỉ vào khả năng xếp hạng valley của thuật toán.

Các cấu trúc làm thiết lập này khó hơn gồm byline, chuỗi câu cô lập, bảng dữ liệu, danh sách lãi suất, và bài báo ghép nhiều mẩu tin. Chúng tạo các đổi dạng bề mặt hoặc khoảng cách từ vựng không tương ứng ổn định với document boundary.

### Hàm ý khi áp dụng cho trajectory slicing

Đây là suy luận áp dụng từ các quan sát trên, không phải kết luận được tác giả kiểm chứng trên trajectory:

- Cần xác định rõ **loại boundary đích** trước khi chọn TextTiling: chuyển mục tiêu cục bộ, chuyển pha (planning/tool use/verification), hay ranh giới giữa hai trajectory độc lập. Đánh giá một loại bằng nhãn của loại khác có thể biến các boundary hợp lý thành false positive, đúng như thí nghiệm 6.5.
- Nếu nối nhiều trajectory hoặc nhiều task vào cùng một chuỗi để chạy segmentation, không nên mặc định gap nối hai trajectory sẽ luôn có depth lớn hơn mọi chuyển pha nội tại. Cần giữ metadata về đầu/cuối trajectory hoặc đánh giá riêng bài toán document-boundary detection.
- Với log agent, tool output, template, message wrapper, danh sách, hay retry có thể đóng vai trò tương tự byline/bảng/liệt kê trong 6.5: chúng làm đổi token mạnh mà không nhất thiết là đổi mục tiêu. Cần chuẩn hoá hoặc gộp các artefact này trước khi tạo lexical score.
- Gold boundary nên kèm một giao thức gán nhãn chỉ rõ granularity và cách xử lý summary, retry, hoặc chuyển pha ngắn; nếu không, mức đồng thuận thấp của annotator sẽ giới hạn ý nghĩa của precision/recall.
