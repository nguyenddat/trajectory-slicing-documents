# 1. Các formulation failure attribution

## 1.1. Who&When’s formulation: causal agent–step attribution

[Who&When](<raw/Which Agent Causes Task Failures and When On Automated Failure Attribution of LLM Multi-Agent Systems.md>) phát biểu automated failure attribution cho một **failed trajectory** của MAS theo lượt: tại mỗi thời điểm chỉ một trong \(N\) agent hành động. Input là trajectory thất bại cùng thông tin hệ cần để hiểu hành động; output là \((i^*,t^*)\), gồm *failure-responsible agent* và *decisive error step*.

Action của agent \(i\) tại \(t\) là decisive nếu thay nó bằng một action đúng, giữ nguyên prefix và điều chỉnh các action sau, làm outcome chuyển từ failure sang success. Nếu có nhiều candidate, target là decisive error sớm nhất. Do đó, formulation không liệt kê mọi lỗi quan sát được mà chọn một điểm can thiệp có ý nghĩa nhân quả: **ai** cần được cải thiện và **khi nào** hành động của họ làm failure có thể tránh được.

[AgenTracer](<raw/AgenTracer Who Is Inducing Failure in the LLM Agentic Systems.md>) **kế thừa nguyên vẹn** formulation này: decisive error và cặp \((i^*,t^*)\) không đổi. Đóng góp của bài là operationalize criterion bằng oracle rectification/replay và fault injection để sinh nhãn ở quy mô lớn; đây là thay đổi ở cách tạo dữ liệu, không phải một formulation mới.

## 1.2. AEGIS’s formulation: multi-label agent–error-mode attribution

[AEGIS](<raw/Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>) cũng lấy failed MAS trajectory \(\tau\) làm input, nhưng không hỏi bước nào quyết định failure. Bài đặt taxonomy \(Y=\{y_1,\ldots,y_M\}\) và yêu cầu model dự đoán attribution map \(G(\tau)\): **toàn bộ** các cặp `(agent, error_modes)` đúng; một agent có thể mang nhiều mode, một trajectory có thể có nhiều agent lỗi.

Vì vậy, output trả lời “**ai** có lỗi và **lỗi thuộc mode nào**”, không trả lời “lỗi quyết định xảy ra **khi nào**”. Formulation cũng không đòi một action, khi được sửa, phải đảo outcome. Nó dùng multi-label taxonomy làm target diagnosis, thay vì causal agent–step target của Who&When.

## 1.3. AgentDebug’s formulation: critical-error localization để phục hồi single agent

[AgentDebug](<raw/Where LLM Agents Fail and How They can Learn From Failures.md>) xét một **single LLM agent** với failed trajectory \(\tau=\{(s_t,a_t)\}_{t=1}^{T}\). Output gồm critical step sớm nhất, module (memory, reflection, planning, action hoặc system), error type, evidence, root cause, cascading effects và correction guidance; agent sau đó re-rollout từ vị trí này.

Critical error là root cause sớm nhất đưa task vào đường không thể phục hồi, sao cho sửa nó *có thể* làm trajectory đi tới success. Ý nghĩa của criterion gần decisive error của Who&When—đều tìm một lỗi sớm có giá trị can thiệp—nhưng AgentDebug không có `agent_id` và không phát biểu kế thừa Who&When. Nguồn còn mô tả không hoàn toàn thống nhất cách kiểm chứng: Section 3.2 nói test counterfactual rollout, trong khi Algorithm 1/prompt cho phép LLM suy luận toàn trace không rollout. Vì thế `critical error` không thể luôn được xem là nhãn đã oracle-verified phản thực.

## 1.4. AgentRx’s formulation: first-unrecoverable-failure attribution

[AgentRx](<raw/AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>) nhận toolset/schema, policy miền (nếu có), task instruction và failed trajectory chuẩn hóa theo step. Output formulation là \((\hat{s},\hat{y})\): **critical step** và **failure category** trong taxonomy chín loại; agent id có thể hiện diện trong record dataset nhưng không thuộc output đích.

Critical failure là failure đầu tiên mà không agent nào trong trajectory phục hồi được. Khi gán nhãn, tác giả đánh dấu mọi failure rồi lần ngược từ outcome sai để tìm failure sớm nhất chưa được recovery; khi suy luận, judge chọn violation sớm nhất đủ giải thích failure và có thể bỏ qua evidence false positive bằng context toàn trace. Như vậy, AgentRx không chọn lỗi xuất hiện đầu tiên, mà chọn lỗi đầu tiên **vẫn còn hiệu lực sau cơ chế recovery**. Bài không formalize tập continuation hoặc thủ tục counterfactual kiểm chứng như Who&When/TraceElephant.

## 1.5. TraceElephant’s formulation: role-aware failure inevitability

[TraceElephant](<raw/Seeing the Whole Elephant A Benchmark for Failure Attribution in LLM-based Multi-Agent Systems.md>) mở rộng đơn vị quy kết từ agent sang **functional component**: agent tường minh hoặc module planning, orchestration, tool use trong scaffold. Mỗi step có input, output, component id, step id, metadata task, cấu hình và trạng thái tool/environment; vì vậy input không chỉ là output log.

Với failed trace \(T\), failure step là \(t^*\) sớm nhất sao cho **mọi continuation hợp lệ** từ prefix đến \(t^*\) đều thất bại; component hành động ở \(t^*\) là responsible component. Output vẫn là component–step như Who&When, nhưng criterion đã đổi từ *earliest correctable mistake* thành *earliest failure-inevitable step*. Một hallucination upstream chưa là failure step nếu verifier được giao kiểm tra vẫn có thể sửa; nếu verifier bỏ lỡ, chính verifier mới là component được quy kết. Full observability là điều kiện để biết component nào đã có context và nghĩa vụ recovery, chứ không chỉ là thêm evidence cho cùng nhãn cũ.

# 2. Điểm chung và riêng của các formulation

## 2.1. Điểm chung: đều biến outcome failure thành diagnosis có thể hành động

Điểm chung nền tảng là cả năm formulation đều bắt đầu từ **trajectory thất bại**, không dừng ở score hay final answer, và trả về thông tin có thể định hướng cải thiện. Đây không phải ngẫu nhiên: [Who&When](<raw/Which Agent Causes Task Failures and When On Automated Failure Attribution of LLM Multi-Agent Systems.md>) phê bình benchmark/LLM-as-a-judge vẫn để việc nối failure với component cần sửa cho con người; [AEGIS](<raw/Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>) nêu nút thắt dữ liệu để tự động hóa chẩn đoán đó; [AgentDebug](<raw/Where LLM Agents Fail and How They can Learn From Failures.md>) xuất phát từ error cascade làm final failure không chỉ ra root cause; [AgentRx](<raw/AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>) và [TraceElephant](<raw/Seeing the Whole Elephant A Benchmark for Failure Attribution in LLM-based Multi-Agent Systems.md>) đều cho rằng lỗi hiển thị trong log chưa chắc là điểm cần can thiệp.

Vì vậy, ý đồ chung là trả lời một phiên bản của câu hỏi: *sau failed run, cần thay đổi ở đâu để hệ tốt hơn?* Tuy nhiên, đây chỉ là **mục tiêu chẩn đoán chung**, không phải một shared gold target. Không phải formulation nào cũng tìm causal root duy nhất, cũng không phải formulation nào cũng cần agent id, bước thời gian hay taxonomy.

## 2.2. Điểm riêng: mỗi formulation chọn một nghĩa khác nhau của “lỗi cần sửa”

| Formulation | Đầu ra trọng tâm | Tiêu chí quyết định | Ý đồ của nghiên cứu gốc |
| --- | --- | --- | --- |
| Who&When | Một agent và một step | Sửa action ở step đó, điều chỉnh downstream và outcome có thể đổi; chọn decisive step sớm nhất | Biến evaluation failure thành **causal responsibility** của thành phần MAS cần cải thiện. |
| AEGIS | Mọi cặp agent–error mode | Khớp attribution map theo taxonomy; không có criterion thời gian/phản thực | Giữ nhiều lỗi để tạo failure profile và học diagnosis agent–type ở quy mô lớn, thay vì ép trajectory về một root duy nhất. |
| AgentDebug | Critical step + module/type/evidence/feedback | Root cause sớm nhất mà sửa có thể giúp task success | Phục vụ **debug và re-rollout** cho single agent; cần cả vị trí lẫn nội dung feedback, không chỉ attribution. |
| AgentRx | Critical step + failure category | Failure đầu tiên không được bất kỳ agent nào recovery | Loại các lỗi đã được hệ hấp thụ/sửa, để feedback nhắm đúng failure thực sự chặn task completion. |
| TraceElephant | Responsible component + step | Prefix sớm nhất mà mọi continuation hợp lệ đều failure | Quy trách nhiệm theo **role và nghĩa vụ recovery**, không quy đơn giản cho mistake upstream đầu tiên. |

Ba khác biệt chi phối việc đọc kết quả. Thứ nhất, **đơn vị quy kết** thay đổi: Who&When/TraceElephant quy cho agent hoặc component; AEGIS quy cho tập agent–mode; AgentDebug/AgentRx chủ yếu quy cho step để can thiệp. Thứ hai, **mức đa lỗi** thay đổi: AEGIS cố ý giữ tất cả labels, còn Who&When, AgentDebug, AgentRx và TraceElephant chọn một điểm chính vì chúng phục vụ một hành động sửa cụ thể. Thứ ba, **ngữ nghĩa nhân quả/recovery** không đồng nhất: Who&When dùng counterfactual correctability, AgentRx dùng unrecovered failure quan sát được, TraceElephant dùng inevitability trên mọi continuation hợp lệ; AgentDebug chỉ gần counterfactual về ý nghĩa và không luôn kiểm chứng bằng rollout.

## 2.3. Ý nghĩa theo thời gian: kế thừa không đồng nghĩa với phát triển formulation

Theo timeline, Who&When là mốc formulation causal agent–step. AgenTracer kế thừa mốc này nguyên vẹn và phát triển **cách sinh nhãn**, nên không nên đưa nó vào bảng các formulation cạnh tranh. AEGIS và AgentDebug là hai phân nhánh 2025 theo hai mục tiêu khác: multi-label diagnosis cho MAS và debugging/recovery cho single agent. Đầu 2026, AgentRx đặt recovery thành tiêu chí chọn critical failure; sau đó TraceElephant là refinement gần nhất của formulation agent–step, vì giữ dạng output nhưng thay đổi substantive nghĩa của responsibility bằng role-aware inevitability.

Do đó, không thể đọc accuracy giữa các benchmark như một ranking chung về “failure attribution tốt hơn”. So sánh trực tiếp chỉ hợp lý khi gold target giống nhau—điển hình là Who&When và AgenTracer. Với các formulation khác, câu hỏi cần đặt trước là: cần biết **ai chịu trách nhiệm**, **lỗi thuộc loại gì**, **điểm nào nên re-rollout**, hay **khi nào failure không còn có thể tránh được**. Cùng một trace có thể nhận nhiều nhãn khác nhau mà không mâu thuẫn, vì các formulation trả lời các câu hỏi can thiệp khác nhau.
