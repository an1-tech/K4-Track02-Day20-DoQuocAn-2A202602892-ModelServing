# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 15163.7 | 15163.8 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 7097.9 | 7098.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.2 | 7408.8 | 7409.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **9890.1** · total **9890.3**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput because it **ignores SLOs (Service Level Objectives) at saturation**.

Here is the breakdown of the difference based on the text:

*   **Raw Throughput** ignores SLOs: The text states, "Throughput at saturation ignores SLOs." This means that if a service hits its limits (saturation), the raw throughput metric will not re

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing key-value pairs in non-contiguous pages.

By moving the KV cache from contiguous memory blocks to non-contiguous pages, the architecture eliminates wasted space that would otherwise be consumed by internal fragmentation, thereby optimizing GPU memory usage.

**When does splitting prefill and decode help?**

> Based on the provided context, splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bandwidth-bound**.

This is because the context states that prefill is compute-bound and decode is memory-bandwidth-bound. By splitting these operations, the system can utilize different pools for each phase, allowing the engine to skip prefill entirely when a shared prefix is ava


## Which N16-N19 pieces are real

_N16 là stub localhost thay cho cloud/IaC; N17 là stub danh sách trong bộ nhớ; N18 là stub TOY_DOCS thay cho lakehouse; N19 là stub keyword overlap, không dùng vector index/embedding server; N20 là endpoint llama-server thật. Ba query đều hoàn thành, in ID/score context và trả lời. Mean embed=0,0 ms, retrieve=0,1 ms, llm=9890,1 ms, total=9890,3 ms: LLM chiếm gần 100%. Giá trị embed=0,0 là không gọi embedder cộng với làm tròn, không phải tốc độ embedding thật. Stage llm bao gồm HTTP, chờ server, prefill và decode, không phải chỉ compute. Để giảm total 2× phải tác động vào stage này: thử thread tối ưu, giới hạn output phù hợp và giảm context thừa; sau đó đo lại cùng query và chấm chất lượng. Không thể giảm tổng đáng kể bằng tối ưu retrieval 0,1 ms. Có lỗi nội dung dù pipeline chạy thành công: query goodput mở đầu nói goodput bỏ qua SLO rồi mâu thuẫn với giải thích sau; query disaggregation trộn thêm prefix caching. Vì vậy báo cáo này chứng minh tích hợp và latency, không chứng minh chất lượng RAG._
