# 1. Static Agentic

## 1.1. Cách hoạt động

Static Agentic là cấu hình agent-based được dùng để đánh giá trên [TraceElephant](<TraceElephant dataset.md>). Nó không phải model attribution được huấn luyện riêng: tác giả sửa mini-SWE-agent để điều tra full failure trace và kết luận cặp component/agent chịu trách nhiệm với decisive failure step.

**Đầu vào:** trace tĩnh đầy đủ — metadata cấp trace và I/O của từng step, gồm tool log nếu có. Trong kịch bản evaluation with ground truth, agent còn nhận reference task-level; nếu không, nó chỉ dùng trace.

**Quy trình suy luận:**

1. Agent có tool xem output của tất cả step để nắm diễn tiến toàn cục.
2. Khi cần kiểm tra một vị trí, nó gọi tool xem cả input và output của step đó; nhờ vậy có thể chuyển qua lại giữa toàn trace và context cục bộ.
3. Agent lặp việc truy xuất field cần thiết rồi submit kết luận attribution.

**Điều kiện dừng và đầu ra:** khi gọi tool submit final answer, nó trả agent/component chịu trách nhiệm và decisive failure step. Nguồn không công bố policy duyệt field, prompt hoặc số lượt tool call cố định; không suy ra rằng nó luôn xét tuần tự hay tất cả step.

**Đặc điểm vận hành:** cấu hình chỉ điều tra trace tĩnh, không re-run hệ. Ba extension thêm vào mini-SWE-agent là: xem output mọi step; xem input/output của một step; submit câu trả lời cuối.
