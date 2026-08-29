# 1. MAST Failure Taxonomy

## 1.1. Phạm vi

MAST (*Multi-Agent System Failure Taxonomy*) là taxonomy thực nghiệm cho failure trong quá trình thực thi của LLM multi-agent system. Taxonomy gồm 14 failure mode, được nhóm theo ba nguồn vấn đề: thiết kế hệ thống, phối hợp giữa agent và xác minh tác vụ. Các mode được ánh xạ tới giai đoạn tiền thực thi, thực thi hoặc hậu thực thi, nơi nguyên nhân gốc thường xuất hiện.

## 1.2. Failure mode

### 1.2.1. System Design Issues

- **FM-1.1 — Disobey task specification:** không tuân thủ ràng buộc hoặc yêu cầu của tác vụ.
- **FM-1.2 — Disobey role specification:** không tuân thủ trách nhiệm hay giới hạn của vai trò agent.
- **FM-1.3 — Step repetition:** lặp lại không cần thiết một bước đã hoàn thành.
- **FM-1.4 — Loss of conversation history:** mất hoặc bỏ qua lịch sử tương tác gần đây.
- **FM-1.5 — Unaware of termination conditions:** không nhận ra điều kiện nên kết thúc tương tác.

### 1.2.2. Inter-Agent Misalignment

- **FM-2.1 — Conversation reset:** khởi động lại hội thoại không cần thiết, làm mất ngữ cảnh hoặc tiến trình.
- **FM-2.2 — Fail to ask for clarification:** không yêu cầu làm rõ khi thông tin mơ hồ hoặc thiếu.
- **FM-2.3 — Task derailment:** lệch khỏi mục tiêu hay trọng tâm của tác vụ.
- **FM-2.4 — Information withholding:** không chia sẻ thông tin quan trọng có thể ảnh hưởng quyết định của agent khác.
- **FM-2.5 — Ignored other agent’s input:** không xem xét đầy đủ thông tin hay đề xuất của agent khác.
- **FM-2.6 — Reasoning-action mismatch:** hành động không nhất quán với lập luận đã nêu.

### 1.2.3. Task Verification

- **FM-3.1 — Premature termination:** kết thúc tương tác/tác vụ trước khi trao đổi đủ thông tin hoặc đạt mục tiêu.
- **FM-3.2 — No or incomplete verification:** bỏ qua hoặc chỉ kiểm tra một phần kết quả/hành vi hệ thống.
- **FM-3.3 — Incorrect verification:** kiểm tra hoặc đối chiếu thông tin/quyết định không đầy đủ, dẫn đến xác nhận sai.

## 1.3. Nghiên cứu nguồn

- [Cemri et al. (2025), *Why Do Multi-Agent LLM Systems Fail?*](<raw/Why Do Multi-Agent LLM Systems Fail.md>)
