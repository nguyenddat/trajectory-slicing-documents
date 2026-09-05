# 1. Vấn đề đặt ra

## 1.1. Khoảng trống developer-facing về khả năng quan sát

Bài báo cùng nằm trong bối cảnh failure attribution của hệ LLM multi-agent và nhu cầu nối kết quả đánh giá với thành phần phải sửa, đã được trình bày tại [Bối cảnh chung](<Bối cảnh chung.md>). Khoảng trống riêng mà tác giả nhắm tới là **khả năng quan sát phù hợp với người phát triển**: người debug thường có task instruction, prompt, thông điệp trung gian, lời gọi tool và trạng thái môi trường; trong khi benchmark [Who&When](<Who&When dataset.md>) chỉ công bố output của agent trong trace, không có input/context dẫn tới output đó.

Theo phân tích của tác giả trên 184 failure case của Who&When, ít nhất 21% trường hợp không thể quy kết đáng tin cậy chỉ từ output log. Vì vậy, vấn đề không đơn thuần là cần thêm một kỹ thuật dự đoán từ log black-box, mà là cần một benchmark cho kịch bản developer-facing, nơi nguyên nhân có thể nằm trong thông tin bị truyền mất, prompt theo role, cấu hình agent, hoặc tương tác tool/môi trường.

## 1.2. Bài toán và formulation

TraceElephant xét một hệ gồm các **functional component** hoạt động theo lượt; component có thể là agent tường minh trong MAS, hoặc module planning, orchestration và tool use trong scaffold single-agent. Với một failed trace đầy đủ, đầu ra cần tìm vẫn là component chịu trách nhiệm và bước failure quyết định.

Điểm mới của decisive error, gồm tiêu chí earliest failure-inevitable step và nguyên tắc role-aware/recoverability-aware, là khái niệm dùng chung để đối chiếu với Who&When. Phần formalization và ví dụ verifier được tách tại [Failure Attribution - Formulation](<Failure Attribution - Formulation.md#4-failure-attribution-duoi-full-execution-observability-trong-traceelephant>), thay vì lặp lại trong note dataset.

## 1.3. Nghiên cứu nguồn

- [Chen et al., *Seeing the Whole Elephant: A Benchmark for Failure Attribution in LLM-based Multi-Agent Systems*](<Seeing the Whole Elephant A Benchmark for Failure Attribution in LLM-based Multi-Agent Systems.md>)

# 2. Dataset TraceElephant

## 2.1. Đặc trưng cơ bản

Mỗi instance benchmark là một **execution trace thất bại đầy đủ** kèm code/cấu hình thực thi lại được của hệ đã sinh trace, nhãn component chịu trách nhiệm và decisive failure step. Sau cleaning, tác giả thu 380 lần chạy, trong đó **220 run thất bại** được đưa vào benchmark:

| Hệ thống | Nguồn task | Tổng trace | Trace thất bại |
| --- | --- | ---: | ---: |
| Captain-Agent | GAIA | 126 | 73 |
| Captain-Agent | AssistantBench | 21 | 12 |
| Magentic-One | GAIA | 119 | 74 |
| Magentic-One | AssistantBench | 30 | 17 |
| SWE-Agent | SWE-Bench Verified | 84 | 44 |

Trace được lưu JSON. Metadata cấp trace ghi task id/instruction, tên hệ, agent configuration (roster, prompt, tool) và system architecture (tài liệu thiết kế/code). Mỗi step có id, agent id/name, **input context** đầy đủ; output gồm response và, nếu có, raw tool log (tool, arguments, result, status). Vì vậy unit dữ liệu giữ cả điều component đã thấy khi ra quyết định, chứ không chỉ thứ tự output.

## 2.2. Nguồn dữ liệu mẫu và phương pháp sinh trace

Tác giả dùng nguyên implementation, định nghĩa agent, prompt và tool interface chính thức của ba hệ có kiến trúc khác nhau: Captain-Agent tạo/lắp đội agent động; Magentic-One có orchestrator và các agent vai cố định; SWE-Agent là scaffold một agent thiên về tool cho software engineering. Captain-Agent và Magentic-One chạy các task GAIA, AssistantBench; SWE-Agent chạy SWE-Bench Verified. Mỗi system–task pair dùng cấu hình cố định, temperature 0,3, và được lặp nhiều trial để giữ hành vi tool tương đối ổn định nhưng vẫn có đa dạng do decoding.

Việc thu trace dùng LLM API middleware như proxy: chặn request của MAS tới LLM, lưu payload/message và decoding parameter, chuyển tiếp tới backend rồi lưu response; khi quan sát được, middleware cũng lưu tool call do output LLM dẫn tới. Vì proxy này không sửa agent logic, prompt hay control flow, tác giả nhằm thu input/output đầy đủ mà vẫn giữ execution gốc.

## 2.3. Các bước sinh và gán nhãn

1. **Chạy hệ gốc trên task nguồn.** Tác giả chạy ba MAS theo cấu hình đã định và giữ phân bố failure phát sinh tự nhiên, thay vì ép mỗi nguồn có cùng failure rate.
2. **Ghi trace qua middleware.** Proxy thu toàn bộ LLM request/response và tương tác tool quan sát được, rồi mỗi run nhận id riêng.
3. **Chuẩn hóa tối thiểu.** Tác giả chỉ dùng regex để trích agent name, phân loại output là LLM thuần hay tool-mediated, rồi gán step id theo thứ tự thời gian và agent id theo lần xuất hiện đầu; nội dung input/output vẫn được giữ raw.
4. **Chọn instance failure.** Sau cleaning, chỉ các run có outcome failure trở thành mẫu attribution; executable system và configuration tương ứng được giữ để replay/inspect state.
5. **Gán nhãn độc lập và ghi độ chắc chắn.** Ba annotator có ít nhất một năm kinh nghiệm phát triển/debug MAS xác định responsible component và earliest inevitable step, đồng thời đánh dấu confident/uncertain. Bài báo báo cáo Krippendorff's alpha ở vòng đầu là 0,72 cho agent và 0,64 cho step.
6. **Đồng thuận và kiểm tra chéo.** Case bị ít nhất một người đánh dấu uncertain được xem chung, thống nhất bằng đồng thuận thay vì majority vote. Sau đó mỗi người rà một phần nhãn của người khác; case có bất nhất được thảo luận/gán lại đến khi nhất quán.

Mô tả nguồn có một điểm chưa rõ: Appendix nói trace được chia đều cho ba annotator ở vòng đầu, nhưng đồng thời báo cáo alpha cho “independent annotations”. Bài không nói rõ mọi trace có bao nhiêu nhãn độc lập trước khi tính alpha; note vì vậy không suy ra một cơ chế overlap cụ thể hơn.

## 2.4. Mục đích và đánh đổi của thiết kế

Thiết kế full trace + môi trường replay nhằm biến benchmark từ transcript output-only thành đối tượng debug: phương pháp có thể phân biệt input upstream bị mất/biến đổi với lỗi reasoning của component đang hành động, đồng thời kiểm tra giả thuyết bằng re-execution. Giữ nguyên hệ gốc và preprocessing tối thiểu là đánh đổi ưu tiên fidelity, khả năng so sánh giữa framework, thay vì ép mọi MAS vào một schema semantic đã diễn giải.

Full observability cũng có rủi ro lộ prompt, intermediate artifact, tool detail hoặc dữ liệu riêng tư. Tác giả chỉ chạy task benchmark công khai trong môi trường kiểm soát, dùng browser không có profile/cookie cá nhân, tự động lọc path/identifier rồi manual review trước khi release; họ nêu khả năng redaction/anonymization selective khi cần. Các biện pháp này giảm rủi ro nhưng không làm TraceElephant trở thành log của deployment thực tế.

# 3. Phương pháp failure attribution được đánh giá

## 3.1. Không đề xuất một attribution model độc lập mới

Đóng góp trung tâm của bài là benchmark và so sánh cấu hình, không phải một model failure attribution được huấn luyện mới. Bài dùng lại ba prompt-based technique — [All-at-once](<All-at-once.md>), [Binary Search](<Binary-search.md>) và [Step-by-step](<Step-by-step.md>) — rồi xây hai cấu hình agentic để khai thác TraceElephant:

- [Static Agentic](<Static Agentic.md>) điều hướng giữa trace tổng quát và field I/O của step bằng mini-SWE-agent.
- [Dynamic Agentic](<Dynamic Agentic.md>) lấy hypothesis từ Static Agentic, replay từ candidate step và can thiệp input để kiểm tra phản thực trong một cửa sổ ngắn.

Hai cấu hình này trả lời cùng output agent/component–step của formulation. Dynamic Agentic xác nhận trách nhiệm **cục bộ**; nó không được định nghĩa là sửa toàn task hay chứng minh candidate là nguyên nhân nhân quả đủ cho failure toàn cục.

## 3.2. Thiết lập đánh giá cần thiết

Static configuration cung cấp metadata và full step I/O; Dynamic cung cấp thêm môi trường runnable. Các ablation static lần lượt bỏ metadata, input, hoặc cả hai để mô phỏng trace output-only. Mỗi phương pháp được thử với và không có task-level ground truth: đáp án chính thức của GAIA/AssistantBench, hoặc pass/fail từ test suite của SWE-Bench. Đánh giá bằng accuracy khớp nhãn agent/component và accuracy khớp decisive step; kết quả trung bình ba run độc lập.

Các phương pháp khác như ECHO, AgenTracer, GraphTracer và FAMAS không được đưa vào thực nghiệm: theo tác giả, chúng không có implementation/source runnable, và các bản tái hiện của tác giả không đạt số liệu giống bài gốc.

# 4. Đánh giá và insight

## 4.1. Full observability đặc biệt quan trọng khi định vị step

Khi bỏ cả metadata lẫn input — setting chỉ còn output tương đương Who&When — agent accuracy giảm từ 0,66 xuống 0,54 và step accuracy từ 0,30 xuống 0,17 trong Static Agentic. Tác giả diễn giải đây là giảm tương đối khoảng 22% ở agent-level nhưng 76% ở step-level: thiếu decision context làm khó nhất việc chọn đúng thời điểm quyết định, không chỉ việc biết ai có vẻ liên quan.

Theo tác giả, khác biệt trọng tâm không phải trace có chứa reasoning text hay tool description hay không, mà có lưu **chính xác context** component đã quan sát trước action: prompt role-specific, history hiển thị, context do hệ dựng, cấu hình và thông tin tool/môi trường. Chính thông tin này cho phép phân biệt lỗi upstream truyền/mất thông tin với lỗi của component hành động.

## 4.2. Replay động cải thiện kiểm tra step, còn static agentic tốt nhất trong static setting

Dynamic Agentic đạt step-level cao hơn Static Agentic khoảng 10% tương đối (0,333 so với 0,303, có ground truth), trong khi agent-level chỉ tăng nhẹ. Lý giải của tác giả là replay/counterfactual giúp loại candidate step giả, còn trách nhiệm cấp agent phần lớn đã suy ra được từ role và coordination trong trace tĩnh đầy đủ.

Trong các cấu hình static, Static Agentic thường tốt nhất vì có thể lần theo chain analysis/tool use và zoom giữa global–local fields. Nếu chỉ so ba prompting method, All-at-once tốt hơn Binary Search và Step-by-step trong thử nghiệm này. Bài giả thuyết trace của TraceElephant dài hơn làm prompt tăng dần ở Step-by-step vượt effective context window ở các lượt sau; đây là khác với [Who&When](<Who&When dataset.md>), nơi step-level tốt nhất dưới Step-by-step.

## 4.3. Kiến trúc MAS và vị trí lỗi thay đổi độ khó attribution

Tác giả thấy component làm thao tác cụ thể với môi trường ngoài — web/browsing của Captain-Agent, Magentic-One và code editing của SWE-Agent — chiếm hơn một nửa failure. Orchestrator/planner vẫn là nguồn đáng kể (18–29%), với lỗi decomposition, chọn agent hay coordination có thể lan truyền và chỉ lộ ra về sau.

Phân bố step cũng phụ thuộc kiến trúc: Captain-Agent lắp đội động nên failure rải từ agent selection tới tool call; Magentic-One và SWE-Agent thiết kế thủ công tập trung nhiều failure sớm vào planning/routing. Trong Magentic-One, attribution step đầu đặc biệt khó; tác giả giải thích các giả định hoặc phân rã ban đầu sai có thể chỉ biểu hiện sau nhiều vòng exploration/re-planning. Vì vậy, họ rút ra cần phương pháp architecture-aware thay vì coi mọi agent type và mọi vị trí trace tương đương.

## 4.4. Reference outcome và năng lực reasoning vẫn là tín hiệu hỗ trợ, không thay thế observability

Có ground truth làm mọi setting giảm khó hơn, nhất là step-level. Agentic method giảm ít hơn khi không có reference, điều tác giả cho là do replay/kiểm tra hành vi có thể bù một phần tín hiệu thiếu. Các backbone reasoning/context mạnh (Claude-4.5-Sonnet, DeepSeek-R1, GPT-4o trong thử nghiệm) nhìn chung tốt hơn model yếu hơn, nhưng kết quả vẫn thay đổi theo kiến trúc và điều kiện quan sát; benchmark không suy ra rằng model mạnh tự giải quyết attribution.

# 5. Limitations và Future Works

## 5.1. Limitations tác giả nêu

Bài có section *Limitations* riêng. Scope là developer-facing setting có full trace nên không bao phủ black-box scenario hay mọi kiến trúc hiện có/tương lai. Benchmark chỉ có ba MAS; tác giả chọn Captain-Agent, Magentic-One và SWE-Agent để phủ lần lượt đội động, orchestrator tập trung và workflow software engineering, nhưng cũng thừa nhận insight có thể không khái quát hoàn toàn khỏi ba hệ này.

## 5.2. Future works và hàm ý tác giả đề xuất

Bài không có section *Future Work* độc lập, nhưng mục implications nêu các hướng kỹ thuật rõ ràng:

1. Xây attribution **architecture-aware**, dùng prior về cấu trúc tập trung/động, workflow tool-heavy/planning-heavy để ưu tiên component và interaction dễ lỗi.
2. Nâng Static Agentic bằng vòng sinh–kiểm hypothesis hoặc reasoning trên graph tương tác đã trích, để xử lý dependency dài và ambiguity trong trace tĩnh.
3. Dùng môi trường runnable cho phân tích sâu hơn: đọc code, dựng lại control flow, khám phá state space quanh failure, fault injection, hoặc causal discovery can thiệp nhiều biến — vượt quá replay single-step hiện tại.
4. Fine-tune model attribution gọn hơn bằng trace và environment, thêm agent graph, tool-call sequence và temporal dependency làm tín hiệu học.
5. Tích hợp capture trace, visualization interaction và gợi ý failure point vào framework phát triển MAS để giảm overhead debug.
