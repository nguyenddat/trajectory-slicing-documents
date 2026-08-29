# 1. Vấn đề đặt ra

## 1.1. Bối cảnh và khoảng trống

Bối cảnh hệ LLM multi-agent dễ thất bại, cũng như nhu cầu nối kết quả đánh giá với thành phần cần cải thiện, đã được trình bày tại [Bối cảnh chung](<../Bối cảnh chung.md>). Với một trajectory dài bị thất bại, [Who&When](<../Failure Attribution - Formulation.md>) cho thấy các LLM reasoning mạnh vẫn rất yếu trong việc xác định đúng agent và bước đã gây lỗi quyết định. Theo tác giả, hai khoảng trống còn lại là: thiếu **dữ liệu gán nhãn đủ lớn** để huấn luyện failure tracer, và thiếu một tracer vừa chính xác vừa đủ nhẹ để chẩn đoán log multi-agent dài.

Các benchmark gán nhãn thủ công hiện có mà bài nêu — [MAST](<MAST dataset.md>) và Who&When — chỉ có lần lượt khoảng 200 và 127 failure trajectory theo cách tác giả tường thuật. Điều này không đủ cho việc huấn luyện hay đánh giá có hệ thống ở quy mô lớn. Bài không phê bình formulation của Who&When; thay vào đó, nó coi kết quả thấp của các LLM trên formulation này là bằng chứng rằng cần đồng thời mở rộng resource huấn luyện và xây một attributor chuyên biệt.

## 1.2. Formulation được kế thừa từ Who&When

AgenTracer **kế thừa**, chứ không thay đổi, formulation automated failure attribution của [Who&When](<../Failure Attribution - Formulation.md#1-failure-attribution>). Hệ được xét theo lượt, với một agent hành động ở mỗi bước. Với trajectory thất bại \(\tau\), cần trả về cặp \((i^*,t^*)\): agent chịu trách nhiệm và **bước lỗi quyết định sớm nhất** — hành động mà nếu được sửa bằng hành động oracle rồi mô phỏng lại phần sau thì outcome chuyển từ failure sang success.

Điểm cụ thể hóa của bài nằm ở **cách thực thi** rectification oracle để tạo nhãn: analyzer agent đề xuất một sửa đổi cục bộ, sau đó pipeline replay trajectory để kiểm tra outcome. Vì vậy, đầu ra vẫn là một nguyên nhân chính có định vị thời gian, không phải taxonomy error mode hay tập nhiều agent–error pairs như AEGIS. Formalization dùng chung được cập nhật tại [Failure Attribution - Formulation](<../Failure Attribution - Formulation.md#1-failure-attribution>).

## 1.3. Nghiên cứu nguồn

- [Zhang et al. (2025), *AgenTracer: Who Is Inducing Failure in the LLM Agentic Systems?*](<../raw/AgenTracer Who Is Inducing Failure in the LLM Agentic Systems.md>)

# 2. Thiết kế dữ liệu TracerTraj

## 2.1. Đặc trưng cơ bản

**TracerTraj-2.5K** gồm các failed multi-agent trajectory, mỗi mẫu gắn một cặp nhãn \((i^*,t^*)\) gồm failure-responsible agent và decisive error step. Đây là dataset mới do AgenTracer tạo để huấn luyện/evaluate tracer, chứ không phải benchmark manual mới độc lập như Who&When.

Appendix báo cáo 2.476 trajectory–error-step pair (1.288 coding, 630 mathematical reasoning, 558 general agentic task), nên tên “2.5K” là số làm tròn. Cùng bảng còn ghi 4.655 *curated trajectories* (2.170, 1.185 và 1.300 theo ba miền), nhưng không giải thích chính xác quan hệ giữa tổng này với 2.476 pair có nhãn. Tác giả nói test split theo tỷ lệ 9:1, trong khi các dòng test cộng thành 266 mẫu (147/63/56, khoảng 10,7% của 2.476). Các tổng này cần được kiểm tra lại với bản phát hành dataset trước khi tái lập; bài cũng không tách số lượng nhãn đến từ counterfactual replay và từ fault injection.

Nó bao phủ sáu framework MAS, trải cả ba mức tự động hóa:

| Mức tự động hóa | Framework |
| --- | --- |
| Cấu hình thủ công | MetaGPT, AutoGen, Smolagents |
| Tự động hóa một phần | AgentPrune |
| Tự động hóa hoàn toàn | AFlow, OWL-Workforce |

Phần thiết lập chính nói có sáu benchmark: MBPP+, KodCode, Blackjack (coding), GAIA (general agentic), MATH và GSM8K (math). Tuy nhiên, Table 3 ở appendix lại thêm **HotpotQA** vào general-agentic domain. Như vậy tài liệu nguồn mâu thuẫn về số benchmark (sáu ở phần chính, bảy nếu theo appendix); note không tự loại HotpotQA để hợp thức hóa số liệu.

## 2.2. Nguồn dữ liệu mẫu và phương pháp sinh dữ liệu

Tác giả chạy mỗi framework trên truy vấn từ các benchmark nói trên, thu trajectory thô rồi tách theo evaluator nhị phân thành \(T_{succ}\) và \(T_{fail}\). Hai nguồn được dùng song song:

- **Failure tự nhiên \(T_{fail}\):** pipeline thử sửa từng bước bằng counterfactual replay để tìm lỗi quyết định sớm nhất.
- **Failure tổng hợp từ \(T_{succ}\):** pipeline làm nhiễu có chương trình một hành động trong trajectory đã thành công rồi replay. Nếu nhiễu đó làm hệ thất bại, agent và bước bị nhiễu là nhãn được biết theo cấu trúc.

Analyzer agent và perturbation operator đều dựa trên DeepSeek-R1. Analyzer nhận full failure trajectory, environmental feedback (ví dụ lỗi code/tool) và ground-truth solution; prompt yêu cầu một cách sửa cục bộ, không đưa nguyên lời giải hoàn chỉnh. Perturbation operator chọn sửa đổi cụ thể tại một bước; bài dùng nó để làm thất bại trajectory thành công chứ không dùng taxonomy lỗi định trước.

## 2.3. Các bước sinh và gán nhãn

1. **Thu và phân loại trajectory thô.** Chạy sáu MAS trên các truy vấn nguồn, rồi dùng evaluator \(\Omega\) phân chúng thành success hay failure.
2. **Gán nhãn failure tự nhiên bằng counterfactual replay.** Với mỗi \(\tau\in T_{fail}\), duyệt bước từ đầu tới cuối. Ở bước \(t\), analyzer đề xuất action sửa \(a'_t\), thay action gốc bằng \(a'_t\), rồi resimulate phần còn lại. Nếu trajectory mới thành công, lưu \((\tau,\langle\mu(t),t\rangle)\) và dừng duyệt trajectory đó; vì duyệt tuần tự, đây là decisive error sớm nhất tìm được.
3. **Sinh và gán nhãn failure tổng hợp.** Với mỗi \(\tau\in T_{succ}\), lấy ngẫu nhiên \(K\) điểm có thể tiêm lỗi. Ở từng điểm, perturbation operator thay action gốc bằng action bị corrupt rồi replay. Nếu outcome chuyển sang failure, lưu cặp \((\tilde\tau,\langle\mu(t),t\rangle)\) và dừng thử thêm trên trajectory gốc đó.
4. **Hợp nhất và chia dữ liệu.** Hợp \(D^-\) từ replay failure tự nhiên với \(D^+\) từ injection thành \(D_{tracer}\), sau đó lấy held-out test split theo tỷ lệ 9:1.

## 2.4. Mục đích và đánh đổi của thiết kế

Counterfactual replay khai thác failure đã xảy ra để nhãn không chỉ là một lỗi được cài sẵn; việc sửa lần lượt và kiểm tra outcome là nỗ lực vận hành hóa định nghĩa decisive error của Who&When. Fault injection bổ sung lỗi có nhãn chính xác theo cấu trúc từ những run vốn thành công, nên tác giả dùng nó để tăng diversity và precision của resource mà không cần chuyên gia đọc toàn bộ log.

Đổi lại, mọi nhãn của nhánh replay phụ thuộc vào việc analyzer đề xuất được action sửa thích hợp và vào ground truth/evaluator có sẵn; những failure không được một sửa đổi như vậy chuyển thành success không xuất hiện trong \(D^-\). Nhánh injection chỉ bao phủ các failure do một action bị perturb làm outcome chuyển sang failure. Đây là hệ quả trực tiếp của quy trình, không phải limitation mà tác giả nêu thành section riêng; bài cũng không báo cáo tỷ lệ hay phân bố hai nhánh để đánh giá mức đại diện của từng nhánh.

# 3. Đánh giá phương pháp và insight

## 3.1. Thiết lập đánh giá

Tác giả huấn luyện [AgenTracer-8B](<../failure attribution methods/AgenTracer-8B.md>) trên TracerTraj và đánh giá ở agent-level accuracy lẫn step-level accuracy. Ngoài test split theo ba miền của TracerTraj, họ dùng hai subset chưa thấy khi huấn luyện của [Who&When](<../Failure Attribution - Formulation.md>): handcrafted từ Magnetic-One và automated từ AG2. Mỗi attributor được xét cả khi có (*w/ G*) và không có (*w/o G*) ground-truth solution; khi infer, các model nhận toàn trajectory trong thiết lập all-at-once.

Họ còn kiểm tra giá trị chẩn đoán trong vòng self-improvement: đưa trajectory thất bại (không có \(G\)) cho AgenTracer-8B, Self-Refine hoặc CRITIC để sinh feedback, inject feedback vào lần giải kế tiếp của MaAS, OWL-Workforce hoặc MetaGPT, và lặp tối đa ba vòng trên GAIA, HumanEval+ và MATH-500.

## 3.2. Failure attribution vẫn khó với LLM tổng quát, kể cả khi có đáp án đúng

Theo tác giả, các model nhỏ cho step-level judgment rất thấp; nhiều model lớn cũng chưa định vị tốt decisive step. Việc đưa ground truth vào prompt không cải thiện một cách nhất quán và đôi khi còn làm điểm attribution giảm. Do đó, đáp án cuối là tín hiệu hữu ích nhưng không thay thế được suy luận quan hệ nhân quả qua toàn trajectory.

## 3.3. Tracer chuyên biệt cải thiện cả attribution ngoài phân bố

Tác giả báo cáo AgenTracer-8B vượt backbone Qwen3-8B và các proprietary LLM lớn trên cả agent-level lẫn step-level, không chỉ trong held-out TracerTraj mà còn trên Who&When chưa thấy. Đây là bằng chứng thực nghiệm của bài rằng dữ liệu từ replay/injection kết hợp reward đa độ phân giải có thể dạy một phần kỹ năng decisive-error localization chuyển sang failure log được gán nhãn thủ công.

## 3.4. Lỗi gốc có thể xuất hiện sớm nhưng chỉ lộ ra ở bằng chứng muộn

Case study của tác giả cho thấy một hành động lấy sai file ở bước sớm mới là root cause, dù biểu hiện rõ hơn xuất hiện ở một bước sau. Họ dùng trường hợp này để giải thích vì sao chẩn đoán tại một action bề mặt hoặc chỉ nhìn một đoạn ngắn của trace dễ quy kết cho triệu chứng downstream thay vì lỗi quyết định.

## 3.5. Attribution có thể cung cấp feedback hữu ích hơn phản tư chung chung

Trong thí nghiệm ba vòng, feedback từ AgenTracer-8B cải thiện các hệ/tác vụ được thử, còn CRITIC có thể làm hiệu năng giảm. Kết luận của tác giả là failure localization có định vị cụ thể có thể tạo feedback đủ hành động cho self-improvement; đây là kết quả trên các cấu hình thử nghiệm của bài, không phải bằng chứng rằng mọi MAS tự sửa được sau một lần attribution.

# 4. Limitations và Future Works

## 4.1. Limitations

Bài không có section *Limitations* và cũng không tự nêu một danh sách giới hạn của AgenTracer hay TracerTraj. Hai lưu ý có thể đối chiếu trực tiếp với nguồn là:

1. **Phụ thuộc ground truth không đơn giản.** Tác giả quan sát rằng cho \(G\) vào attributor không nhất quán giúp accuracy và đôi khi có thể làm phán đoán lệch đi. Điều này là caveat thực nghiệm của failure attribution, không phải một chứng minh giới hạn riêng của AgenTracer-8B.
2. **Báo cáo dataset không hoàn toàn nhất quán.** Như Mục 2.1, mô tả sáu benchmark mâu thuẫn với Table 3 có HotpotQA. Đây là vấn đề trong tài liệu nguồn cần kiểm tra lại trước khi dùng số liệu bao phủ để tái lập, không phải limitation mà tác giả tuyên bố.

Các hệ quả của cơ chế annotation — replay cần analyzer/ground truth và injection chỉ sinh failure từ perturbation một action — đã được tách rõ là đánh đổi thiết kế tại Mục 2.4, không quy chúng thành phát biểu của tác giả ngoài những gì bài nêu.

## 4.2. Future Works

Bài không đưa roadmap *Future Work* kỹ thuật cụ thể. Conclusion chỉ đặt hướng dài hạn: dùng failure tracer để tiến tới hệ agentic tự chẩn đoán, tự sửa và tự phát triển hơn. Bài không đề xuất thuật toán, dataset mở rộng, hay kế hoạch đánh giá cụ thể nào cho hướng này.
