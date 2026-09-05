# 1. Bối cảnh nghiên cứu
Việc tái định hình các Large Language Models — LLMs thành các **agent** ([[References#^patil2023gorilla|patil2023gorilla]], [[References#^memgpt|memgpt]], [[References#^wang2024survey|wang2024survey]]) + các **hệ đa tác tử mang tính agentic** (_agentic multi-agent systems_) đã nhận được sự quan tâm đáng kể ([[References#^metagpt|metagpt]]; [[References#^li2023camel|li2023camel]]; [[References#^wu2023autogen|wu2023autogen]]).

Những hệ agentic cho thấy tiềm năng đáng kể trong nhiều lĩnh vực khác nhau: 
- Lập trình ([[References#^metagpt|metagpt]], [[References#^chatdev|chatdev]])
- Khám phá khoa học ([[References#^ghafarollahi2409sciagents|ghafarollahi2409sciagents]], [[References#^gottweis2025towards|gottweis2025towards]])
- Giải quyết các bài toán thực tế phức tạp ([[References#^fourney2024magentic|fourney2024magentic]], [[References#^openhands|openhands]]).

Mặc dù MAS ngày càng được sử dụng rộng rãi, mức cải thiện hiệu năng của chúng thường vẫn khá hạn chế so với các framework single-agent [[References#^agentless|agentless]], hoặc thậm chí so với những baseline đơn giản như **best-of-N sampling** [[References#^kapoor2024ai|kapoor2024ai]].

Sau khi được xây dựng, các hệ thống này thường được cải tiến thông qua một quy trình lặp khi chúng thất bại trong những tình huống cụ thể: 
- **Đánh giá dựa trên các benchmark đã được thiết lập, tiếp theo là thực hiện failure attribution thủ công và sau đó tinh chỉnh hệ thống**. Chu trình này được lặp lại cho đến khi đạt được kết quả mong muốn. 
- **Failure attribution** — tức xác định những thành phần của hệ thống trực tiếp dẫn đến thất bại của tác vụ — là một bước quan trọng, đóng vai trò nền tảng trong việc định hướng các cải tiến tiếp theo.

Mặc dù quan trọng, quá trình này phần lớn vẫn bị bỏ qua vì:
- Đòi hỏi lượng lớn công sức, chẳng hạn như phân tích các log lịch sử phức tạp và xử lý những chi tiết kỹ thuật phức tạp của hệ thống. 
- Việc ánh xạ kết quả đánh giá từ benchmark tới các thành phần gây ra lỗi phụ thuộc rất nhiều vào kiến thức chuyên môn theo miền, 

=> Đặt ra thêm những yêu cầu đối với người thực hành. Khi hệ thống ngày càng trở nên phức tạp, thách thức này cũng ngày càng lớn do số lượng thành phần cần được xem xét trong quá trình failure attribution ngày càng tăng.

Các nỗ lực thủ công trước đây đã sử dụng một cách tiếp cận không trực tiếp để hỗ trợ failure attribution trong hệ đa tác tử: Xây dựng các benchmark ngày càng **fine-grained**.
- Kỳ vọng rằng nhiều metric hơn sẽ giúp việc failure attribution diễn ra nhanh hơn ([[References#^zhuge2024agent|zhuge2024agent]]). 
- **DevAI** ([[References#^zhuge2024agent|zhuge2024agent]]): benchmark về lập trình trong đó tích hợp nhiều yêu cầu đầu ra khác nhau, qua đó cung cấp một cách đánh giá chi tiết hơn so với benchmark **SWE-Bench** được sử dụng rộng rãi ([[References#^swe-bench|swe-bench]]), vốn chỉ tập trung vào tỷ lệ giải quyết cuối cùng (_final resolution rates_).

Tuy nhiên, bất chấp những tiến bộ này, quá trình failure attribution dựa trên kết quả benchmark vẫn là một quy trình thủ công; các benchmark này chỉ cung cấp thêm nhiều metric để sử dụng như các điểm tham chiếu mà không giải quyết một cách căn bản những thách thức nền tảng của bài toán. Khi benchmark ngày càng toàn diện hơn, một câu hỏi nền tảng vẫn chưa được trả lời:

Tuân theo nguyên tắc rằng: “evaluation is not an end in itself, but a means to improvement.” ([[References#^scriven1981evaluation|scriven1981evaluation]]) tức là, **đánh giá không phải là mục đích tự thân, mà là một phương tiện để cải thiện hệ thống**.

Tuy khái niệm Automated Failure Attribution xuất hiện sau, bài toán này được [[References#^mast|mast]] tiếp cận trước tiên nhưng thuộc thể loại: Error Categorization. Nó xây dựng một taxonomy các failure modes để hiểu **“Why do MAS fail?”**. Ngay sau đó, 

Nghiên cứu [[References#^who&when|who&when]] đề xuất và hình thức hóa một bài toán nghiên cứu mới:
**automated failure attribution trong các hệ đa tác tử LLM.** với mong muốn thay thế failure attribution thủ công, qua đó cho phép nguồn lực con người tập trung vào việc cải thiện chức năng của hệ thống thay vì phải thực hiện các quá trình chẩn đoán tốn nhiều thời gian.

[[References#^who&when|who&when]] cho rằng dự đoán agent đã khó, nhưng xác định **decisive error step** còn khó hơn đáng kể: Việc chuyển từ _categorization_ sang _localization_ khó hơn rất nhiều.

# 2. Motivation

## 2.1. **Trajectory dài = khó.** 
**[[Who&When dataset|Who&When]]** chia failure log thành 5 mức từ **5–17 steps tới 93–130 steps** và thấy cả agent-level lẫn step-level accuracy đều giảm khi context dài lên; **step-level accuracy nhạy hơn**, và ở nhóm dài nhất cả ba phương pháp gần như về 0 ([[References#^who&when|who&when]]):
- Phương pháp **All-at-Once** có receptive field toàn cục nên tốt cho _who_, nhưng chính paper nói model gặp “space-in-the-needle problem”: có toàn bộ history nhưng khó pinpoint đúng decisive step trong log dài.
- **Step-by-Step** local hơn nên tốt hơn cho exact step localization, nhưng đổi lại chi phí rất cao. Trong thí nghiệm hand-crafted systems, token cost là khoảng **17,106** cho All-at-Once và **87,720** cho Step-by-Step; hybrid lên **149,177**.

**[[Trail dataset|TRAIL]]** có bằng chứng trực tiếp nhất: paper báo cáo rằng **tất cả performance metrics đều anti-correlated với input length**, và kết luận raw trace dài hơn làm TRAIL khó hơn đối với model. Đồng thời nhiều trace của TRAIL chiếm phần đáng kể hoặc vượt context limit hiện tại ([[References#^trail|trail]]).

**[[AgentRx|AgentRx]]** cũng xác nhận đúng cơ chế này từ một hướng khác. τ\tau-bench trung bình khoảng **4,889 tokens/trajectory**, trong khi Magentic khoảng **16,484 tokens/trajectory**. Paper nói rõ ở Magentic, one-shot generation dễ bị **context dilution**, còn localized step-by-step constraint generation đáng tin cậy hơn ([[References#^agentrx|agentrx]]).

## 2.2. Phân bố lỗi và kiến trúc MAS có tương quan

[[MAST dataset|MAST]] cho thấy failure distributions khác đáng kể giữa các MAS và thường phản ánh architectural characteristics/design philosophies
- AppWorld, OpenManus và HyperAgent có failure profile khác nhau; 
- MetaGPT với ChatDev trên cùng ProgramDev và thấy tỷ lệ các nhóm failure thay đổi đáng kể.
=> Đồng thời tác giả cẩn thận nói rằng khi system/task configurations khác nhau, các profile này nên được xem như **system-specific failure profiles**, không phải comparison nhân quả hay ranking trực tiếp giữa MAS ([[References#^mast|mast]]).

**[[TraceElephant dataset]]** kết luận rõ rằng cần **architecture-aware attribution** ([[References#^trace_elephant|trace_elephant]]):
- failure-responsible agent types khác nhau theo system;
- decisive error steps có distribution khác nhau;
- attribution accuracy phụ thuộc vào **agent type và step position**;

## 2.3. Câu hỏi đặt ra
- RQ1: Under a training-free and non-LLM setting, can trajectory segmentation improve existing failure-attribution baselines on long or difficult execution trajectories, and under which trajectory and segmentation conditions does it provide the greatest benefit?
- RQ2: To what extent does MAS architecture have a causal effect on the distribution and localization of failures, beyond correlations induced by task characteristics, backbone models, and other execution settings?