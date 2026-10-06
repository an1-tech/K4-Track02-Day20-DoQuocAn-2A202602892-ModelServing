# Bonus - Batch-size sweep (chunked prefill)

Host `Windows-AMD64` · llama.cpp `b10488` ·
`threads=10` `ngl=0` · metric `pp512`

| -b (logical) | -ub (micro) | pp512 (tok/s) | vs best |
|:--|--:|--:|--:|
| 128 | 128 | 110.8 | 69% |
| 256 | 256 | 134.2 | 83% |
| 512 | 256 | 131.4 | 82% |
| 512 | 512 | 119.4 | 74% |
| 1024 | 512 | 149.2 | 93% |
| 2048 | 512 | 161.1 | 100% |

Best: `-b 2048 -ub 512` at 161.1 tok/s
(1.45x the slowest point tested).

This sweep only measures the throughput half of the trade. The cost it hides is
TTFT for queued requests: a larger micro-batch holds the device longer per step,
so anything waiting behind it waits longer. To see both halves, re-run
`make load-50` with your best and worst settings via
`.venv/bin/python labs/02-serve/serve.py -- -b N -ub M` and compare P95.

## Your finding

_Ở sweep CPU, 10 thread, pp512 và 2 lần lặp mỗi điểm, cấu hình -b 2048 -ub 512 đạt 161,1 tok/s; -b 128 -ub 128 đạt 110,8 tok/s, chênh khoảng 1,45×. Đây là thay đổi cả logical batch và micro-batch, không tách riêng tác động từng knob. Batch lớn có thể giảm overhead cho prefill, nhưng các điểm -b 512/1024/2048 đều đủ chứa prompt 512 token và cùng -ub 512 vẫn khác tốc độ: scheduling, cache, nhiệt độ/power limit và thứ tự sweep có thể góp phần. Chỉ hai lần lặp, không có khoảng tin cậy nên chưa coi 1,45× là cải thiện production chắc chắn. -b 2048 -ub 512 là ứng viên cần kiểm tra dưới tải mixed short/long; phải đo lại TTFT/P95 và goodput tại cùng SLO. Sweep này chỉ đo throughput prefill, không đo tác động tới queued decode request hoặc chất lượng._
