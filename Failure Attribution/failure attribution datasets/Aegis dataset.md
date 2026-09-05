# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Bối cảnh chung về tính mong manh của hệ LLM multi-agent (MAS), việc một sai sót có thể lan truyền qua tương tác giữa các agent, và nhu cầu nối kết quả đánh giá với thành phần cần cải thiện đã được trình bày tại [Bối cảnh chung](<Bối cảnh chung.md>). AEGIS tập trung vào trở ngại thực dụng để tự động hóa bước chẩn đoán này: muốn quy kết một failure về **agent chịu trách nhiệm** và **loại lỗi** của họ, cần dữ liệu trajectory có nhãn chi tiết ở quy mô lớn và đa dạng.

Theo tác giả, các resource hiện có đều dựa vào chuyên gia đọc execution log phức tạp. [Who&When](<Failure Attribution - Formulation.md>) chỉ có 184 lỗi được gán nhãn; [MAST](<MAST dataset.md>) phân tích hơn 150 tác vụ để rút ra 14 error mode; còn [TRAIL](<Trail dataset.md>) có 148 trace và 841 lỗi gán nhãn theo cách bài AEGIS tường thuật. Các bộ này cho thấy failure attribution cần xét hành vi bên trong trajectory thay vì chỉ outcome cuối, nhưng quy trình gán nhãn thủ công tốn kém nên không thể mở rộng dễ dàng sang nhiều kiến trúc MAS và miền tác vụ.

Tác giả gọi đây là một **scalability deadlock**: các LLM mạnh hiện tại vẫn hạn chế ở error attribution, trong khi dữ liệu lớn theo tác vụ có thể giúp cải thiện chúng lại quá đắt để tạo bằng annotation người. Vì vậy, khoảng trống AEGIS nhắm tới không phải phát hiện thêm một failure mode mới, mà là tạo được dữ liệu failure có nhãn attribution đáng tin cậy, có thể tái lập và mở rộng để huấn luyện/evaluate model chẩn đoán. Bài lập luận rằng thành công của sinh dữ liệu tự động ở reasoning và software engineering mở ra cơ hội chưa được khai thác cho bài toán này.

## 1.2. Bài toán fine-grained error attribution

AEGIS phát biểu **fine-grained error attribution** trên failed trajectory: quy kết các agent có lỗi cùng error mode tương ứng, thay vì chọn một decisive error step. Formalization đầy đủ và đối chiếu với Who&When được tách tại [Failure Attribution – Formulation](<Failure Attribution - Formulation.md#2-fine-grained-error-attribution-trong-aegis>).

Với AEGIS, ràng buộc quyết định nằm ở dữ liệu cho bài toán này: gold label (ground truth) phải đủ chính xác để làm tín hiệu học, đồng thời sinh được ở quy mô lớn mà không lặp lại nút thắt annotation người. Do đó, nghiên cứu cần tạo failure có kiểm soát từ execution thành công để nhãn agent–error mode có thể suy ra và kiểm chứng.

## 1.3. Nghiên cứu nguồn

- [Kong et al. (2025), *Aegis: Automated Error Generation and Attribution for Multi-Agent Systems*](<Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>)

# 2. Thiết kế dữ liệu AEGIS

## 2.1. Đặc trưng cơ bản

Mỗi mẫu chính của AEGIS là một **failed MAS trajectory** kèm attribution map: các faulty agent và tập error mode của từng agent. Dataset cuối có **9.533 trajectory lỗi**, chứa **24.843 error instance** (trung bình khoảng 2,6 lỗi được chèn mỗi trajectory). Đây là số trajectory lỗi được báo cáo; mỗi mẫu được sinh từ một trajectory thành công tương ứng, nên cấu trúc dữ liệu còn tạo các cặp correct–faulty để dùng cho contrastive learning.

Dataset phủ sáu benchmark/tác vụ và sáu framework MAS:

| Chiều bao phủ | Nguồn/cấu hình                                                                                                                                                                                                                                              |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tác vụ        | MATH và GSM8K (toán), HumanEval (sinh/kiểm thử mã), SciBench (khoa học), MMLU-Pro (kiến thức đa lĩnh vực), GAIA (tác vụ trợ lý tổng quát).                                                                                                                  |
| Kiến trúc MAS | MacNet (cấu hình chain/star/tree/fully-connected), DyLAN (đồ thị động), LLM Debate (tranh luận và tổng hợp đồng thuận), AgentVerse (solver–critic–evaluator phân cấp), Magentic-One (orchestrator–executor), SmolAgents (multi-step ReAct có tool calling). |
| Nhãn lỗi      | 14 error mode từ [MAST Failure Taxonomy](<MAST Failure Taxonomy.md>), thuộc ba nhóm lớn: specification issues, inter-agent misalignment và task verification failures.                                                                                   |

Tất cả agent trong lần chạy baseline dùng GPT-4o-mini với temperature 0 để giảm ngẫu nhiên; adaptive manipulator chạy ở temperature 0,7 để sinh can thiệp đa dạng. Bài dùng MASLab làm codebase chung; với Magentic-One và SmolAgents không có sẵn trong MASLab, tác giả phát triển trực tiếp trên source code của chúng.

## 2.2. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

AEGIS bắt đầu từ các task trong sáu benchmark trên mà một MAS giải đúng trong cấu hình xác định. Với mỗi task, tác giả cố định policy chạy — ví dụ temperature bằng 0 và random seed cố định — rồi thu một correct reference trajectory. Cách chọn điểm xuất phát thành công này nhằm để failure sinh sau đó có thể quy về can thiệp, thay vì lẫn với lỗi vốn có của lần chạy gốc.

Từ mỗi trajectory đúng, họ tạo nhiều faulty counterpart theo các **injection plan** khác nhau. Một plan chỉ định tập các cặp `(target agent, target error modes)`, nên có thể tiêm một hay nhiều lỗi, vào một hay nhiều agent. Việc tiêm không sửa MAS gốc: wrapper không xâm lấn, dựa trên monkey patching, chặn hành vi của agent đích để sửa context hoặc thay response.

Adaptive manipulator dùng LLM để sinh sửa đổi phù hợp task và error mode đã chọn. Nó dùng ngẫu nhiên một trong hai chiến lược: **prompt injection**, thay input state của agent trước khi hành động; hoặc **response corruption**, thay action/response đúng bằng một action/response lỗi. Ví dụ, cùng một mode có thể thành phép tính sai nhưng có vẻ hợp lý ở task toán, hoặc vòng lặp vô hạn ở task lập trình.

## 2.3. Các bước sinh và gán nhãn

1. **Thu trajectory thành công có thể tái lập.** Chạy từng MAS trên task nguồn trong cấu hình xác định và chỉ lấy các run đạt outcome thành công. Trajectory này là baseline và positive anchor của các faulty variant.
2. **Lập kế hoạch lỗi.** Chọn một injection plan gồm agent đích và một hay nhiều mode trong 14 mode MAST; cùng trajectory đúng có thể có nhiều plan để tạo các failure variant khác nhau.
3. **Can thiệp theo ngữ cảnh trong khi chạy lại.** Khi agent đích đến lượt, wrapper gọi manipulator để prompt-inject context hoặc corrupt response; các agent và bước không phải đích tiếp tục chạy trong MAS gốc. Với DyLAN, tác giả thêm post-hoc label refinement vì topology/tương tác động cần kiểm tra lại fidelity của nhãn.
4. **Đánh giá outcome và lọc.** Chỉ giữ trajectory nếu can thiệp thật sự làm hệ thất bại. Evaluator theo từng benchmark kiểm tra đáp án số, test code, lựa chọn trắc nghiệm hoặc — với GAIA — so sánh ngữ nghĩa bằng GPT-4o-mini; các run vẫn thành công không vào tập failure.
5. **Gán gold label theo plan.** Với trajectory được giữ, tác giả đặt attribution map bằng đúng tập lỗi đã dự kiến trong injection plan. Nhãn vì vậy được chương trình suy ra từ can thiệp, thay vì annotator phải đọc toàn bộ log để tìm lại agent và error mode.
6. **Kiểm tra fidelity của nhãn.** Ba chuyên gia, không thấy programmatic label, độc lập phân loại root-cause error của 100 trajectory Aegis-Bench theo 14 mode MAST. Tác giả báo cáo Fleiss' \(\kappa=0{,}85\) giữa người–người và \(\kappa=0{,}81\) giữa nhãn chương trình với người, như kiểm tra rằng lỗi được tiêm vẫn được người đọc nhận ra theo taxonomy.

## 2.4. Mục đích và đánh đổi của thiết kế

Thiết kế correct-to-faulty giải nút thắt annotation: biết rõ intervention plan giúp tác giả tạo nhãn agent–error mode tái lập ở quy mô lớn, đồng thời giữ positive–negative pairs cho ba cách học mà bài xét. Việc dùng taxonomy [MAST Failure Taxonomy](<MAST Failure Taxonomy.md>) — vốn được rút ra từ failure tự nhiên — nhằm khiến can thiệp dựa vào những pattern đã quan sát thay vì lỗi bịa ngẫu nhiên. Bao phủ nhiều benchmark và topology MAS nhằm tránh dataset chỉ học một workflow hay một miền task.

Đổi lại, gold label là nhãn của **lỗi được tiêm có chủ đích**, không phải kết quả của việc khám phá mọi failure tự phát trong trajectory. Tác giả cũng thừa nhận việc dùng taxonomy có sẵn và injection plan được kiểm soát hạn chế khả năng bao quát các failure phức tạp, mới nổi. Do đó họ kiểm tra fidelity với annotator và so sánh phân bố semantic của AEGIS-Bench với [MAST](<MAST dataset.md>) và [Who&When](<Failure Attribution - Formulation.md>); các kiểm tra này là bằng chứng hỗ trợ tính giống dữ liệu thực, không biến dữ liệu tổng hợp thành log failure tự nhiên.

# 3. Đánh giá và insight

## 3.1. Thiết lập đánh giá

Tác giả lấy 100 trajectory từ mỗi trong sáu benchmark để lập **Aegis-Bench** test set; dữ liệu còn lại được chia train/validation 80/20. Họ đánh giá khả năng khái quát ngoài phân bố trên [Who&When](<Failure Attribution - Formulation.md>), đồng thời so sánh model proprietary/open-weight zero-shot với model học từ AEGIS bằng [Aegis-SFT](<Aegis-SFT.md>), [Aegis-GRPO](<Aegis-GRPO.md>) và contrastive learning [DCL](<DCL.md>).

Vì một trajectory có thể có nhiều agent và mode lỗi, đánh giá tách ba mức: **Pair** (đúng cặp agent–mode), **Agent** (đúng agent, bỏ qua mode) và **Error** (đúng mode, bỏ qua agent). Mỗi mức dùng cả Micro-F1 và Macro-F1; Macro-F1 đặc biệt cho biết model có xử lý được mode hiếm thay vì chỉ các nhãn phổ biến hay không.

## 3.2. Dữ liệu tổng hợp có thể dạy kỹ năng attribution, nhưng không thay thế hoàn toàn failure tự nhiên

Theo tác giả, SFT trên AEGIS đem lại cải thiện lớn nhất và một model 14B được fine-tune vượt các baseline foundation lớn hơn trong điểm tổng hợp. Cải thiện cũng xuất hiện trên Who&When, nên họ xem đây là bằng chứng rằng correct-to-faulty synthesis dạy được một phần kỹ năng chẩn đoán chuyển sang benchmark có nhãn người.

Tuy nhiên, transfer không đồng đều ở mọi mức nhãn. Kết quả appendix cho tương quan mạnh hơn giữa hai benchmark ở việc nhận diện agent phổ biến và error mode, trong khi Agent Macro-F1 chịu ảnh hưởng của các role hiếm khác nhau giữa dataset, còn Pair-level rất thấp và khó ổn định. Vì vậy, AEGIS và Who&When bổ sung cho nhau chứ không thể thay thế lẫn nhau như một benchmark duy nhất.

## 3.3. Nhận diện *ai* dễ hơn giải thích *vì sao*

Trên các model, Agent-level attribution thường cao hơn Error-level attribution; Pair-level lại khó nhất vì buộc ghép đúng cả hai.

Chênh lệch Micro-F1 cao hơn Macro-F1 cũng cho thấy model học được failure thường gặp nhưng vẫn yếu ở các mode hiếm, cụ thể và dài đuôi.

## 3.4. Độ khó phụ thuộc đồng thời vào task và kiến trúc MAS

Hiệu quả không cố định theo một model hay một dataset. SFT có lợi thế rộng ở phần lớn miền, nhưng chênh lệch thu hẹp ở GAIA — tác vụ trợ lý tổng quát. Theo kiến trúc, các model nhìn chung dễ attribution hơn ở framework có cấu trúc như Debate và AgentVerse, còn DyLAN và MacNet với topology phức tạp khó hơn; chính các trường hợp khó này lại nhận mức cải thiện lớn hơn khi fine-tune trên AEGIS.

Tác giả cũng quan sát trade-off ở framework ít được đại diện hơn: SFT đôi khi giảm nhẹ so với base model trên Magentic-One và SmolAgents, dấu hiệu over-specialization; GRPO ổn định hơn trên các hệ này. Kết luận của họ không phải một cách học luôn thắng, mà việc chọn cách huấn luyện phải xét cả mức đại diện của kiến trúc và domain trong data.

## 3.5. Tín hiệu lỗi thưa cần ràng buộc biểu diễn và logic

Kết quả ablation DCL cho thấy bỏ semantic guidance từ prototype error/agent hoặc bỏ hierarchical consistency đều làm giảm chất lượng.

=> Contrastive learning không chỉ cần phân biệt trajectory đúng–sai, mà còn phải giữ quan hệ hợp logic giữa dự đoán agent, error mode và cặp của chúng.

# 4. Limitations và Future Works

## 4.1. Limitations

Bài không có section *Limitations* riêng. Trong conclusion, tác giả trực tiếp thừa nhận hai ràng buộc gắn với cách xây AEGIS:

1. **Taxonomy định trước giới hạn không gian lỗi.** AEGIS tiêm 14 mode của [MAST Failure Taxonomy](<MAST Failure Taxonomy.md>), nên dataset không tự khám phá những mode chưa nằm trong taxonomy đó.
2. **Injection plan có kiểm soát chưa bao quát failure mới nổi.** Failure được tạo từ intervention định trước trên trajectory thành công; vì vậy, tác giả nêu rõ hướng này còn hạn chế trước các failure phức tạp, emergent hơn trong MAS.

Ngoài limitation được nêu ở conclusion, phần kết quả cho thấy hai caveat thực nghiệm. Thứ nhất, với các lỗi tinh vi và tích lũy, tất cả model được thử — kể cả AEGIS — vẫn có thể quy kết sai hoặc không tìm được nguyên nhân gốc. Thứ hai, SFT tiếp tục tăng điểm trên Aegis-Bench nhưng giảm trên [Who&When](<Failure Attribution - Formulation.md>) sau epoch thứ hai; tác giả quy hiện tượng này cho overfitting vào pattern hình thức của dữ liệu tổng hợp. Đây là giới hạn quan sát được của learner/dataset, không phải một limitation section độc lập của bài.

## 4.2. Future Works

Bài cũng không có roadmap *Future Work* chi tiết. Hướng tương lai mà tác giả nêu trực tiếp là mở rộng từ taxonomy định trước và injection có kiểm soát sang các **failure phức tạp, emergent** hơn. Họ đặt mục tiêu dài hạn là hệ agentic có thể tự sửa chữa (*self-repairing*); bài không mô tả thuật toán hay lộ trình cụ thể để đạt mục tiêu này.
