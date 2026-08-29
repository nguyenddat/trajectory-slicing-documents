# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Việc đánh giá hệ agentic để tìm nơi cần cải thiện, cũng như phần chẩn đoán thủ công từ log, đã được trình bày tại [Bối cảnh chung](<../Bối cảnh chung.md>). TRAIL đặt trọng tâm khác: cần **quan sát và đánh giá chính bản ghi thực thi** của workflow agentic ở độ phân giải đủ nhỏ để hỗ trợ debug và tìm nguyên nhân gốc. Phạm vi của bài không chỉ là multi-agent mà gồm cả workflow single-agent và multi-agent.

Theo tác giả, các benchmark agent hiện có chủ yếu đánh giá kết quả đầu-cuối; cách này không cho đủ khả năng quan sát tính không xác định và quá trình giải bài nhiều bước của agent. Việc đọc thủ công các trace dài, theo miền ứng dụng, vừa không mở rộng theo số lượng và độ phức tạp của lần chạy, vừa khó hơn debug phần mềm truyền thống vì trace đan xen suy luận của LLM với đầu ra từ công cụ bên ngoài.

Khoảng trống còn nằm ở dạng dữ liệu và phạm vi lỗi cần quan sát. Các framework phân tích trace trước đó chủ yếu dùng trace văn bản đã được parse, trong khi framework agent phổ biến ghi trace có cấu trúc theo chuẩn như OpenTelemetry; tác giả dẫn các nghiên cứu cho thấy LLM vẫn khó xử lý dạng dữ liệu này. Về taxonomy/benchmark, [MAST](<MAST dataset.md>) tập trung vào lỗi reasoning và coordination, còn ACPBench dùng dữ liệu tổng hợp để kiểm tra suy luận nguyên tử về hành động và kỹ năng planning. Theo tác giả, hai hướng này chưa bao quát các lỗi thực thi hệ thống và lập kế hoạch xuất hiện trong workflow thực tế — chẳng hạn lỗi API hay điều phối tác vụ — vốn cũng hữu ích cho người dùng và kỹ sư tối ưu ứng dụng agentic. Họ cũng yêu cầu benchmark bám vào ứng dụng thực tế, thay vì xoay quanh dữ liệu giả lập.

## 1.2. Bài toán đánh giá và định vị lỗi trên trace

TRAIL cụ thể hóa nhu cầu trên thành bài toán cho một LLM-judge: với **toàn bộ trace thực thi dài, có cấu trúc** của một workflow agentic (các span LLM và công cụ được ghi theo OpenTelemetry/OpenInference), hãy tìm **tất cả span có lỗi**. Với mỗi lỗi, đầu ra cần gán nhãn ở lá của taxonomy, định vị bằng `span_id`, và nêu bằng chứng, mô tả cùng mức ảnh hưởng. Taxonomy bao phủ ba nhóm lỗi: reasoning, thực thi hệ thống, và planning/coordination. Bài toán cũng yêu cầu đánh giá toàn trace theo reliability, security, tuân thủ chỉ dẫn và độ tối ưu của kế hoạch.

Mục tiêu không phải chỉ kết luận workflow thành công hay thất bại, mà là đánh giá có hệ thống và định vị lỗi ở cấp span để kết quả dùng được cho debug hoặc root-cause analysis. Ràng buộc trọng yếu là trace giữ nguyên dạng có cấu trúc, bám vào lần chạy thực tế, và có thể rất dài — thậm chí vượt context window của một số LLM. Vì vậy, bộ đánh giá phải suy luận được qua quan hệ giữa nhiều bước, đồng thời phân biệt lỗi trong hành vi agent với lỗi ở công cụ hoặc hệ thống.

## 1.3. Nghiên cứu nguồn

- [Deshpande et al. (2025), *TRAIL: Trace Reasoning and Agentic Issue Localization*](<../raw/TRAIL Trace Reasoning and Agentic Issue Localization.md>)

# 2. Thiết kế dữ liệu TRAIL

## 2.1. Đặc trưng cơ bản

Mỗi mẫu của TRAIL là một **execution trace có cấu trúc** gồm các span LLM và tool, đi kèm nhãn lỗi ở cấp span và các điểm đánh giá toàn trace. Dataset có hai split: GAIA (tìm kiếm thông tin mở) và SWE-Bench-Lite (sửa lỗi phần mềm). Có hai tổ hợp sinh dữ liệu tương ứng:

| Split          | Workflow tạo trace                                | Model nền                             | Cấu hình/tín hiệu đáng chú ý                                                        |
| -------------- | ------------------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------- |
| GAIA           | OpenDeepResearch phân cấp: manager + search agent | `o3-mini-2025-01-31` cho cả hai agent | Workflow multi-agent, dùng các công cụ tìm kiếm/duyệt nội dung.                     |
| SWE-Bench-Lite | CodeAct agent đơn tác tử                          | `claude-3-7-sonnet-20250219`          | Sandbox, Python interpreter, `gitingest`; prompt có giới hạn đầu ra và ép khám phá. |

Bài mô tả TRAIL gồm **148 trace** và **841 lỗi được gán nhãn**. Tuy nhiên, Table 5 lại ghi 118 trace GAIA và 31 trace SWE-Bench (tổng 149), với 579 và 256 lỗi tương ứng (tổng 835); bảng này vẫn cùng ghi 1.987 span. Vì các số tổng hợp không nhất quán trong tài liệu nguồn, note giữ cả hai cách báo cáo; cần kiểm tra bản phát hành dataset trước khi dùng các tổng này làm số liệu thực nghiệm.

## 2.2. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

TRAIL bắt đầu từ các mẫu chỉ-văn-bản của hai benchmark có tác vụ thực tế và cần khám phá không gian tìm kiếm hay môi trường: GAIA cho tìm kiếm thông tin mở, và SWE-Bench-Lite cho định vị/sửa lỗi trong repository GitHub. Tác giả dùng hai nguồn này để trace không chỉ là dữ liệu giả lập, đồng thời bao quát cả workflow multi-agent và single-agent.

Để sinh trace GAIA, họ chạy cấu trúc phân cấp của Hugging Face OpenDeepResearch: manager giao việc tìm kiếm cho search agent; cả hai dùng `o3-mini-2025-01-31`. Với SWE-Bench-Lite, họ chạy một CodeAct agent đơn tác tử dùng `claude-3-7-sonnet-20250219`, trong sandbox có Python interpreter và `gitingest`. Riêng split này, prompt còn đặt giới hạn độ dài đầu ra và buộc agent khám phá để chủ động làm lộ lỗi planning/context handling. Toàn bộ lần chạy được ghi bằng OpenTelemetry, cụ thể là chuẩn OpenInference, và giữ ở dạng trace có cấu trúc tương thích công cụ observability ngoài thực tế.

## 2.3. Các bước sinh và gán nhãn

1. **Chạy workflow trên các tác vụ nguồn và thu trace.** Tác giả thực thi hai cấu hình agent nói trên trên GAIA và SWE-Bench-Lite, rồi thu các span LLM và tool theo OpenTelemetry/OpenInference thay vì chuyển chúng thành log văn bản đã parse.
2. **Gán nhãn tuần tự ở cấp span.** Bốn annotator có chuyên môn software engineering và log debugging xem từng span LLM/tool trong trace. Theo taxonomy TRAIL, họ ghi `span_id`, loại lỗi, bằng chứng, mô tả và mức ảnh hưởng thấp/trung bình/cao; đồng thời chấm toàn trace về reliability, security, tuân thủ chỉ dẫn và độ tối ưu của plan.
3. **Kiểm định nhãn.** Tác giả dùng một tập 63 trace riêng để đo agreement giữa annotator, rồi thực hiện bốn vòng kiểm tra độc lập với các nhà nghiên cứu ML. Việc kiểm tra vẫn giới hạn trong nội dung trace: họ không xác minh thông tin web hay thông tin bên ngoài trace, vì LLM baseline cũng không được kỳ vọng làm việc đó.

## 2.4. Mục đích và đánh đổi của thiết kế

Thiết kế dùng GAIA và SWE-Bench-Lite, cùng một cấu hình multi-agent và một cấu hình single-agent, nhằm giữ tính sát thực tế trong khi phủ cả lỗi tìm kiếm thông tin, kỹ nghệ phần mềm, planning và coordination. Việc ghi trace trực tiếp bằng OpenTelemetry/OpenInference cũng nhằm để dữ liệu dùng được với phần mềm tracing/observability thực tế, thay vì chỉ phù hợp một format benchmark riêng.

Tác giả bổ sung ràng buộc prompt cho CodeAct để làm lộ lỗi planning và quản lý ngữ cảnh một cách hữu cơ; đây là cách tăng cơ hội quan sát các loại lỗi mà split single-agent cần kiểm tra. Đổi lại, gold label cần đọc toàn bộ span và nhiều vòng kiểm định: theo tác giả, việc gán nhãn một trace mất khoảng 30–40 phút, còn kiểm tra chất lượng tốn thêm thời gian. Họ chấp nhận giới hạn không đối chiếu sự thật ngoài trace để giữ quy trình nhất quán với năng lực thông tin mà các LLM-judge được đánh giá có thể sử dụng.

# 3. Đánh giá và insight

## 3.1. Thiết lập đánh giá

Tác giả đánh giá các LLM closed- và open-weight, gồm model reasoning và không-reasoning. Mỗi model nhận raw trace JSON, phải đồng thời nhận diện loại lỗi và định vị span; họ còn so sánh thứ hạng trên TRAIL với các leaderboard long-context, và ablation mức `reasoning.effort` của o3. Kết quả được lấy trung bình qua ba lần chạy.

## 3.2. TRAIL khó chủ yếu vì suy luận trên trace dài, không chỉ vì taxonomy

Nhiều trace có đầu vào tiệm cận hoặc vượt context window của model được thử nghiệm; đầu ra gán nhãn đầy đủ cũng có horizon dài. Tác giả quan sát mọi metric đều giảm khi raw trace dài hơn. Vì vậy, đánh giá trace có cấu trúc là một bài toán long-context reasoning thực tế, chứ không thể xem như phân loại từng span độc lập.

## 3.3. Reasoning ở test time giúp cả nhận diện lẫn định vị

Ngoại trừ o1, các model reasoning vượt model không-reasoning trên cả gán category và định vị lỗi; chênh lệch rõ hơn ở tiêu chí phải làm đúng cả hai. Khi giữ nguyên o3 và hạ dần `reasoning.effort`, mọi metric đều giảm. 
## 3.4. Không phải mọi loại lỗi đều có độ khó như nhau

**Context Handling Failures**, **Tool Selection Errors** và **Task Orchestration Errors** là các nhóm khó dự đoán nhất trên hầu hết model. Chúng đòi hỏi suy luận về trạng thái, lựa chọn thay thế và quan hệ giữa các bước, thay vì chỉ đối chiếu một span với taxonomy.

Ngược lại, hallucination kiểu **Language-only** thường dễ nhận diện hơn. Các loại như **Goal Deviation** và **Poor Information Retrieval** có chênh lệch lớn giữa các model, còn **Formatting Errors** không tăng đều theo năng lực reasoning hay độ mới của model. Do đó, tác giả không xem một điểm tổng hợp là đủ để kết luận model nào phù hợp mọi nhu cầu debug: khả năng judge phụ thuộc mạnh vào loại lỗi cần phát hiện.

## 3.5. Phân bố lỗi cho thấy cần giữ cả lỗi hiếm nhưng nghiêm trọng

Trong TRAIL, lỗi tạo đầu ra — đặc biệt format và không tuân thủ chỉ dẫn — chiếm phần lớn annotation. Tuy nhiên, lỗi thực thi hệ thống như API failure ít gặp hơn vẫn có thể gây hậu quả nghiêm trọng và khó khôi phục. Đây là lý do tác giả giữ taxonomy có các nhóm lỗi đuôi dài, thay vì chỉ tối ưu benchmark cho những loại lỗi xuất hiện thường xuyên.

# 4. Limitations và Future Works

## 4.1. Limitations

1. **Phạm vi modality còn hẹp.** TRAIL và taxonomy chủ yếu xét đầu vào, đầu ra văn bản. Tác giả lưu ý sự phát triển của agent đa phương thức đòi hỏi mở rộng taxonomy cẩn thận để bao quát các lỗi mới, chẳng hạn lỗi khi dùng công cụ đa phương thức.
2. **Nhiều nhãn đuôi dài có rất ít ví dụ.** Dù các lỗi này có thể có ảnh hưởng cao, số mẫu thấp khiến việc kiểm tra LLM-judge có nhận diện đúng chúng hay không trở nên khó hơn.

## 4.2. Future Works

Tác giả đề xuất sinh dữ liệu tổng hợp cho các lỗi hiếm nhưng tác động lớn: sửa đổi có hệ thống các trace hiện có để tạo lỗi nghiêm trọng, không thể khôi phục trong context mà LLM có thể xử lý. Mục tiêu là tăng dữ liệu đánh giá cho các nhãn ít xuất hiện, thay vì coi chúng là không quan trọng vì tần suất thấp.
