# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **10 physical · 12 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 19.6 | 53% |
| 5 | 36.8 | 100% |
| 10 | 33.0 | 90% |
| 12 | 29.1 | 79% |
| 24 | 16.2 | 44% |

**Best**: `-t 5` at 36.8 tok/s
**Slowest tested**: `-t 24` at 16.2 tok/s (2.27x spread)
**Against the physical-core default** (`-t 10`, 33.0 tok/s): 1.11x

Use this in your run:

```bash
LAB_N_THREADS=5 make bench
```

## Your explanation

_Knee nằm quanh 5 thread: tg128 tăng từ 19,58 tok/s ở 1 thread lên 36,76 ở 5, rồi giảm còn 33,04 ở 10, 29,07 ở 12 và 16,21 ở 24. Hạ từ mặc định 10 xuống 5 tăng throughput 1,11×; 5 so với 24 đạt 2,27× nhưng đó là so với cấu hình cố tình oversubscribe, không phải speedup so với mặc định. CPU i5-1335U có core không đồng nhất; số core physical không bảo đảm số thread tối ưu. Khi decode gần trần bandwidth, thêm thread có thể tăng tranh chấp cache/memory và chi phí scheduling thay vì thêm công việc hữu ích. Chưa đo counter phần cứng nên không khẳng định bandwidth là nguyên nhân duy nhất. Lượt sweep đầu có một smoke request chạy chồng; đã chạy lại toàn bộ sweep khi không gửi request và giữ kết quả lượt sau trong bảng/JSON này. Baseline HTTP và tg128 là hai workload khác nhau, không ghép tốc độ của chúng thành cùng một before/after._
