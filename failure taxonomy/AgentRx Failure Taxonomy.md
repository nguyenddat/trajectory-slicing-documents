# 1. AgentRx Failure Taxonomy

## 1.1. Phạm vi

Taxonomy của AgentRx là taxonomy xuyên miền gồm chín loại **root-cause failure**, được suy ra bằng grounded-theory coding từ các failed trajectory của API workflow, incident management và tác vụ web/file. Nó dùng để gán nhãn critical failure -- failure đầu tiên không được hệ phục hồi -- trong [AgentRx dataset](<AgentRx dataset.md>), và làm checklist ngữ nghĩa cho framework [AgentRx](<../failure attribution methods/AgentRx.md>).

Các loại không được tác giả nhóm tiếp theo một cấu trúc phân cấp. Source dùng không thống nhất tên loại thứ nhất: phần taxonomy viết *Plan Adherence Failure*, Table 1 viết *Instruction Adherence*, còn prompt judge viết *Instruction/Plan Adherence Failure*. Note dùng tên cuối để giữ cả hai khía cạnh, vì định nghĩa đều là không theo chỉ dẫn/kế hoạch. Taxonomy không bao gồm nhãn `Inconclusive`: đó là phương án dự phòng trong prompt judge khi không thể xếp một dự đoán vào chín loại đã định, không phải một root-cause category của benchmark.

## 1.2. Failure mode

1. **Instruction/Plan Adherence Failure:** agent bỏ qua chỉ dẫn hoặc kế hoạch đã thống nhất. Bao gồm thiếu thực hiện (bỏ bước) và thừa thực hiện (hành động/tool call ngoài kế hoạch) làm vi phạm plan, policy miền hoặc ràng buộc của orchestrator.
2. **Invention of New Information:** agent thêm, xoá hoặc thay đổi thông tin không có căn cứ trong input, context hay tool output; gồm hallucination, fact không được hỗ trợ hoặc bỏ thông tin liên quan không có lý do.
3. **Invalid Invocation:** tool call không hợp lệ, như thiếu argument bắt buộc, argument sai dạng hoặc giá trị không qua schema validation.
4. **Misinterpretation of Tool Output:** agent suy luận sai về tool output của mình hoặc của agent khác, dẫn đến giả định/hành động sai.
5. **Intent--Plan Misalignment:** agent hiểu sai mục tiêu hay ràng buộc của người dùng và lập kế hoạch sai.
6. **Underspecified User Intent:** không thể hoàn thành task vì tại thời điểm đó thiếu thông tin cần thiết.
7. **Intent Not Supported:** yêu cầu một hành động mà tool hiện có không hỗ trợ.
8. **Guardrails Triggered:** safety policy hoặc hạn chế truy cập bên ngoài chặn thực thi.
9. **System Failure:** lỗi kết nối/hệ thống khi gọi tool, chẳng hạn endpoint API không truy cập được.

## 1.3. Nghiên cứu nguồn

- [Barke et al. (2026), *AgentRx: Diagnosing AI Agent Failures from Execution Trajectories*](<../raw/AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>)
