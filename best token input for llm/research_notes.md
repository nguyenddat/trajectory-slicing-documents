# Best Token Input for LLM

## Kết luận

Không có một ngưỡng phần trăm cố định áp dụng cho mọi LLM. Context window tối đa là giới hạn kỹ thuật, không bảo đảm chất lượng vẫn ổn định khi input tiến gần giới hạn.

Nếu chưa có benchmark riêng, nên bắt đầu với khoảng **40–60% context window** cho input trong production. Đây là heuristic triển khai, không phải quy luật khoa học.

## Heuristic thực tế

| Mức sử dụng context | Đánh giá |
|---|---|
| 0–30% | Thường an toàn |
| 30–50% | Thường là vùng tốt để dùng production |
| 50–70% | Nên benchmark theo task thực tế |
| 70–85% | Rủi ro suy giảm tăng, nhất là RAG và reasoning |
| 85–100% | Chỉ dùng sau khi đã kiểm thử |

Phải tính cả output và overhead:

```text
input_budget = max_context - max_output - system_prompt - tool_overhead
```

## Vì sao không có ngưỡng chung?

- Chất lượng phụ thuộc vào model, tokenizer, loại tác vụ và độ nhiễu.
- Thông tin ở giữa context có thể bị bỏ qua nhiều hơn thông tin ở đầu hoặc cuối, dù chưa chạm giới hạn token.
- Tác vụ retrieval, multi-hop reasoning, summarization và generation có mức suy giảm khác nhau.
- Context dài còn làm tăng latency, chi phí và bộ nhớ KV cache.

## Khuyến nghị kiểm thử

Đo riêng accuracy, faithfulness, latency và chi phí ở các mức 25%, 50%, 75% và 90% context window. Với tài liệu dài, nên rerank/chunk, loại bỏ nội dung trùng lặp và đặt thông tin quan trọng gần đầu hoặc cuối context.

## Nguồn

1. Liu et al., *Lost in the Middle: How Language Models Use Long Contexts* (TACL 2023): https://arxiv.org/abs/2307.03172
2. Hsieh et al., *RULER: What's the Real Context Size of Your Long-Context Language Models?*: https://arxiv.org/abs/2404.06654

## Ghi chú phương pháp

Các nguồn được tìm và đọc qua Playwright. Consensus đã được thử theo yêu cầu, nhưng tài khoản đã hết quota tìm kiếm trong tháng tại thời điểm nghiên cứu.
