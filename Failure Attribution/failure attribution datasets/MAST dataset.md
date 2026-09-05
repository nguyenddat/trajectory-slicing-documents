# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Bối cảnh chung về việc phát triển hệ LLM multi-agent (MAS) và nhu cầu nối đánh giá với cải thiện đã được trình bày tại [Bối cảnh chung](<Bối cảnh chung.md>). MAST nhấn mạnh một dấu hiệu cụ thể của khoảng trống này: dù MAS được kỳ vọng giúp phân rã tác vụ, chuyên môn hóa và phối hợp suy luận, mức tăng hiệu năng của chúng trên benchmark thường nhỏ so với single-agent hoặc cả baseline best-of-N. Do đó, điều cần hiểu không chỉ là hệ có điểm thấp, mà là **vì sao MAS thất bại**.

Theo tác giả, việc trả lời câu hỏi này cần nhận diện và phân tích có hệ thống các mẫu thất bại lặp lại trong execution trace. Tuy nhiên, lỗi trong MAS không có nguyên nhân gốc rõ ràng như phần mềm truyền thống: chúng có thể là hiệu ứng chồng lấp của hành vi model riêng lẻ, tương tác giữa các agent và thiết kế tổng thể của hệ. Đồng thời, chưa có một framework chuẩn với định nghĩa thống nhất để nhận diện và gán nhãn lỗi giữa các hệ khác nhau; vì thế việc so sánh và phân tích xuyên hệ không nhất quán.

Các công trình về agent workflow, memory hay state control chủ yếu giải một vấn đề cụ thể hoặc đưa ra tổng quan ở mức cao. Benchmark agent hiện có cần thiết nhưng chủ yếu nhìn từ trên xuống qua hiệu năng gộp hay các mục tiêu cấp cao như độ tin cậy và an toàn. Các taxonomy cho hội thoại nhiều lượt, sinh mã, hoặc tương tác người–agent cũng không nhắm tới failure trong quá trình thực thi tự chủ, tương tác và phối hợp của MAS. Theo tác giả, các hướng này vì vậy chưa cung cấp một taxonomy tinh vi, được rút ra từ dữ liệu, để giải thích nguyên nhân thất bại trên nhiều hệ và nhiều tác vụ.

## 1.2. Bài toán đặc trưng hóa failure mode của MAS

Nghiên cứu định nghĩa một MAS thất bại khi nó không đạt mục tiêu tác vụ dự định. Với execution trace của MAS — nơi các agent tương tác, sử dụng công cụ và đi tới kết quả — bài toán là nhận diện các **failure mode** quan sát được, gán chúng theo một vocabulary có định nghĩa thống nhất, và nêu lý do cho nhãn. Mục tiêu là từ các nhãn ở từng trace tạo được failure profile để phân tích động lực thất bại, so sánh hệ thống và chỉ ra chỗ thiết kế cần cải thiện.

Ràng buộc quan trọng của bài toán là một trace có thể chứa nhiều failure mode, còn các hành vi bề ngoài giống nhau có thể bắt nguồn từ những dạng lỗi khác nhau. Việc gán nhãn vì vậy không chỉ là phát hiện một lỗi đơn lẻ hay suy ra một nguyên nhân duy nhất, mà phải phân biệt các mẫu về **thiết kế hệ thống**, **lệch phối hợp liên-agent** và **xác minh tác vụ**. Tác giả cũng không phát biểu bài toán như quy kết nhân quả tới một agent và một bước lỗi quyết định, như Who&When; đầu ra của MAST là các failure mode và lý do của chúng trong trace.

## 1.3. Nghiên cứu nguồn

- [Cemri et al. (2025), *Why Do Multi-Agent LLM Systems Fail?*](<Why Do Multi-Agent LLM Systems Fail.md>)

# 2. Thiết kế dữ liệu MAST-Data

## 2.1. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

MAST-Data gồm 1.642 execution trace được gán nhãn từ bảy MAS mã nguồn mở: MetaGPT, ChatDev, HyperAgent, AppWorld, AG2, Magentic-One và OpenManus. Các trace bao quát tác vụ lập trình, giải toán và tác vụ agent tổng quát. Bảng cấu hình của dataset dùng bốn họ model: GPT-4, Claude, Qwen2.5-Coder và CodeLlama. Dataset chỉ giữ những hệ có thể quan sát đầy đủ quá trình thực thi.

Trước khi mở rộng thành dataset, tác giả thu 150 trace từ năm hệ (HyperAgent, AppWorld, AG2, ChatDev và MetaGPT) trên tác vụ lập trình và giải toán. Tập ban đầu này là cơ sở thực nghiệm để tìm ra failure mode và xây [MAST Failure Taxonomy](<MAST Failure Taxonomy.md>), thay vì áp một bộ nhãn có sẵn lên dữ liệu.

## 2.2. Các bước sinh và gán nhãn

1. **Lấy mẫu để khám phá failure mode.** Sáu chuyên gia đọc kỹ 150 trace đầu, được chọn theo *theoretical sampling* để phủ các mục tiêu hệ thống và kiểu tương tác khác nhau. Họ dùng Grounded Theory: open coding hành vi lỗi, so sánh liên tục giữa các trace, ghi nhận insight và khái quát thành các failure mode; quá trình dừng khi trace mới không tạo thêm insight về mode mới.
2. **Chuẩn hóa taxonomy bằng gán nhãn độc lập.** Từ taxonomy sơ bộ, ba chuyên gia gán nhãn độc lập các nhóm gồm năm trace chọn ngẫu nhiên. Sau mỗi vòng họ thảo luận bất đồng và sửa định nghĩa, thêm, bỏ hoặc gộp mode cho tới khi đạt đồng thuận cao. Bước này tạo các nhãn MAST có thể áp dụng nhất quán giữa MAS khác nhau.
3. **Kiểm tra khả năng khái quát.** Tác giả áp taxonomy đã chốt cùng quy trình gán nhãn người cho hai MAS và benchmark chưa xuất hiện trong giai đoạn phát triển — OpenManus/Magentic-One và MMLU/GAIA — trước khi dùng chúng trong tập lớn.
4. **Mở rộng gán nhãn bằng LLM.** LLM annotator dùng o1 nhận execution trace, định nghĩa MAST và few-shot examples từ dữ liệu đã được người gán nhãn; nó trả về các failure mode trong trace kèm lý do. Tác giả kiểm định pipeline này trên tập giữ lại có nhãn chuyên gia, rồi dùng nó để gán nhãn toàn bộ 1.642 trace. Họ đồng thời công bố MAST-Data-human, gồm các trace trong nghiên cứu đồng thuận được chuyên gia gán nhãn kèm giải thích.

## 2.3. Mục đích và đánh đổi của thiết kế

Tác giả dùng phân tích thủ công trước để failure mode được rút ra từ nhiều hệ và tác vụ, giảm nguy cơ taxonomy chỉ phản ánh một framework. Việc này cần thiết vì lỗi MAS thường là kết quả chồng lấp giữa hành vi model, tương tác agent và thiết kế hệ, nên khó xác minh nguyên nhân gốc; đồng thời chưa có định nghĩa chuẩn để gán nhãn xuyên hệ. Đổi lại, riêng giai đoạn đọc 150 trace đã tốn hơn 20 giờ cho mỗi chuyên gia, và việc giải quyết bất đồng ở các vòng đồng thuận tốn thêm khoảng 10 giờ.

LLM annotator là lựa chọn để mở rộng từ taxonomy đã được kiểm định sang hơn 1.600 trace, nhưng được đặt sau — không thay cho — bước xây và hiệu chỉnh nhãn bằng chuyên gia. Dataset cũng đánh đổi độ bao quát lấy khả năng gán nhãn đáng tin cậy: tác giả không đưa các hệ đóng như Manus vào tập chính vì không có đầy đủ trace nội bộ và thông tin model cần để phân tích chi tiết. Tác giả cũng nêu rõ MAST là bước đầu hướng tới cách hiểu thống nhất, không tuyên bố bao phủ mọi mẫu failure có thể có.

# 3. Đánh giá và insight

## 3.1. Failure profile là đặc thù của từng hệ

Distribution failure mode khác biệt rõ giữa các MAS và liên hệ chúng với kiến trúc. 

=> Không có một cách sửa chung cho mọi MAS; nên dùng failure profile để chọn điểm can thiệp thay vì chỉ dựa vào success rate gộp. 

## 3.2. Thiết kế hệ và model cùng định hình lỗi

Giữ kiến trúc, thay model nền => thay đổi profile lỗi

Giữ model và benchmark, thay kiến trúc MAS => trade-off khác nhau giữa lỗi thiết kế, lệch phối hợp và verification.

=> Không thể quy toàn bộ lỗi cho năng lực của LLM nền. Tuy nhiên, cải thiện prompt hay workflow đơn lẻ chỉ giúp một phần và không nhất quán giữa thiết lập; độ tin cậy cao cần phối hợp cải tiến ở cấp kiến trúc lẫn model.

## 3.3. Verification là điểm yếu độc lập với task success

Verification vẫn nhiều ở các model khác nhau, kể cả trong hệ đã có agent hoặc pha kiểm tra riêng. Một verifier chỉ kiểm tra bề mặt, chẳng hạn code có chạy hay không, có thể bỏ qua việc sản phẩm có thỏa mục tiêu tác vụ hay các edge case không. 

Các trace thành công không hẳn không có lỗi: failure mode liên quan verification có thể xuất hiện ở cả run thành công lẫn thất bại. 

## 3.4. Độ khó tác vụ và loại failure phải được xét cùng nhau

Tác giả giữ nguyên AG2 và GPT-4o, rồi chỉ thay benchmark từ GSM sang MMLU và OlympiadBench. Khi bài toán khó hơn, số failure trung bình tăng; hơn nữa, tỷ lệ lỗi giữa các nhóm cũng đổi, chẳng hạn GSM có ít lỗi về specification và lệch phối hợp hơn hai benchmark còn lại. Vì vậy, một failure profile không chỉ phản ánh kiến trúc MAS hay model nền mà còn chịu ảnh hưởng của loại và độ khó tác vụ. Khi thấy một hệ có nhiều lỗi trên một benchmark, không thể quy ngay chúng cho thiết kế hệ mà cần đối chiếu trên các tác vụ tương đương.

## 3.5. Taxonomy chi tiết hữu ích nhưng làm tự động hóa khó hơn

Một số failure mode cụ thể có triệu chứng gần nhau lại tương quan vừa phải; 

=>  LLM annotator có thể nhầm lẫn các nguyên nhân gốc khác nhau đó. Vì thế, độ chi tiết giúp diagnosis có hành động hơn, nhưng cũng đòi hỏi định nghĩa nhãn và kiểm định annotator chặt chẽ hơn.

# 4. Limitations và Future Works

## 4.1. Limitations

Bài không có section *Limitations* riêng. Các giới hạn dưới đây là những điểm tác giả trực tiếp nêu trong phần taxonomy, phân tích gán nhãn và thảo luận giải pháp.

1. **Taxonomy chưa thể bao quát mọi failure pattern.**
2. **Độ chi tiết của MAST gây khó cho gán nhãn tự động.** 
## 4.2. Future Works

Bài cũng không đưa ra roadmap *Future Work* riêng. Trong Appendix G, tác giả nêu các hướng nghiên cứu mở sau để xây MAS bền vững hơn:

1. **Verification nhiều tầng và theo domain.** Kết hợp kiểm tra đúng đắn mức thấp với xác minh mục tiêu mức cao; khai thác kiến thức ngoài, kết quả kiểm thử trong quá trình sinh, và cơ chế kiểm tra phù hợp cho từng loại tác vụ.
2. **Giao thức giao tiếp có cấu trúc hơn.** Chuẩn hóa ý định và tham số trong thông điệp agent để giảm mơ hồ, đồng thời hỗ trợ kiểm tra tính nhất quán của tương tác.
3. **Huấn luyện agent theo vai trò và năng lực phối hợp.** Tác giả nêu fine-tuning bằng reinforcement learning theo vai trò; riêng failure lệch phối hợp có thể cần cả cải tiến kiến trúc lẫn năng lực suy luận về nhu cầu thông tin của agent khác.
4. **Định lượng bất định.** Dùng confidence measure và ngưỡng thích nghi để agent chỉ hành động khi đủ tin cậy, hoặc dừng lại để thu thập thêm thông tin khi confidence thấp.
5. **Quản lý memory và state cho tương tác nhiều agent.** Phát triển cơ chế duy trì ngữ cảnh và trạng thái dùng chung nhằm giảm mơ hồ trong giao tiếp; tác giả lưu ý nghiên cứu hiện tại vẫn chủ yếu tập trung vào single-agent.
