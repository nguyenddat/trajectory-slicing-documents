# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Bối cảnh về nhu cầu dùng trace để nối đánh giá với cải thiện hệ agentic đã được trình bày tại [Bối cảnh chung](<../Bối cảnh chung.md>). Bài này xét **single LLM agent** có memory, reflection, planning và tool/action trên tác vụ nhiều bước. Điểm mới trong vấn đề mà tác giả nhấn mạnh là *error propagation*: một lỗi gốc sớm làm sai suy luận và hành động về sau, nên failure cuối không chỉ ra trực tiếp nơi cần sửa.

Theo tác giả, các nghiên cứu failure trước chủ yếu liệt kê loại lỗi hoặc trình bày case study; chúng chưa có cơ chế hệ thống để truy dấu về root cause, càng chưa dùng chẩn đoán đó để agent tự phục hồi. Vì vậy, taxonomy mô tả failure, như [MAST](<MAST dataset.md>) trong phạm vi MAS, chưa tự trả lời lỗi nào trong một rollout là điểm cần can thiệp để đổi kết cục; bài này đặt mục tiêu diagnosis có thể hành động được cho trajectory single-agent.

## 1.2. Formulation: critical-error localization và debugging

Với một trajectory hoàn chỉnh \(\tau=\{(s_t,a_t)\}_{t=1}^{T}\) của agent, mô tả task và kết quả đánh giá thất bại, bài toán là tìm **critical error**: lỗi sớm nhất là nguyên nhân gốc khiến task không thể hoàn thành. Output chẩn đoán không phải một agent chịu trách nhiệm, mà gồm bước, module (memory/reflection/planning/action/system), error type, evidence, root cause, ảnh hưởng lan truyền và feedback sửa lỗi; agent được re-rollout từ bước đó để thử phục hồi.

Về ngữ nghĩa, tiêu chí này gần với *decisive error* trong [Failure Attribution - Formulation](<../Failure Attribution - Formulation.md>): đều tìm lỗi sớm mà nếu được sửa thì failure có thể đổi thành success. Tuy nhiên, nguồn không nói AgentDebug kế thừa formulation Who&When, không xét MAS hay output `agent_id`. Điểm mới của formulation AgentDebug là gắn vị trí thời gian với **module + error taxonomy + feedback + re-rollout**, thay vì chỉ quy kết ai và khi nào; chi tiết đối chiếu được tách tại [Failure Attribution - Formulation](<../Failure Attribution - Formulation.md>). Bài không cho định nghĩa toán học riêng cho \(C_{crit}\), chỉ nêu tiêu chí này trong thuật toán và prompt.

## 1.3. Các artefact đề xuất

- [AgentErrorTaxonomy](<../failure taxonomy/AgentError Taxonomy.md>): vocabulary gồm năm module lỗi.
- **AgentErrorBench:** benchmark trace lỗi có nhãn theo taxonomy và root cause tối thiểu.
- [AgentDebug](<../failure attribution methods/AgentDebug.md>): chẩn đoán lỗi critical rồi tạo feedback để re-rollout.

## 1.4. Nghiên cứu nguồn

- [Zhu et al. (2025), *Where LLM Agents Fail and How They can Learn From Failures*](<../raw/Where LLM Agents Fail and How They can Learn From Failures.md>)

# 2. Thiết kế dữ liệu AgentErrorBench

## 2.1. Đặc trưng cơ bản

Mỗi mẫu là một **failed trajectory** của single LLM agent trên ALFWorld, WebShop hoặc GAIA, kèm nhãn lỗi ở cấp decision step/module, tập root-cause tối thiểu giải thích chuỗi lỗi sau đó, và feedback có thể hành động. Dataset có 200 trajectory: 100 ALFWorld, 50 WebShop và 50 GAIA. Đây là benchmark mới do bài đề xuất, không phải chỉ là một split của benchmark nguồn.

## 2.2. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

Tác giả bắt đầu bằng hơn 500 failed trajectory từ ALFWorld, WebShop và GAIA để các chuyên gia tìm các mẫu lỗi lặp lại và xây [AgentErrorTaxonomy](<../failure taxonomy/AgentError Taxonomy.md>). Sau đó họ curate 200 trajectory đại diện từ ba benchmark trên để tạo AgentErrorBench.

Tài liệu nguồn không nêu model nền, prompt, số lần chạy, hay tiêu chí chọn 200 trajectory trong phần mô tả dataset; vì vậy không thể suy ra các cấu hình agent đã sinh trace hoặc mức độ đại diện ngoài phân bố 100/50/50. Đây là khác với phần downstream evaluation, nơi bài mới nêu backbone và số lần re-rollout.

## 2.3. Các bước sinh và gán nhãn

1. **Khám phá taxonomy.** Chuyên gia phân tích thủ công hơn 500 failed trajectory để rút ra các failure pattern theo năm module.
2. **Chọn benchmark và chuẩn bị hướng dẫn.** Tác giả curate 200 trajectory; guideline về root cause, ví dụ hiệu chuẩn và quy tắc phân biệt nhãn được chỉnh qua ba vòng pilot annotation.
3. **Gán nhãn ở cấp decision step.** Mười graduate annotator có kinh nghiệm NLP/LLM agent xem từng action, reflection và plan; họ gán error type theo taxonomy, đồng thời tìm **tập nhỏ nhất** các root-cause failure giải thích error cascade, thay vì đánh dấu toàn bộ lỗi bề mặt. Họ cũng ghi feedback để sửa.
4. **Huấn luyện, gán nhãn kép và phân xử.** Annotator được tác giả phản hồi ở giai đoạn training; một subset chung được double-annotate độc lập, bất đồng được thảo luận tập thể và dùng để làm rõ ranh giới nhãn. Agreement cuối đo bằng Cohen’s \(\kappa=0.55\) giữa các module.

## 2.4. Mục đích và đánh đổi của thiết kế

Việc dùng ba môi trường khác nhau nhằm kiểm tra diagnosis trên embodied interaction, web shopping có ràng buộc và reasoning/tool use mở, thay vì taxonomy chỉ bám một agent framework. Gán nhãn người ở cấp bước và yêu cầu root-cause tối thiểu giúp benchmark phục vụ debug: tránh xem mọi hậu quả trong cascade là những điểm sửa độc lập.

Đổi lại, nhãn nhân quả cần guideline, pilot, double annotation và phân xử nên khó mở rộng. Tác giả chọn 200 trace được curate thay vì tự động sinh quy mô lớn; Appendix A.1 cũng nêu chi phí human annotation là trở ngại để thu đủ dữ liệu huấn luyện một debugging model chuyên biệt. Hơn nữa, khả năng tái tạo tập trace còn bị hạn chế bởi các cấu hình rollout không được mô tả trong bài.

# 3. Đánh giá và insight

## 3.1. Thiết lập đánh giá

Để đánh giá localization, tác giả chạy AgentDebug và các baseline Direct Prompting, Brute Force, Binary Search trên 200 trajectory của AgentErrorBench; metric lần lượt yêu cầu đúng step, đúng step+module, và đúng cả step+module+error type. Trong thiết lập này, model detector là GPT-4.1 với temperature 0.

Để đo recovery, họ chạy AgentDebug trên ALFWorld, GAIA và WebShop với các backbone GPT-4o-mini, Qwen3-8B và Qwen3-Next-80B; tối đa năm re-rollout bắt đầu tại critical step. Baseline gồm Self-Refine, Vanilla Debugger, Tree-of-Thought và Best-of-N, được ghép ngân sách token với AgentDebug.

## 3.2. Localization có cấu trúc cần nhiều hơn chọn một bước lỗi

Theo Table 1, AgentDebug vượt các baseline ở cả step, step+module và all-correct; lợi thế rõ nhất ở yêu cầu đúng đồng thời vị trí, module và error type. Điều này phù hợp với lập luận của tác giả rằng profile lỗi theo taxonomy giúp detector phân biệt root cause với lỗi bề mặt ở các bước sau, thay vì chỉ prompt LLM chọn một vị trí trong log.

Tuy nhiên, số liệu tóm tắt trong nguồn không nhất quán: Table 1 báo trung bình Step/Step+Module/All Correct là 45.0%/31.3%/24.3%, còn đoạn *Findings* sau thí nghiệm debugging ghi 50.0% Step và 42.5% All Correct. Note giữ cả hai cách báo cáo thay vì chọn một con số làm kết quả chuẩn.

## 3.3. Sửa root cause có ích hơn self-refinement hay mở rộng rollout không định hướng

Tác giả quan sát AgentDebug tăng task success trên cả ba benchmark, có lúc đến 26% tương đối, và vượt Self-Refine cùng Best-of-N khi chi phí được ghép. Insight được họ rút ra là feedback gắn với lỗi critical giúp dồn compute vào điểm trajectory đã lệch, còn self-revision hay tăng số rollout không biết nguyên nhân vẫn có thể sửa các triệu chứng.

## 3.4. Chất lượng detector, kiến trúc rollout và ngân sách thử đều là điều kiện của recovery

Thêm attempt tiếp tục tăng success, đặc biệt ở backbone nhỏ; nhưng ablation cho thấy detector GPT-4.1 tốt hơn rõ các model detector thay thế. Trong so sánh rollout trên ALFWorld zero-shot, thiết kế modular (memory–reflection–planning–action) đạt kết quả cao nhất trong các chiến lược thử. Vì vậy, tác giả không quy success gain chỉ cho feedback: hiệu quả còn phụ thuộc vào model dùng để chẩn đoán, cấu trúc rollout và ngân sách re-rollout.

## 3.5. Error propagation ưu tiên can thiệp sớm vào memory và reflection

Phân tích cascade của tác giả cho thấy lỗi memory và reflection thường xuất hiện sớm/trung tuyến và làm sai kế hoạch ở nhiều bước tiếp theo; planning lỗi cũng có thể lặp lại khi agent cố thực thi một chiến lược vi phạm ràng buộc hay bất khả thi. Action error đôi khi dễ thấy và có thể phục hồi hơn, còn system error như tool crash hay step limit thường là điểm kết thúc ngay. Từ đó, tác giả khuyến nghị phát hiện/sửa sớm và tăng cường memory retrieval, theo dõi tiến độ hoặc verification prompt.

# 4. Limitations và Future Works

## 4.1. Limitations

1. **Phủ benchmark còn hẹp.** Tác giả nêu AgentErrorBench chỉ gồm ba benchmark, còn nhỏ về quy mô và đa dạng domain; nó chưa kiểm tra môi trường đa phương thức, tác vụ horizon dài hay ứng dụng safety-critical như y tế và tài chính.
2. **Khó có dữ liệu để huấn luyện debugger chuyên biệt.** Human annotation quy mô lớn bị xem là quá đắt đối với môi trường học thuật ít nguồn lực. Bài dùng prompt engineering với LLM có sẵn để giảm chi phí, nhưng tác giả thừa nhận cách này có thể kém hiệu năng một model debugging chuyên biệt được huấn luyện đầy đủ.

## 4.2. Future Works

Bài không có section *Future Work* riêng. Hướng tương lai tác giả trực tiếp nêu ở Appendix A.1 là mở rộng AgentErrorBench sang môi trường đa phương thức, task dài hơn và domain safety-critical. Dù Appendix cũng đặt vấn đề dữ liệu huấn luyện cho debugger chuyên biệt, nguồn không đề xuất một roadmap kỹ thuật cụ thể để thu thập hay huấn luyện model đó; vì vậy không diễn giải thêm thành future work của tác giả.
