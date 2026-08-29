# Aegis-SFT

## Cách hoạt động

Aegis-SFT là cách học supervised của [AEGIS](<../raw/Aegis Automated Error Generation and Attribution for Multi-Agent Systems.md>) trên [Aegis dataset](<Aegis dataset.md>). Nó học trực tiếp ánh xạ từ log một failure sang attribution map.

**Đầu vào.** Ở cả train và suy luận, prompt chứa task, toàn bộ conversation log đã serialize của failed trajectory, vai trò của model và định nghĩa các error mode. Khi train, mỗi prompt còn đi cùng gold attribution map của AEGIS, được chuyển thành target JSON.

**Quy trình suy luận.**

1. Model nhận toàn bộ prompt/trajectory trong một lượt.
2. Model sinh tự hồi quy chuỗi JSON mô tả các faulty agent và tập error mode tương ứng.

**Điều kiện dừng và đầu ra.** Không có bước duyệt hoặc thu hẹp log. Lần sinh kết thúc khi model hoàn tất output; kết quả là JSON attribution map agent–error mode.

**Đặc điểm vận hành cần thiết.** Trong huấn luyện, Aegis-SFT tối đa hóa xác suất target JSON có điều kiện theo prompt (negative log-likelihood). Phương pháp cần gold label đầy đủ nhưng không cần thiết kế reward; ở suy luận, nó chỉ cần một lần sinh trên toàn trajectory.
