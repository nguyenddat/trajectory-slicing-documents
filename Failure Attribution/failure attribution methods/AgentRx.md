# AgentRx

## Cách hoạt động

AgentRx là framework failure attribution trong [Barke et al. (2026), *AgentRx: Diagnosing AI Agent Failures from Execution Trajectories*](<AgentRx Diagnosing AI Agent Failures from Execution Trajectories.md>). Nó được đánh giá trên [AgentRx dataset](<AgentRx dataset.md>) và dùng [AgentRx Failure Taxonomy](<AgentRx Failure Taxonomy.md>) làm không gian nhãn/checklist.

**Đầu vào.** Toolset \(\mathcal{T}\) gồm tool/agent và schema input--output; domain policy \(\Pi\) tùy chọn; task instruction \(I\); và failed trajectory \(\mathcal{T}r=\langle s_1,\ldots,s_n\rangle\). Một step có thể có agent name, tool name, step index, tool output hoặc nội dung hội thoại. Framework còn nhận taxonomy checklist \(K\), trong đó mỗi category có các câu hỏi yes/no và tiêu chí ngắn.

**Quy trình suy luận.**

1. **Chuẩn hóa trace.** AgentRx chuyển format log dị thể của mỗi miền về intermediate representation chung để cùng schema constraint có thể đọc agent/tool/event ở từng step.
2. **Sinh global constraint một lần.** Từ tool schema và policy, framework sinh \(C_G\) rồi lưu global store. Đây là constraint không phụ thuộc một prefix cụ thể, như schema/protocol/capability rule.
3. **Sinh dynamic constraint theo step.** Với từng step \(k\), framework dùng \(I\), prefix \(\mathcal{T}r_{\leq k}\) và global store để sinh \(C^k_D\), lưu local store \(L_k\). Chúng mã hóa quan hệ phát sinh trong trajectory, ví dụ consistency giữa tool output trước đó và diễn giải hiện tại.
4. **Áp dụng constraint có guard.** Tại step \(k\), evaluate \(C_k=C_G\cup C^k_D\) chỉ khi guard của constraint thỏa trên prefix/current step. Assertion trả `SAT` hay `VIOL`; check có thể là predicate programmatic trên field có cấu trúc, hoặc semantic predicate do LLM checker đánh giá trên evidence được chỉ định.
5. **Ghi validation log.** Mỗi `VIOL` được thêm vào log theo step, kèm assertion, taxonomy target và evidence (event/window liên quan). Constraint pass không thành violation entry.
6. **Judge root cause.** Sau khi duyệt trajectory, LLM judge nhận \(I\), toàn trace, checklist \(K\) và validation log \(V\). Judge chọn step vi phạm sớm nhất đủ giải thích outcome sai, rồi chọn category phù hợp và rationale. Violation không là hard requirement: judge có thể bác false positive nhờ context toàn trajectory.

**Điều kiện dừng và đầu ra.** Constraint được sinh/evaluate đến step cuối \(n\); judge sau đó trả critical step \(\hat{s}\) và failure category \(\hat{y}\). Default *All-at-Once* dự đoán cả hai trong một call. Paper cũng thử *Step-then-Category*: call thứ nhất chọn step, call sau gán category có điều kiện trên step đó; đây là biến thể đánh giá, không phải output thêm agent id.

**Đặc điểm vận hành cần thiết.** Mặc định sinh/evaluate constraint theo prefix để phù hợp diagnosis incremental, nhưng paper còn thử *one-shot*: sinh một tập constraint từ toàn trajectory một lần để giảm chi phí. Constraint semantic bổ sung cho constraint programmatic nhưng có thể tạo tín hiệu nhiễu; vì vậy validation log là bằng chứng hỗ trợ thay vì verdict tự động. Global-only và dynamic-only đều là ablation; framework đầy đủ kết hợp cả hai.
