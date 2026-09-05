# Aegis-GRPO

## Cách hoạt động

Aegis-GRPO dùng Group Relative Policy Optimization (GRPO) để huấn luyện model attribution của [AEGIS](<Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>) bằng reward phân cấp, thay vì chỉ bắt chước target JSON như [Aegis-SFT](<Aegis-SFT.md>).

**Đầu vào.** Prompt chứa task, failed trajectory, và taxonomy error mode. Trong huấn luyện, reward còn nhận gold attribution map để đối chiếu; ở suy luận, model chỉ nhận prompt/trajectory và taxonomy.

**Quy trình suy luận và học.**

1. Policy sinh các output JSON attribution cho cùng một prompt; mỗi output được parse thành tập các cặp `(agent, error mode)`.
2. Output JSON không hợp lệ nhận reward âm. Với output parse được, reward cho điểm theo từng cặp đúng, đồng thời cho partial credit không lặp lại khi chỉ đúng agent hoặc chỉ đúng error mode.
3. Reward cộng điểm nhỏ cho format hợp lệ, phạt cặp dự đoán trùng và danh sách dự đoán quá dài, rồi chuẩn hóa theo score tối đa có thể đạt của gold label.
4. GRPO dùng các reward này để cập nhật policy theo so sánh tương đối giữa các output cùng prompt.

**Điều kiện dừng và đầu ra.** Khi suy luận, phương pháp không tìm kiếm tuần tự trong log: model dừng khi hoàn tất một JSON và trả về attribution map agent–error mode. Trong một lượt huấn luyện, chấm reward dừng sau khi output được parse và so với gold label.

**Đặc điểm vận hành cần thiết.** Reward giàu tín hiệu hơn exact-match nhị phân vì phân biệt đúng cặp, đúng agent và đúng mode. Đổi lại, nó chỉ dùng được khi huấn luyện có gold label có cấu trúc; strict JSON là một phần của objective vì output sai format bị phạt.
