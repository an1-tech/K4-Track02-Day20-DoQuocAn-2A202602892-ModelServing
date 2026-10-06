# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=10` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2602 | 524 / 580 | 25.8 / 26.5 | 2144 / 2251 / 2251 | 38.8 |
| UD-Q2_K_XL | 0.39 | 2492 | 705 / 808 | 28.2 / 33.1 | 2502 / 2856 / 2856 | 35.5 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.09x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation

_Ở lần chạy CPU ngl=0, 10 thread, mỗi quant hoàn thành 10/10 request. UD-Q2_K_XL giảm từ 0,50 xuống 0,39 GB (22%), nhưng decode giảm từ 38,8 xuống 35,5 tok/s (8,5%); TTFT P50 tăng từ 524 lên 705 ms và TPOT P50 tăng từ 25,8 lên 28,2 ms. Với cùng câu hỏi “Explain TTFT and TPOT in two short sentences.”, temperature=0 và max_tokens=128, đã xác nhận Q4_K_M trên 8080 và UD-Q2_K_XL trên 8090: primary giải thích nhầm TTFT/TPOT thành Time-to-Fault/Time-to-Potential Outage, còn compare nói về Teflon/Polycryl và in 3D. Cả hai đều sai ở câu hỏi này; một câu hỏi chưa đủ đánh giá chất lượng tổng thể. Ưu tiên Q4_K_M vì nhanh hơn trong baseline, dù tốn thêm 0,11 GB. Ít bit không đảm bảo nhanh hơn: chi phí dequantization và kernel có thể bù phần tiết kiệm bandwidth; chưa đo profiler nên đây là giả thuyết. Với nearest-rank và 10 mẫu, P95/P99 E2E đều là mẫu lớn nhất, không phải hai ước lượng tail độc lập._
