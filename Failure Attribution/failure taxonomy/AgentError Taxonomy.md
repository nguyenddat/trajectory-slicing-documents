# 1. AgentErrorTaxonomy

## 1.1. Phạm vi

AgentErrorTaxonomy là taxonomy cho failure trong rollout của **single LLM agent** mà [Zhu et al. (2025), *Where LLM Agents Fail and How They can Learn From Failures*](<Where LLM Agents Fail and How They can Learn From Failures.md>) đề xuất. Nó phân loại lỗi theo năm module của agent: memory, reflection, planning, action và system. Bốn nhóm đầu gắn với các output trong vòng lặp ra quyết định; nhóm system dành cho lỗi từ tool, hạ tầng hoặc giới hạn thực thi nằm ngoài suy luận của agent.

Taxonomy này là vocabulary để gán nhãn từng module/bước trong [AgentErrorBench](<AgentErrorBench dataset.md>) và để [AgentDebug](<AgentDebug.md>) định vị lỗi critical. Nó không quy kết lỗi cho một agent trong hệ multi-agent.

## 1.2. Các loại lỗi

### 1.2.1. Memory

- **Over-simplification / Incomplete Summary:** tóm tắt lịch sử quá thô, bỏ chi tiết cần cho suy luận sau.
- **Hallucination (False Memory):** nhớ lại sự kiện hay trạng thái chưa từng xảy ra.
- **Retrieval Failure:** có thông tin liên quan trong ngữ cảnh nhưng không truy xuất khi cần.

### 1.2.2. Reflection

- **Progress Misassessment:** đánh giá sai mức tiến độ hoặc trạng thái hoàn thành.
- **Outcome Misinterpretation:** diễn giải sai phản hồi trực tiếp của hành động/môi trường.
- **Causal Misattribution:** nhận ra failure nhưng quy sai nguyên nhân, làm lệch kế hoạch sau.
- **Hallucination:** reflection về sự kiện hay kết quả không hề xảy ra.

### 1.2.3. Planning

- **Constraint Ignorance:** bỏ qua ràng buộc như thời gian, ngân sách, không gian hay yêu cầu tác vụ.
- **Impossible Action:** đề ra bước không khả thi về vật lý hoặc logic dưới tiền điều kiện hiện có.
- **Inefficient Planning:** kế hoạch dài dòng hay thiếu hợp lý, lãng phí bước và tăng nguy cơ chạm giới hạn.

### 1.2.4. Action

- **Planning–Action Disconnect:** hành động thực hiện không khớp với ý định của kế hoạch.
- **Format Error:** action sai cú pháp/định dạng mà môi trường yêu cầu.
- **Parameter Error:** tham số action không hợp lý hoặc malformed.

### 1.2.5. System

- **Step Limit Exhaustion:** hết số bước cho phép dù hành vi có thể vẫn hợp lý.
- **Tool Execution Error:** tool/API thực thi lỗi và làm hỏng các bước sau.
- **LLM Limit:** lỗi do ràng buộc API/model như timeout hay token limit.
- **Environment Error:** simulator hay môi trường vi phạm quy tắc kỳ vọng (bug, crash, network), không phải lỗi của agent.

## 1.3. Nghiên cứu nguồn

- [Zhu et al. (2025), *Where LLM Agents Fail and How They can Learn From Failures*](<Where LLM Agents Fail and How They can Learn From Failures.md>)
