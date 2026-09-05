# AgenTracer-8B

## Cách hoạt động

AgenTracer-8B là failure attributor được huấn luyện trên [TracerTraj](<TracerTraj dataset.md>) bởi nghiên cứu [AgenTracer](<AgenTracer Who Is Inducing Failure in the LLM Agentic Systems.md>). Nó không phải pipeline tạo nhãn AgenTracer; nhiệm vụ của model là dự đoán decisive error pair \((i^*,t^*)\) trên một failure trajectory mới.

**Đầu vào.** Ở suy luận, model nhận toàn bộ failed trajectory/log và environmental feedback đi kèm. Trong setting *w/ G* của bài, nó còn được cung cấp ground-truth solution; setting *w/o G* không có thông tin này. Khi huấn luyện, mỗi trajectory còn có gold pair agent–step từ TracerTraj.

**Quy trình suy luận và học.**

1. Trong mỗi rollout huấn luyện, policy Qwen3-8B hiện tại sinh một nhóm candidate pair \((\hat i_k,\hat t_k)\) cho cùng full trajectory.
2. Mỗi candidate được chấm bằng multi-granular reward: output phải đúng format; reward agent-level là nhị phân theo việc \(\hat i_k=i^*\); reward step-level giảm trơn theo khoảng cách giữa \(\hat t_k\) và \(t^*\) bằng Gaussian kernel.
3. GRPO cập nhật policy từ so sánh reward tương đối giữa các candidate. Bài bỏ KL term của GRPO chuẩn và dùng dynamic clipping để chuyển dần từ exploration sang exploitation.
4. Khi infer, model áp dụng theo thiết lập all-at-once: suy luận trên toàn trajectory trong một lượt sinh, rồi xuất agent và step dự đoán. Phần reasoning được định dạng trong thẻ `think`, còn đáp án parse được có dạng `agentID | stepID`.

**Điều kiện dừng và đầu ra.** Model không duyệt tuần tự hoặc replay log ở inference. Lần sinh kết thúc khi hoàn tất output có cấu trúc; output là một agent chịu trách nhiệm, một decisive error step, và reasoning/explanation đi kèm. Replay counterfactual chỉ thuộc pipeline tạo nhãn TracerTraj, không được chạy mỗi lần AgenTracer-8B suy luận.

**Đặc điểm vận hành cần thiết.** Huấn luyện dùng Qwen3-8B làm backbone, batch size 32, 8 rollouts và learning rate \(10^{-6}\). Thiết kế reward cho partial credit ở step nhằm tạo tín hiệu dày hơn exact match step, còn format reward bảo đảm kết quả có thể parse để đưa vào vòng debug. Việc nạp toàn log trong một lần cho inference theo lựa chọn all-at-once của bài, nên model giữ được bằng chứng ở đầu và cuối trace nhưng vẫn bị ràng buộc bởi long-context input.
