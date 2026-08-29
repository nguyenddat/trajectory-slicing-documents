# 1. Vấn đề đặt ra

## 1.1. Khoảng trống chẩn đoán từ execution trajectory

Bài báo thuộc bối cảnh failure attribution nhằm nối một lần đánh giá thất bại với nơi cần cải thiện trong hệ agentic, đã được trình bày tại [Bối cảnh chung](<../Bối cảnh chung.md>). Khoảng trống riêng mà [AgentRx](<../raw/AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>) nêu là các execution dài, xác suất, có thể nhiều agent, tool output nhiễu và side effect: failure có thể lan truyền trước khi bị quan sát. Do đó, chỉ biết outcome cuối không chỉ ra failure nào thực sự đã chặn task.

Tác giả phê bình các benchmark agent phổ biến chủ yếu đo terminal success, không có gold label cho failure attribution. Họ cũng đối chiếu với [Who&When](<Who&When dataset.md>): một lỗi được Who&When gán nhãn có thể đã được Orchestrator khắc phục, nên chưa phải nguyên nhân của outcome sai. Vì thế, khoảng trống không chỉ là tìm failure xuất hiện sớm nhất, mà là xác định **first unrecoverable failure** -- bước sai sớm nhất mà trajectory không hồi phục được.

## 1.2. Bài toán và formulation

Với failed execution trajectory gồm message, tool call, tool output và trạng thái môi trường quan sát được, nghiên cứu yêu cầu xác định critical step và category giải thích vì sao failure đó làm task không thể hoàn thành. Những failure event khác trong cùng log vẫn được giữ để thể hiện chain lỗi, nhưng không phải mọi event đều là root cause.

Formulation của critical failure, cùng ranh giới của nó với decisive error trong Who&When và earliest inevitable step trong TraceElephant, được tách tại [Failure Attribution - Formulation](<../Failure Attribution - Formulation.md#5-critical-failure-attribution-trong-agentrx>).

## 1.3. Nghiên cứu nguồn

- [Barke et al. (2026), *AgentRx: Diagnosing AI Agent Failures from Execution Trajectories*](<../raw/AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>)

# 2. Dataset AgentRx

## 2.1. Đặc trưng cơ bản

Mỗi instance là một **failed execution trajectory** có id, toàn bộ trace quan sát được, danh sách failure event và một critical failure. Mỗi failure event ghi step, mô tả ngắn, nhãn taxonomy và rationale; trace chứa message agent/người dùng, tool invocation, tool output và trạng thái môi trường mà hệ đã log được. Taxonomy nhãn là [AgentRx Failure Taxonomy](<../failure taxonomy/AgentRx Failure Taxonomy.md>) gồm chín loại.

Benchmark có **115 trajectory thất bại**; source không mô tả split train/validation/test:

| Miền / hệ      | Loại hệ và task                           | Số failed trajectory |
| -------------- | ----------------------------------------- | -------------------: |
| \(\tau\)-bench | Single agent, retail API workflow         |                   29 |
| Flash          | Multi-agent, incident-management workflow |                   42 |
| Magentic-One   | Multi-agent, open-ended web/file task     |                   44 |

Trong đánh giá, độ dài median lần lượt là 36 step (20--62), 3 troubleshooting-guide step (2--6, mỗi step Flash có trung bình tám substep) và 33 step (5--130). Những số này cho biết benchmark bao gồm cả workflow rất ngắn và log dài, không phải số split bổ sung.

## 2.2. Nguồn dữ liệu mẫu và phương pháp thu trajectory

Tác giả chỉ lấy các lần chạy task thất bại có ít nhất một agent failure, để benchmark đo attribution chứ không đo tỉ lệ thành công.

- **\(\tau\)-bench:** lấy 115 retail task và chạy một trial/task bằng GPT-4o; 29 run thất bại được phân tích. Agent có API và policy miền để hỏi thông tin sản phẩm hoặc cancel/modify/return/exchange đơn hàng, cập nhật địa chỉ.
- **Flash:** lấy 42 trajectory có agent failure từ workflow automation chẩn đoán incident production. Team agent chuyên môn thực hiện troubleshooting guide, gồm query cluster bị ảnh hưởng và bước mitigation.
- **Magentic-One:** lấy ngẫu nhiên 44 failed trajectory từ [Who&When](<Who&When dataset.md>). Hệ có năm agent chuyên về browsing web, truy cập file, viết code và điều phối tác vụ web/file mở.

Việc lấy trace failure tự nhiên từ ba setting khác nhau nhằm để nhãn phản ánh lỗi phát sinh khi chạy hệ, thay vì inject lỗi có kiểm soát. Bài không mô tả split hay một thủ tục cân bằng các tổ hợp hệ--task; vì vậy không thể diễn giải 115 trace như các split cân bằng giữa miền hoặc loại failure.

## 2.3. Các bước sinh và gán nhãn

1. **Chọn failed run và đọc toàn bộ trajectory.** Ba annotator làm việc trên từng miền; họ xem sequence theo step thay vì chỉ terminal answer.
2. **Đánh dấu đầy đủ failure.** Ở open coding, annotator ghi mọi step không làm tiến tới outcome đúng, kể cả lỗi sau đó hệ đã khắc phục. Mỗi event có step index, open code bám sát trace và lý do đó là failure.
3. **Để taxonomy xuất hiện từ dữ liệu.** Các open code được so sánh liên tục; pattern trùng dùng lại code, pattern mới tạo code/định nghĩa mới. Nhóm định kỳ gom code thành category, thảo luận và tinh chỉnh đến theoretical saturation.
4. **Freeze taxonomy rồi closed-code/recode.** Sau khi không còn hiện tượng mới, tác giả cố định category, gán nhãn phần còn lại và gán lại phần đã đọc để kiểm tra taxonomy cũng bao phủ các case ngoài subset ban đầu. Kết quả là taxonomy chín loại dùng chung giữa miền.
5. **Tìm critical failure.** Từ terminal failure, annotator lần ngược đến failure sớm nhất mà hệ không hồi phục; event này là critical failure của instance. Record vẫn giữ các failure đã gặp và rationale để thấy diễn biến lỗi.

## 2.4. Mục đích và đánh đổi của thiết kế

Grounded theory giúp tác giả không áp sẵn label set từ một architecture/miền, rồi kiểm tra taxonomy trên cả single-agent và multi-agent. Ghi tất cả failure trước khi chọn root cause cũng chủ ý tách deviation có thể recovery khỏi error thật sự quyết định outcome -- điểm mà tác giả cho là khác với nhãn [Who&When](<Who&When dataset.md>) ở một số case.

Đánh đổi là annotation người tốn thời gian: trung bình 20 phút/trajectory \(\tau\)-bench, 22 phút Flash và 24 phút Magentic-One, tổng 42,7 giờ cho 115 trajectory. Chính chi phí này là lý do bài đề xuất framework [AgentRx](<../failure attribution methods/AgentRx.md>) để tự động hóa attribution. Tác giả cũng thừa nhận taxonomy rút từ ba miền có thể chưa bao quát mọi failure mode của miền agentic khác.

# 3. Phương pháp failure attribution được đề xuất

## 3.1. AgentRx

AgentRx là framework chẩn đoán domain-agnostic, không phải model được fine-tune. Nó dùng tool schema, policy miền tùy chọn và nội dung trajectory để sinh constraint; những constraint bị vi phạm tạo validation log theo step có evidence. LLM judge dùng log này, toàn trajectory và [AgentRx Failure Taxonomy](<../failure taxonomy/AgentRx Failure Taxonomy.md>) để trả critical step cùng category.

Cách thực thi, loại constraint và điều kiện dừng được tách tại [AgentRx](<../failure attribution methods/AgentRx.md>).

# 4. Đánh giá và insight

## 4.1. Thiết lập cần thiết

Tác giả đánh giá trên cả 115 trajectory thuộc ba miền. Mỗi cấu hình chạy ba lần với GPT-5 mặc định; metric chính là exact critical-step accuracy, category accuracy và average step distance, ngoài ra có accuracy cho step lệch trong bán kính \(\pm1,\pm3,\pm5\). Baseline là LLM judge đọc trajectory với taxonomy và cùng procedure annotation nhưng **không** có validation log; AGENTRX thêm programmatic lẫn semantic constraint.

## 4.2. Violation evidence hữu ích, nhưng taxonomy checklist không thay thế được evidence trong mọi miền

Trên \(\tau\)-bench, thêm violation log đem cải thiện lớn so với baseline cả định vị step lẫn category; checklist đơn lẻ không tạo cải thiện. Flash hưởng lợi từ cả checklist lẫn violation. Nhưng ở Magentic-One, configuration checklist đơn lẻ mạnh nhất trong các one-shot setting, còn evidence vi phạm thưa/nhiễu không luôn giúp. Kết luận của tác giả là hai tín hiệu thường bổ sung nhau, không phải validation log luôn đủ hay taxonomy luôn tốt hơn.

## 4.3. Chiến lược sinh constraint phụ thuộc độ dài trajectory

Sinh one-shot trên toàn trajectory hiệu quả và tiết kiệm chi phí hơn ở \(\tau\)-bench có trace ngắn hơn. Với Magentic-One dài hơn, tác giả thấy one-shot dễ bị context dilution; sinh constraint theo từng prefix giúp hơn trên subset trace không quá 50 step. Flash cho pattern pha trộn. Vì vậy, paper không rút ra một chiến lược tốt nhất độc lập với horizon: one-shot là đánh đổi chi phí, còn step-by-step dùng context cục bộ để chống log dài.

## 4.4. Global và dynamic constraint bổ sung nhau

Trên \(\tau\)-bench -- miền có tool schema lẫn policy rõ -- cả global constraint (schema/policy) và dynamic constraint (task instruction + prefix) đều tốt hơn baseline. Dynamic-only mạnh hơn global-only ở category accuracy, nhưng kết hợp hai loại đạt tốt nhất. Điều này ủng hộ việc kiểm tra cả luật tồn tại trước trajectory và quan hệ phát sinh từ tool output/quan sát trước đó.

## 4.5. Judge không nên tách step và category một cách máy móc

Default của AgentRx dự đoán step và category trong một call. Pipeline Step-then-Category có thể commit vào step nhiễu trước khi dùng ngữ nghĩa taxonomy, nên giảm hiệu quả ở Flash và Magentic; ngược lại lại là setting tốt nhất ở \(\tau\)-bench, nơi trajectory gọn hơn. Tác giả vì vậy gắn độ bền của protocol judge với độ dài/nhiễu của trace, không coi hai-stage là cải thiện phổ quát.

# 5. Limitations và Future Works

## 5.1. Limitations tác giả nêu

1. **Taxonomy có thể chưa phủ hết miền khác.** Dù xuất phát từ ba miền và annotator độc lập theo miền, chín category có thể cần mở rộng với agentic domain khác.
2. **Validation signal có thể yếu hoặc false positive.** Tác giả trực tiếp nêu rằng điều này có thể đánh lạc hướng AgentRx; violation là evidence mềm nên judge phải tự loại case không liên quan.
3. **Framework vẫn gọi LLM.** Paper mô tả AgentRx giảm công annotation và one-shot rẻ hơn như một lựa chọn scale, chứ không nói chi phí suy luận đã được loại bỏ.

## 5.2. Future Works tác giả đề xuất

Bài không có section *Future Work* riêng. Hướng kỹ thuật tác giả nêu trong conclusion là xác định **tập tín hiệu chất lượng cao nhỏ nhất** đủ phân biệt failure thật với noisy flag. Đây là phản hồi trực tiếp cho hạn chế validation signal; paper không đưa roadmap thuật toán chi tiết hơn.
