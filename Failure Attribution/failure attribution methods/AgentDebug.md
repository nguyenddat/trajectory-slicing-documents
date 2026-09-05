# AgentDebug

## Cách hoạt động

AgentDebug là framework debug cho single LLM agent trong [Zhu et al. (2025), *Where LLM Agents Fail and How They can Learn From Failures*](<Where LLM Agents Fail and How They can Learn From Failures.md>). Nó dùng [AgentErrorTaxonomy](<AgentError Taxonomy.md>) để chẩn đoán failed trajectory của [AgentErrorBench](<AgentErrorBench dataset.md>), rồi re-rollout từ điểm lỗi với feedback có mục tiêu.

**Đầu vào.** Một trajectory \(\tau=\{(s_t,a_t)\}_{t=1}^{T}\), taxonomy \(E_{AET}\), tiêu chí critical error \(C_{crit}\), và ngân sách lặp \(I\). Ở triển khai prompt, ngữ cảnh còn gồm mô tả task, environment, các output memory/reflection/planning/action ở từng bước và phản hồi môi trường.

**Quy trình suy luận.**

1. **Gán nhãn module–bước.** Với mỗi bước và mỗi module `memory`, `reflection`, `planning`, `action`, `MapToAET` gán một error type theo taxonomy. Detector prompt yêu cầu evidence và reasoning từ chính nội dung module cùng context tại bước đó.
2. **Chọn lỗi critical.** Nếu `Eval(τ)` đã thành công thì trả lại trajectory. Nếu không, `DetectCriticalErrors` nhận toàn bộ trajectory, profile lỗi, taxonomy và tiêu chí \(C_{crit}\); nó trả về các bước critical \(T_*\), module \(M_*\), error type \(Z_*\) và feedback ban đầu \(\phi^{(0)}\). Nếu không có bước nào, phương pháp trả về `Failure`; nếu có nhiều bước, nó chọn \(t_*=\min(T_*)\), tức bước critical sớm nhất.
3. **Re-rollout với feedback.** Hệ re-execute từ \(t_*\) với \(\phi^{(k-1)}\). Nếu trajectory mới thành công, trả về trajectory đã sửa. Nếu vẫn thất bại, `UpdateFeedback` tinh chỉnh feedback từ rollout mới và thử tiếp từ cùng điểm đó.

**Điều kiện dừng và đầu ra.** Dừng khi `Eval` báo thành công, khi detector không tìm được critical step, hoặc khi đã dùng hết \(I\) lượt re-rollout. Đầu ra là corrected trajectory hoặc `Failure`; báo cáo chẩn đoán trung gian gồm `critical_step`, `critical_module`, `error_type`, evidence, root cause, correction guidance và các cascading effects.

**Đặc điểm vận hành cần thiết.** Bài thực nghiệm đặt tối đa năm re-rollout và cho mọi baseline cùng ngân sách token. Việc định vị không tìm agent chịu trách nhiệm; nó định vị bước, module và loại lỗi trong một single-agent trajectory. Thuật toán ghi rõ Stage 2 là LLM detection “no rollout/counterfactuals”, dù phần văn xuôi gọi bước này là “counterfactual testing” bằng thay action rồi kiểm tra rollout. Prompt công bố thực tế yêu cầu LLM suy luận toàn cục rằng sửa lỗi đó *có thể* làm trajectory thành công; vì vậy nguồn không cho một thủ tục counterfactual thực thi nhất quán để xác minh nhãn critical.
