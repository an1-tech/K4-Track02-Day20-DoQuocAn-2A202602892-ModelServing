# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _Đỗ Quốc An_
**MSSV:** _2A202602892_
**Cohort:** _A20-K4_
**Ngày submit:** _2026-10-06_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** _Windows 11 Home Single Language (AMD64), Python 3.12.10_
- **CPU:** _13th Gen Intel Core i5-1335U_
- **Cores:** _10 physical / 12 logical_
- **CPU extensions:** _Không được probe Windows báo cáo; chưa xác nhận AVX2/AVX-512_
- **RAM:** _15,6 GB_
- **Accelerator:** _Intel Iris Xe Graphics; Vulkan được phát hiện, nhưng các phép đo dùng CPU với ngl=0_
- **llama.cpp asset đã tải:** _llama-b10488-bin-win-vulkan-x64.zip_
- **Model đã dùng:** _Qwen3.5 0.8B_ (`LAB_MODEL=`_qwen35-0.8b_)
- **Quantization:** _Q4_K_M_ + _UD-Q2_K_XL_ (từ `models/active.json`)

**Chạy ở đâu:** _Laptop local Windows; không dùng cloud fallback_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

_Chạy local với Qwen3.5 0.8B và CPU để giảm tải RAM. Python trần trỏ vào Microsoft Store nên dùng py -3.12 và Python trong .venv; đặt PYTHONUTF8=1. PowerShell 5.1 đọc sai UTF-8 của lab.ps1 nên gọi script Python trực tiếp. Probe thấy Vulkan, nhưng đo với LAB_N_GPU_LAYERS=0. Lỗi dấu phân cách đường dẫn của verify được kiểm tra bằng workaround trong bộ nhớ, không sửa mã nguồn._

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2602 | 524 / 580 | 25.8 / 26.5 | 2144 / 2251 / 2251 | 38.8 |
| UD-Q2_K_XL | 0.39 | 2492 | 705 / 808 | 28.2 / 33.1 | 2502 / 2856 / 2856 | 35.5 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

_2-bit giảm 22% dung lượng nhưng decode thấp hơn 8,5%; TTFT cũng cao hơn. Cùng câu hỏi TTFT/TPOT, primary nhầm sang fault/outage, compare nhầm sang vật liệu in 3D. Cả hai sai; một prompt chưa đủ chấm tổng thể. Chọn Q4_K_M vì nhanh hơn trong lần đo này._

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.45 | 18000 | 34000 | 34000 | 8.7 | 0.0% |
| 50 | 0.30 | 34000 | 53000 | 53000 | 10.1 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** _0.66×_
- **P95 tăng:** _1.56×_
- **Effective concurrency ở 50 users:** _10.1_ so với `--parallel` = _4_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _3.19_ / _4_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_RPS giảm 0,66×; P95 tăng 1,56×, concurrency 10,1 vượt 4 slot, deferred đạt 46: có queue. Chọn SLO P95 E2E ≤ 40 giây: 10 users đạt, 50 không đạt. Thử 5 thread rồi đo lại P95; không mặc định thêm slot. Chỉ 16 request hoàn thành ở load 50, nên percentile và Little’s Law còn hạn chế; chưa tính goodput chính xác._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | Localhost thay cho cloud/IaC | stub |
| N17 Data pipeline | Danh sách dữ liệu trong bộ nhớ | stub |
| N18 Lakehouse | TOY_DOCS thay cho lakehouse | stub |
| N19 Vector + features | Keyword overlap, không vector index | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _0.0 ms_
- retrieve: _0.1 ms_
- llm: _9890.1 ms_
- **stage chiếm nhiều nhất:** _llm_ (_gần 100%_ của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

_LLM chiếm gần 100% latency, phù hợp với retrieval đồ chơi chỉ 0,1 ms. Muốn giảm tổng 2× phải tối ưu gọi LLM: thử 5 thread, giảm output/context thừa rồi đo lại và chấm chất lượng. Embed 0,0 ms là không gọi embedder thật. Pipeline chạy thành công nhưng vẫn có câu trả lời sai._

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _Hạ số thread CPU từ mặc định 10 xuống 5; cùng Q4_K_M, ngl=0, workload llama-bench tg128_

```
before:  33.0 tok/s (-t 10, mặc định physical cores)
after:   36.8 tok/s (-t 5)
speedup: 1.11× (36.76 / 33.04, số gốc trong JSON)
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

_Hạ thread từ 10 xuống 5 tăng tg128 từ 33,0 lên 36,8 tok/s, tương đương 1,11× theo số gốc. Throughput tăng mạnh từ 1 lên 5 thread, rồi giảm ở 10, 12 và 24; knee nằm quanh 5. Thêm thread vượt knee có thể tăng tranh chấp cache/memory và chi phí scheduling khi decode đã gần trần tài nguyên. i5-1335U có core không đồng nhất nên 10 physical cores không đồng nghĩa 10 thread tối ưu. Đây là giải thích phù hợp với đường cong, chưa phải kết luận từ profiler; nhiệt độ, power limit và workload nền cũng có thể ảnh hưởng. Lượt sweep đầu có smoke request chạy chồng nên đã chạy lại toàn bộ khi không gửi request; bảng/JSON giữ lượt sau. Server load vẫn dùng 10 thread để giữ baseline nhất quán. Speedup 1,11× chỉ đo trên llama-bench tg128, chưa chứng minh cùng mức cải thiện cho HTTP hoặc P95; cần load test lại trước khi coi 5 thread là cấu hình production._

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _B2: batch-size sweep trên CPU; B3: before/after của sweep này. Chưa làm B1/B4/B5_

**Numbers:**

```
before:  110.8 tok/s (pp512, -b 128 -ub 128)
after:   161.1 tok/s (pp512, -b 2048 -ub 512)
speedup: 1.45× (cùng CPU, model Q4_K_M, 10 thread, ngl=0)
```

**Điều này nói lên gì mà deck chưa nói:**

_Batch/micro-batch thay đổi throughput prefill trên cùng máy: điểm tốt nhất đo được nhanh hơn điểm nhỏ nhất khoảng 1,45×. Đây là số từ bonus, không dùng lại thread sweep base. Tuy nhiên đổi cả -b và -ub chưa tách được cơ chế riêng. Các điểm có cùng -ub 512 vẫn dao động dù prompt chỉ 512 token, nên thứ tự chạy, cache và power limit có thể góp phần. Chỉ hai lần lặp; chưa đo queued TTFT/P95, không suy ra production goodput tăng 1,45×. Cấu hình -b 2048 -ub 512 chỉ là ứng viên cho load test tiếp theo._

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [x] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

_Dùng ChatGPT/Codex để đọc đề, hướng dẫn lệnh PowerShell, chạy phép so chất lượng, thread sweep, smoke/load/metrics/pipeline trên laptop, và hỗ trợ điền số liệu cùng diễn giải vào các chỗ trống của mẫu. Số đo do script chạy thật; phần nhận xét do AI hỗ trợ soạn cần được người nộp đọc, kiểm tra và hiểu trước khi nộp. Không tạo screenshot giả._
