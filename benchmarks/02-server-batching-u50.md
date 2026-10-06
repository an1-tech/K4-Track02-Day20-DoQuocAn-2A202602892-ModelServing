# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 10 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.19 of 4 slots (80%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 2792 |

Highest sampled value was **3.19 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

_Đã lấy 10 mẫu trong 60 giây chồng thời gian với load 50 users. Peak n_busy_slots_per_decode là 3,19/4 (khoảng 80%), lớn hơn 1, chứng minh có batching. requests_processing đạt 4 và requests_deferred đạt 46, chứng minh các slot kín trong khi request khác chờ. Effective concurrency ở 50 users là 10,1, lớn hơn peak busy slots vì Little’s Law tính cả request chờ, còn gauge mô tả số slot trung bình mỗi decode call. Hai đại lượng không phải cùng phép đo nên không cần bằng nhau. Với bằng chứng có queue và slot đang xử lý, ưu tiên các gauge trực tiếp; RPS × latency chỉ là ước lượng, nhất là khi chỉ 16 request hoàn thành và còn request chưa xong lúc hết 60 giây. Server dùng lại từ smoke/load-10 nên peak gauge không phải thí nghiệm bắt đầu với mọi counter bằng 0. kv_cache_usage_ratio không được build xuất; giữ n/a, không coi là số 0 đo được._
