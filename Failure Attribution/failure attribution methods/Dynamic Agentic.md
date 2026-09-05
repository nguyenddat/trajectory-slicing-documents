# 1. Dynamic Agentic

## 1.1. Cách hoạt động

Dynamic Agentic là cấu hình replay/counterfactual được dùng để đánh giá trên [TraceElephant](<TraceElephant dataset.md>). Nó mở rộng [Static Agentic](<Static Agentic.md>): mục tiêu là kiểm tra **trách nhiệm cục bộ** của candidate attribution, không sửa toàn task hay chứng minh đầy đủ quan hệ nhân quả của cả trajectory.

**Đầu vào:** full static trace, môi trường có thể replay của instance, và (tùy setting evaluation) reference outcome. Candidate chứa tối đa n=3 bộ (mistake agent, mistake step, reason) cùng expected oracle — output kỳ vọng của step nếu lỗi theo reason đó không xảy ra. Expected oracle do Static Agentic suy ra trong hypothesis, không phải ground-truth annotation của người.

**Quy trình suy luận:**

1. Chạy Static Agentic để sinh tối đa ba candidate attribution và expected oracle tương ứng.
2. Replay trajectory. Khi đến candidate step, middleware sửa input request bằng hướng dẫn để agent tránh lỗi được giả định.
3. Quan sát ba step tiếp theo (k=3): chúng có khớp expected oracle không, failure mode cũ có tái diễn không, và agent có đi chệch task objective không. Quan sát này được dùng để xác nhận hay tinh chỉnh candidate.

**Điều kiện dừng và đầu ra:** mỗi counterfactual dừng sau cửa sổ ba step, không chạy task đến completion. Output benchmark vẫn là agent/component và decisive step; bài không nêu quy tắc scoring hay tie-break để chọn giữa nhiều candidate, nên không thể mô tả một aggregation rule chi tiết hơn.

**Đặc điểm vận hành:** tác giả chọn k=3 vì cửa sổ ngắn hơn thường không lộ biến đổi hành vi, còn cửa sổ dài hơn thêm variance từ stochasticity downstream, tool noise và planning không liên quan. Temperature replay cố định 0,3. So với Static Agentic, cấu hình này cần re-execution và can thiệp input; tác giả báo cáo nó cải thiện chủ yếu step-level attribution.
