# 1. Bối cảnh nghiên cứu chung

## 1.1. Hệ LLM agentic và vòng lặp cải thiện

Hệ LLM agentic dùng LLM để lập kế hoạch, suy luận, gọi công cụ và tương tác với môi trường qua nhiều bước. Phạm vi trong kho tài liệu gồm cả hệ **multi-agent** — các tác tử có vai trò khác nhau phối hợp theo một mục tiêu — lẫn scaffold single-agent và deep-research agent. Các hệ này đã cho thấy tiềm năng trong lập trình, khám phá khoa học và giải quyết những tác vụ thực tế phức tạp, nhưng chuỗi thực thi dài cùng tương tác với công cụ làm chúng khó dự đoán và khó debug.

Khi một lần chạy thất bại, vòng lặp phát triển thường là: đánh giá trên benchmark, chẩn đoán lần chạy thất bại, rồi sửa hoặc tinh chỉnh hệ thống. Do đó, đánh giá không chỉ cần cho biết kết quả cuối có thành công hay không; nó cần tạo được bằng chứng để quyết định **cần cải thiện ở đâu**.

## 1.2. Khoảng trống giữa đánh giá outcome và chẩn đoán quá trình

Nhiều benchmark agent chủ yếu chấm outcome cuối, chẳng hạn task completion hoặc final answer. Một điểm số hay nhãn failure có thể cho biết hệ không đạt mục tiêu, nhưng không tự chỉ ra phần nào của trajectory làm kết quả trở nên sai hoặc không đáng tin, cũng không cho biết thành phần nào cần sửa. Vì vậy, việc nối evaluation với cải thiện vẫn thường phụ thuộc vào người phát triển đọc log, hiểu kiến trúc và dùng chuyên môn miền.

Vấn đề này đặc biệt khó vì trajectory agentic có thể dài, đa bước, xác suất, chứa suy luận ngôn ngữ tự nhiên, message giữa agent, tool output và thay đổi môi trường. Lỗi sớm còn có thể lan truyền qua các bước sau, trong khi một lỗi quan sát được cũng có thể được agent hoặc verifier ở sau khắc phục. Chỉ nhìn outcome cuối — hoặc chọn lỗi xuất hiện đầu tiên theo thời gian — vì thế chưa đủ để xác định điểm can thiệp có ích.

## 1.3. Các mục tiêu chẩn đoán không đồng nhất

Các nghiên cứu dùng những mục tiêu bổ sung cho nhau, không nên gộp chung thành một loại nhãn "failure attribution":

- **Đặc trưng hóa failure mode:** mô tả các mẫu lỗi lặp lại trong trace để so sánh hệ thống hoặc nhận diện điểm yếu về thiết kế, phối hợp hay kiểm chứng.
- **Định vị lỗi trong trajectory:** tìm một hay nhiều span/bước có lỗi, đồng thời phân biệt chúng với khám phá vô hại, tentative hypothesis, noise hoặc lỗi đã được phục hồi.
- **Quy kết nguyên nhân để can thiệp:** tìm agent/component và bước lỗi có quan hệ với failure cuối theo tiêu chí nhân quả hoặc recoverability, chẳng hạn decisive error hoặc first unrecoverable failure.

Taxonomy hay tập span lỗi không tự động trả lời agent/bước nào là nguyên nhân cần sửa; ngược lại, một nhãn nguyên nhân đơn lẻ không thay thế failure profile của toàn trace. So sánh chi tiết các formulation attribution được tách tại [Failure Attribution - Formulation](<Failure Attribution - Formulation.md>).

## 1.4. Khả năng quan sát của execution trace

Chẩn đoán ở mức quá trình đòi hỏi trace, thay vì chỉ output cuối. Tùy kịch bản, bằng chứng cần thiết có thể gồm task instruction, prompt và input context, message trung gian, lời gọi tool cùng kết quả, cấu hình agent và trạng thái môi trường. Output-only trace vẫn phù hợp cho một số bối cảnh black-box, nhưng trong debug developer-facing, thông tin thiếu có thể khiến nguyên nhân không thể quy kết đáng tin cậy; trace đầy đủ và môi trường replay được cũng cho phép kiểm tra giả định chẩn đoán.

Ngay cả khi trace có sẵn, chúng thường dài, dị thể và có cấu trúc theo framework. Một bộ chẩn đoán vì vậy phải vừa giữ được quan hệ thời gian/phụ thuộc giữa các bước, vừa phân biệt suy luận của agent, lỗi tool hay lỗi hệ thống; đây là lý do chẩn đoán trace không chỉ là đọc một transcript ngắn bằng LLM.

## 1.5. Nút thắt dữ liệu gán nhãn và khả năng mở rộng

Các benchmark chẩn đoán cần trajectory có nhãn theo mục tiêu tương ứng — failure mode, span/bước lỗi, hay component–step nguyên nhân — cùng evidence để kiểm chứng. Việc chuyên gia đọc trace dài và gán nhãn nhân quả tốn công, khó thống nhất giữa hệ/miền và khó mở rộng; vì vậy dữ liệu nhỏ, thiếu đa dạng hoặc không đủ observability đều giới hạn việc huấn luyện lẫn đánh giá attributor.

Các nghiên cứu phản hồi nút thắt này theo những đánh đổi khác nhau: annotation chuyên gia để bảo đảm nhãn và bối cảnh đầy đủ; taxonomy/guideline để chuẩn hóa; hoặc replay, counterfactual correction và fault injection để mở rộng dữ liệu. Các cơ chế này là lựa chọn thiết kế dữ liệu theo từng bài, không phải bằng chứng thay thế cho gold label trong mọi kịch bản.

## 1.6. Hướng nghiên cứu liên quan

Hướng chung là chuyển evaluation từ một phép chấm outcome thành một quy trình có thể hỗ trợ debug: quan sát trajectory, tạo tín hiệu chẩn đoán đúng loại, rồi nối tín hiệu đó với sửa đổi hệ thống hoặc re-rollout. Các nghiên cứu khác nhau ở phạm vi hệ (single/multi-agent), mức quan sát, đơn vị nhãn và nghĩa của "nguyên nhân"; vì vậy chỉ nên đối chiếu phương pháp sau khi giữ rõ formulation và dữ liệu mà chúng sử dụng.

## 1.7. Nghiên cứu liên quan trong kho tài liệu

- [Zhang et al. (2025), *Which Agent Causes Task Failures and When? On Automated Failure Attribution of LLM Multi-Agent Systems*](<Who&When dataset.md>) đặt tên và định nghĩa automated failure attribution cho MAS: nối failure với agent chịu trách nhiệm và decisive error step.
- [Cemri et al. (2025), *Why Do Multi-Agent LLM Systems Fail?*](<MAST dataset.md>) đặt câu hỏi nền tảng về lý do MAS thất bại, phát triển taxonomy và trace dataset để đặc trưng hóa failure mode xuyên hệ/tác vụ.
- [Deshpande et al. (2025), *TRAIL: Trace Reasoning and Agentic Issue Localization*](<Trail dataset.md>) chuyển trọng tâm từ end-to-end outcome sang đánh giá/định vị nhiều lỗi trên structured trace của cả workflow single-agent và multi-agent.
- [Zhu et al. (2025), *Where LLM Agents Fail and How They can Learn From Failures*](<AgentErrorBench dataset.md>) nêu error propagation ở single-agent và dùng diagnosis root cause làm điểm bắt đầu cho feedback, debug và re-rollout.
- [Kong et al. (2025), *Aegis: Automated Error Generation and Attribution for Multi-Agent Systems*](<Aegis dataset.md>) nhấn mạnh scalability deadlock: failure attribution cần dữ liệu agent–error mode có nhãn ở quy mô lớn, nhưng annotation người lại đắt.
- [Zhang et al. (2025), *AgenTracer: Who Is Inducing Failure in the LLM Agentic Systems?*](<TracerTraj dataset.md>) kế thừa decisive-error formulation, đồng thời tập trung tự động tạo trajectory có nhãn agent–step và huấn luyện failure tracer chuyên biệt.
- [Chen et al., *Seeing the Whole Elephant: A Benchmark for Failure Attribution in LLM-based Multi-Agent Systems*](<TraceElephant dataset.md>) đặt failure attribution trong kịch bản developer-facing và lập luận rằng output-only trace không đủ cho nhiều chẩn đoán; benchmark của bài giữ full execution trace cùng môi trường replay được.
- [Barke et al. (2026), *AgentRx: Diagnosing AI Agent Failures from Execution Trajectories*](<AgentRx dataset.md>) xét cả single-agent lẫn multi-agent, và định vị first unrecoverable failure thay vì mọi lỗi xuất hiện trong trace.
- [Wang et al. (2026), *Where Do Deep-Research Agents Go Wrong? Span-Level Error Localization in Agent Trajectories*](<raw/Where Do Deep-Research Agents Go Wrong Span-Level Error Localization in Agent Trajectories.md>) mở rộng bối cảnh sang deep-research agent: outcome cuối không tách được harmful error span khỏi exploration hoặc noise, và nhấn mạnh việc một claim thiếu căn cứ chỉ trở thành lỗi khi được dùng như cam kết có hệ quả.
