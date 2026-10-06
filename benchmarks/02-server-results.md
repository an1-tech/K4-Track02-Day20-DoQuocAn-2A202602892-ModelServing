# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=10` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 25 | 0.45 | 18000 | 34000 | 34000 | 8.7 | 0.0% |
| 50 | 16 | 0.30 | 34000 | 53000 | 53000 | 10.1 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.66x** (13% of linear) |
| P95 latency | **1.56x** |
| Effective concurrency at 50 users | 10.1 vs `--parallel 4` slots (occupancy/slot ratio 2.53) |

**Saturated.** Throughput delivered only 0.66x for 5x the offered load, and effective concurrency (10.1) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.66x while P95 moved 1.56x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

> **Small sample.** Only 16 requests completed in the
> shorter run, so these percentiles are indicative rather than solid. Note also that
> locust averages only *completed* requests: when the run ends with requests still
> queued, effective concurrency is an **under**-estimate. Trust the throughput-scaling
> row over the concurrency row here, and run longer (`-t 3m`) if you want firmer numbers.

## Your reading

_Tăng users từ 10 lên 50 không tăng throughput: RPS giảm từ 0,45 xuống 0,30 (0,66×), P95 tăng từ 34 lên 53 giây (1,56×). Effective concurrency tăng từ 8,7 lên 10,1, đều vượt 4 slot; ở load 50 metrics đo processing=4 và deferred=46. Các số này hỗ trợ kết luận queue đã hình thành và tăng tải không giúp throughput. Không xác định được knee chính xác chỉ từ hai mức users, cũng không thể quy toàn bộ phần latency tăng cho queue: long-rag/short mix thực tế và cạnh tranh tài nguyên có thể thay đổi compute time. Load 50 chỉ hoàn thành 16 request; Locust tính trên request đã hoàn thành nên RPS/percentile không phải steady-state chắc chắn. Với SLO minh họa P95 E2E ≤ 40 giây, mức 10 users đạt (34 giây), mức 50 không đạt (53 giây). Tất cả 25 request hoàn thành ở mức 10 có max 33,62 giây, nên goodput theo ngưỡng E2E 40 giây cho tập hoàn thành này bằng RPS 0,455; chưa có timing từng request để tính chính xác goodput mức 50 hoặc goodput theo TTFT/TPOT. Thử 5 thread trước vì sweep tg128 tăng từ 33,04 lên 36,76 tok/s; sau đó phải đo lại load để xác nhận tác động trên P95. Admission/concurrency limit cũng đáng thử để tránh hàng đợi dài; không mặc định thêm slot sẽ giúp trên CPU._
