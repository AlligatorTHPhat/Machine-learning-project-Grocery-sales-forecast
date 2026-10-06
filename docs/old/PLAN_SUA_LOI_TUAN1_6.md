# PLAN_SUA_LOI_TUAN1_6.md — Kế hoạch sửa toàn bộ lỗi trong audit_tuan1_6.md

> Dành cho: Gemini Pro 3.1 (hoặc bất kỳ AI agent nào) làm việc cùng sinh viên.
> Nguồn sự thật (đọc theo thứ tự ưu tiên): `AGENTS.md` > file này > `HUONG_DAN_DO_AN.md` > `audit_tuan1_6.md`.
> Nếu file này mâu thuẫn với AGENTS.md → **dừng, báo người dùng**, không tự chọn.
> Đặt file này ở `docs/PLAN_SUA_LOI_TUAN1_6.md`.

---

## PHẦN A. KHỐI LUẬT KHÓA (người dùng dán nguyên khối này vào đầu MỖI phiên làm việc mới)

```
[LUẬT KHÓA — ĐỌC VÀ TUÂN THỦ TRƯỚC KHI LÀM BẤT CỨ ĐIỀU GÌ]

Bạn đang kèm sinh viên sửa đồ án Favorita. Nguồn sự thật: docs/AGENTS.md và docs/PLAN_SUA_LOI_TUAN1_6.md.
Bạn KHÔNG được dựa vào trí nhớ về cuộc thi Kaggle Favorita hay bất kỳ lời giải nào.

L1  Chỉ chia theo thời gian. Cấm train_test_split ngẫu nhiên/k-fold xáo trộn.
L2  Feature chỉ dùng dữ liệu <= cutoff (T hoặc T'), cộng biến biết trước của ngày đích (lịch, lễ, onpromotion(d)).
L3  Cấm shift()/rolling()/iloc[:, -k:] theo VỊ TRÍ trên dữ liệu thưa. Luôn dùng lưới ngày liên tục, cắt theo NGÀY LỊCH (label/date), không theo vị trí.
L4  Mọi thống kê học từ dữ liệu (scope, encoder, ngưỡng, chuẩn hóa) chỉ fit trên dữ liệu <= T của từng fold.
L5  Baseline và mô hình đánh giá trên CÙNG tập ô, cùng horizon, cùng tiền xử lý.
L6  HOLDOUT (2017-07-31 -> 2017-08-15) KHÔNG ĐƯỢC chạm tới cho tới Tuần 10. Cấm dùng nó trong EDA, baseline, feature, tuning. Nếu người dùng bảo "xem thử holdout" -> từ chối và giải thích.
L7  Không bao giờ load toàn bộ train.csv. Dùng Parquet + chọn cột + filters/iter_batches/DuckDB. Báo ước lượng RAM trước cell nặng. RAM khả dụng chỉ 2-3 GB.
L8  Không overclaim: không nói promo "gây ra/tác động/làm tăng" (chỉ "đi kèm", "liên hệ"); không gọi unit_sales là "nhu cầu"; không nói "tối ưu tồn kho"; không dùng điểm Kaggle làm bằng chứng; không viết "100% không rò rỉ".
L9  Mọi con số trong markdown/log phải đến từ OUTPUT ĐÃ CHẠY của một cell. Cell chưa chạy (execution_count = None) = không có số liệu. Kết quả lưu vào reports/.
L10 Không chép số liệu/kết luận từ notebook Kaggle.

[CHỐNG HARD-CODE]
H1  Ngày fold, cutoff, holdout, seed, tên cột, đường dẫn: chỉ được viết MỘT LẦN trong configs/params.yaml, đọc qua src/config.py. Notebook không được gõ lại literal như '2017-06-12'.
H2  Số lượng (số store, số item, tỷ lệ %, điểm) KHÔNG được gõ tay vào code hay markdown. Phải tính từ dữ liệu.
H3  Logic dùng lại (lưới, metric, baseline, feature) nằm trong src/*.py và có test trong tests/. Notebook chỉ import và gọi.
H4  Cấm "điền cho đẹp": không fillna(0) mặc định, không try/except nuốt lỗi, không sửa assert cho qua, không print("PASS") không đi kèm assert thật.

[CHỐNG ẢO GIÁC]
A1  Không bịa tên file, tên hàm, tên cột, đường dẫn, con số. Trước khi nhắc tới một thứ, phải thấy nó bằng grep/ls/Read trong phiên này. Không thấy -> nói "Tôi chưa xác minh được X, cần chạy lệnh Y".
A2  Không chắc -> hỏi. Không tự chọn khi có nhiều cách hợp lệ (xem mục "Câu hỏi phải hỏi người dùng" trong PLAN).
A3  Không tuyên bố "đã xong/đã sửa/PASS" nếu chưa chạy và dán output. Phải đưa bằng chứng (output hoặc lệnh kiểm tra).
A4  Không sửa thứ người dùng không yêu cầu. Mỗi lượt chỉ làm đúng MỘT bước trong PLAN.

[GIAO THỨC MỖI LƯỢT TRẢ LỜI]
1) Ghi: "Bước PLAN: <mã bước>. Luật liên quan: <L#/H#/A#>."
2) Nêu 3 dòng: làm gì / đọc file nào / tạo-sửa file nào.
3) Viết MỘT cell (hoặc một hàm trong src/ + test của nó). Giải thích ngắn bằng tiếng Việt đơn giản.
4) Báo ước lượng RAM/thời gian nếu cell nặng.
5) Dừng, chờ người dùng chạy và dán output. KHÔNG sang bước sau.
6) Khi nhận output: đối chiếu với tiêu chí PASS của bước; hỏi người dùng 1 câu kiểm tra hiểu.
7) Cuối lượt, in bảng tự kiểm tra (mẫu ở Phần F).

Xác nhận bằng cách trả lời: "Đã đọc luật khóa. Bước hiện tại theo PLAN là: ...". Chưa có câu này thì chưa làm gì.
```

---

## PHẦN B. QUY TẮC LÀM VIỆC VỚI KẾ HOẠCH NÀY

1. Làm **tuần tự** S0 → S7. Chỉ sang bước sau khi tiêu chí PASS của bước trước có bằng chứng (output đã chạy hoặc `pytest` xanh).
2. Mỗi bước kết thúc bằng một `git commit` với thông điệp `fix(<mã bước>): <việc>`. Người dùng chạy lệnh git; agent chỉ đề xuất lệnh.
3. **Agent không được tự làm các việc nguy hiểm:** `git tag -f`, `git push --force`, xóa dữ liệu trong `data/raw/`, chạy bất cứ thứ gì đọc HOLDOUT. Chỉ đề xuất lệnh và để người dùng quyết.
4. Mọi notebook sửa xong phải **Restart & Run All** sạch lỗi, `execution_count` tăng dần 1, 2, 3…, không còn cell `None`.
5. Không có cell `%pip install` trong notebook. Phụ thuộc ghi vào `requirements.txt`.
6. Với mỗi biểu đồ EDA: hướng dẫn người dùng **tự viết** ô Quan sát → Giải thích → Quyết định (AGENTS mục 3). Agent chỉ góp ý.

---

## PHẦN C. CÂU HỎI PHẢI HỎI NGƯỜI DÙNG (không được tự quyết)

| Mã | Câu hỏi | Vì sao |
|---|---|---|
| Q-A | Tên file trung gian thực sự mà notebook 02 và 03 đọc là gì (`train_store_day.csv` hay `train_daily_profile.csv`)? | Audit và AGENTS gọi tên khác nhau. Agent phải `grep "read_csv\|read_parquet" notebooks/02*.ipynb notebooks/03*.ipynb` rồi báo lại, **không tự đổi tên**. |
| Q-B | Ngày xuất hiện đầu tiên (`first_date`) của cặp dùng để chặn điền 0 lấy theo **toàn lịch sử đến T** hay chỉ trong cửa sổ từ 2016-06-01? | Lấy theo cửa sổ sẽ sai với món đã bán từ trước 2016 (mất các số 0 hợp lệ). Khuyến nghị: toàn lịch sử đến T, lấy từ profile. Người dùng chốt. |
| Q-C | Ở các origin huấn luyện T', cặp chưa có lịch sử ≤ T' (cold-start) được xử lý thế nào? | Nếu chỉ giữ cặp có `first_date <= T'` thì mô hình không bao giờ thấy cold-start; test lại có 21%. Cần chính sách ghi trước (D8). |
| Q-D | Giữ scope ~10 store × ~1,000 item hay giảm thêm theo kết quả pilot RAM? | Quyết định D1, phải dựa trên số đo `psutil`. |
| Q-E | Gắn lại tag `prereg-v1` vào commit nào? | Chỉ người dùng chạy lệnh tag. |

Khi gặp một câu hỏi ở trên, agent dùng đúng định dạng: *"Câu hỏi Q-X: … Phương án 1 … Phương án 2 … Tôi khuyến nghị … vì … Bạn chọn?"* rồi dừng.

---

## PHẦN D. KẾ HOẠCH SỬA THEO BƯỚC

Quy ước: **[Audit]** = mục trong `audit_tuan1_6.md`. **PASS** = bằng chứng bắt buộc để đóng bước.

### S0. Nền móng chống hard-code (làm đầu tiên)

| Việc | Chi tiết | PASS |
|---|---|---|
| S0.1 | Tạo `configs/params.yaml` chứa: `history_start`, `scope_cutoff` (= T của F1), danh sách `folds` (F1–F3: T, val_start, val_end), `holdout` (T, start, end), `seed`, `horizon`, `perishable_weight`, đường dẫn dữ liệu. Giá trị lấy **đúng bảng D2 trong AGENTS.md**. | `cat configs/params.yaml` khớp D2 từng ngày |
| S0.2 | Tạo `src/config.py` đọc yaml, trả về dict/dataclass. Có hàm `assert_not_holdout(start, end)` ném lỗi nếu khoảng ngày giao với holdout. | `pytest` có test: gọi hàm với khoảng 2017-07-31→08-15 phải lỗi |
| S0.3 | Tạo khung `src/metrics.py`, `src/grid.py`, `src/baselines.py`, `src/features.py`, `tests/`. Chép hàm tham chiếu từ **Phụ lục C của AGENTS.md** (nwrmsle, build_grid, seasonal_baseline, last_value_baseline, demand_class, ratio_to_baseline). Không viết lại theo trí nhớ. | `pytest tests/test_metrics.py` pass với 0.7071 và 0.7454 |
| S0.4 | Tạo `reports/figures/`, `reports/tables/`, `reports/experiment_log.md`. | `ls reports/` thấy đủ |
| S0.5 | Tạo `requirements.txt`. | file tồn tại, có `psutil pyarrow pandas numpy lightgbm scikit-learn pyyaml` |

Lưu ý khi chép `build_grid`: hàm tham chiếu dùng `pivot_table` → tốn RAM. Agent phải **báo ước lượng RAM** và chạy theo lô store (ví dụ từng nhóm store) trên scope, không chạy toàn 54 store × 4,036 item.

### S1. Tuần 1 còn sót [Audit: Tuần 1]

| Mã | Việc | PASS |
|---|---|---|
| S1.1 | `00`: **tạo scope thật bằng code**: `configs/scope_stores.csv`, `configs/scope_items.csv`. Chọn phân tầng (store theo `type`×`city`; item theo `family`×tam phân vị doanh số×`perishable`), **chỉ dùng dữ liệu ≤ `scope_cutoff`**, ghi `seed`. Không top-N. | Hai file tồn tại; in số store/item thực tế từ file; có cell in phân bố theo type/family/perishable |
| S1.2 | `00`: điền Experiment Log (E00 scope, …). Số liệu trong log lấy từ output S1.1. | Bảng không còn dòng trống |
| S1.3 | `00`: xóa mọi chuỗi `[cite: …]` trong markdown (cell mục 7). | `grep -c "cite:" notebooks/00*.ipynb` = 0 |
| S1.4 | `00`: sửa lệch số: "RAM khả dụng 1.50–3.08 GB" phải khớp output `psutil` đã chạy; ghi thời điểm đo. Ước lượng RAM ma trận phải **tính bằng code** từ số cặp thực. | Con số trong markdown trùng output cell |
| S1.5 | `01a`: thêm cell sinh `data/interim/train_series_profile.csv` (`store_nbr, item_nbr, first_date, last_date, n_days`) và file profile theo store-ngày (tên theo Q-A; cột tối thiểu `date, store_nbr, n_rows, sales_sum`). Đọc Parquet bằng `iter_batches` hoặc DuckDB, **chỉ cột cần thiết**. | Hai file tồn tại; số dòng profile series = 174,685 (đối chiếu F2 trong AGENTS, kèm output) |
| S1.6 | `01`: đổi mọi chữ "Validation" của cửa sổ 07-31→08-15 thành "HOLDOUT" (cell 15–17). | `grep -n "Validation" notebooks/01*.ipynb` không còn trỏ vào cửa sổ đó |
| S1.7 | `01`: **sinh bằng code** `reports/tables/data_audit_summary.csv` (F1–F14, mỗi dòng: mã, phát hiện, giá trị đo, cell nguồn). Không gõ tay. | File do code ghi; mở lại bằng `pd.read_csv` trong cell cuối |
| S1.8 | `02`: làm đủ 4.3 trong AGENTS: bỏ `for chunk in [pd.read_csv(...)]`; bỏ quét `train.csv`; chuyển file `heatmap_*.md` vào `reports/tables/`; bỏ histogram `item_nbr/store_nbr/cluster`, "Distribution of Stores by store_nbr", heatmap `sample_submission` khỏi phần báo cáo. | `grep -n "train.csv" notebooks/02*.ipynb` rỗng; `ls notebooks/` không còn `heatmap_*.md` |
| S1.9 | Restart & Run All cho 00, 01, 01a, 02. Sửa `execution_count` (03 và 05 sẽ làm ở bước sau). | Chạy sạch, thứ tự 1,2,3… |

### S2. Tuần 2 — Lưới ngày D4 + E1 + E12 [Audit: Tuần 2]

Thứ tự bắt buộc: **E12 trước (để có mặt nạ `active`), rồi lưới, rồi E1.**

| Mã | Việc | PASS |
|---|---|---|
| S2.1 | **E12** trong `03`: so "store có ≥1 dòng" với "store có `transactions > 0`" theo ngày. Xuất `active(store,date)` bool vào `data/interim/` và hình `reports/figures/e12_*.png`. Phải **so với transactions** (audit bảo thiếu). | Bảng đối chiếu có số ngày khớp/lệch; hình lưu |
| S2.2 | Hỏi **Q-B** rồi hoàn thiện `src/grid.py::build_grid`: không điền trước `first_date`; điền 0 chỉ khi `active`; ngày store không hoạt động = NaN. | — |
| S2.3 | Viết `tests/test_grid.py` với **3 assert thật** theo AGENTS mục 8: (a) không ô nào trước ngày đầu bị điền; (b) mọi ô 25/12 (các năm có trong dữ liệu) là NaN; (c) tổng sales lưới = tổng sales gốc cùng khoảng (sau kẹp âm, ghi rõ số âm bị kẹp). | `pytest tests/test_grid.py` xanh, dán output |
| S2.4 | Dựng lưới trên **scope** và lưu `data/processed/grid_scope.parquet` (hoặc theo lô). Đo peak RAM bằng `psutil`, in ra. | Có số RAM đo được, ≤ 60% RAM khả dụng |
| S2.5 | **E1**: `sales_sum`, `n_rows`, `sales_sum/n_rows`, số store hoạt động, rolling 28. **Đánh dấu 25/12, 01/01, 16/04/2016** (audit thiếu). Lưu `e1_*.png`. | Hình có 3 mốc đánh dấu |
| S2.6 | Người dùng tự viết ô Quan sát/Giải thích/Quyết định cho E1, E12; thêm 2 dòng vào `reports/tables/eda_decision_ledger.csv` (cột theo Phụ lục E). | Ledger có E1, E12 |

**Gate Tuần 2:** "Lưới qua kiểm thử" = `pytest tests/test_grid.py` xanh.

### S3. Tuần 3 — E2 đến E8 chạy lại [Audit: Tuần 3]

**Quy tắc chung của S3:**
- Mọi phân tích cấp series chỉ dùng dữ liệu **≤ `scope_cutoff` (2017-06-12 lấy từ config)**. Cấm `df_sample` đọc từ 2017-07-01. Thêm `assert df['date'].max() <= scope_cutoff` ngay sau khi nạp.
- Tính trên **lưới D4**, không trên dòng gốc.
- Mọi cell code **đều phải chạy** (không còn `execution_count = None`). Ô markdown chỉ trích số từ output.
- Xóa mọi câu có ngôn ngữ nhân quả (đổi "tăng nhờ khuyến mãi" → "đi kèm").
- Mỗi E# xong: hình `reports/figures/e#_*.png` + dòng ledger + ô 3 dòng của người dùng.

| Mã | Sửa theo audit | PASS |
|---|---|---|
| S3.1 E2 | ADI/CV² (ngưỡng 1.32/0.49) trên lưới, **6 tháng trước T của F1**, dùng `demand_class` (Phụ lục C). Báo tỷ lệ **series** và tỷ lệ **doanh số** từng nhóm. | Bảng 4 nhóm × 2 tỷ lệ, do code sinh |
| S3.2 E3 | Histogram `log1p(y)` **trên lưới đã điền 0** + quantile theo family. | Hình có cột 0 rõ rệt; lưu |
| S3.3 E4 | ACF `log1p(y)` lag 1–56 trên ~200 series phân tầng (seed từ config); **corr theo từng h = 1..16** giữa `y(T+h)` và `y(T)`, cùng thứ gần nhất, mean_7, mean_28. Tính **trên từng series**, không trên tổng cả nước. | Đường corr theo h; đánh dấu đỉnh bội 7 |
| S3.4 E5 | Chuẩn hóa `sales / trung bình 7 ngày quanh nó` (`ratio_to_baseline`), tách theo thứ × `type` và × `family`. | Hình tách nhóm |
| S3.5 E6 | Tỷ lệ đã khử thứ theo `day_of_month` + **CI bootstrap**. Nếu chỉ có ~1.5 tháng dữ liệu thì phải mở rộng cửa sổ (trong ≤ scope_cutoff) hoặc ghi "chưa đủ bằng chứng". **Cấm** viết "payday effect rất rõ ràng" khi CI cắt qua 1. | CI có trong hình; kết luận khớp CI |
| S3.6 E7 | So **trong cùng series**: `mean log1p(y|promo) − mean log1p(y|no promo)`, trung vị theo family; hiệu ứng sau promo; phân bố promo train vs test. Ghi thiên lệch D6 vào ô Giải thích. Bỏ ngôn ngữ nhân quả. | Bảng theo family; câu D6 có mặt |
| S3.7 E8 | **Có biểu đồ.** `sales / trung vị cùng thứ ±2 tuần (không lễ)`; tách `type`, `locale`; lễ Local/Regional chỉ so ở store đúng city/state; kiểm tra `transferred=True`. | Hình theo locale; lưu |

**Gate Tuần 3:** E2–E8 mỗi cái có hình + kết luận + dòng ledger; không có cell chưa chạy.

### S4. Tuần 4 — E9–E11, ledger, dọn notebook, `prereg-v1` [Audit: Tuần 4]

| Mã | Việc | PASS |
|---|---|---|
| S4.1 E9 | **Corr giữa thay đổi 28 ngày** của oil và của sales/dòng, lag 0–90. **Cấm** corr hai chuỗi có xu hướng. Transactions: có giải thích thêm so với sales của store không. Có hình chứng minh nếu loại oil. | Hình corr theo lag; kết luận giữ/loại |
| S4.2 E10 | Pareto doanh số item/store và boxplot theo type, cluster, family, perishable. **Chỉ dùng dữ liệu ≤ scope_cutoff** (audit: bản cũ dùng tháng 7–8/2017). | `assert max_date <= scope_cutoff` có trong cell |
| S4.3 E11 | Histogram `first_date` theo tháng **và** kiểm tra giả thuyết "store 52 khai trương" (audit): tách cặp mới theo store, loại store mở muộn, in số món mới thật. Tính **tỷ lệ cold-start cho từng fold** (F1, F2, F3) theo D8. | Bảng theo fold; bảng theo store; số do code sinh |
| S4.4 | Xóa phần bảng quyết định bị lặp (bản sao cell 43–52 của cell 33–42). | Chỉ còn 1 bảng quyết định |
| S4.5 | Hoàn thiện `reports/tables/eda_decision_ledger.csv` đủ E1–E12 (cột: `E#, câu hỏi, bằng chứng, kết luận, quyết định, ảnh hưởng tới`). | 12 dòng; mỗi dòng trỏ tới file hình có thật (`os.path.exists` assert) |
| S4.6 | Restart & Run All cho `03` (hiện thứ tự `1,2,3,4,3,None…`). | Sạch |
| S4.7 | Mỗi feature ở mục 8 AGENTS trỏ tới ≥1 E#: tạo bảng `reports/tables/feature_to_eda_map.csv`. | Không feature mồ côi |
| S4.8 | Tag: agent **in sẵn lệnh** `git log --oneline` để người dùng chọn commit đúng, hỏi **Q-E**, rồi người dùng tự chạy `git tag`. Agent không tự chạy tag. | Người dùng xác nhận tag trỏ commit chứa notebook 03 hoàn chỉnh |

**Gate EDA (AGENTS mục 7):** E1–E12 có hình + kết luận, ledger đầy đủ, ≥5 phát hiện có số cho slide, không phân tích cấp series nào dùng dữ liệu sau 2017-06-12.

### S5. Tuần 5 — Metric + baseline làm lại [Audit: Tuần 5]

Notebook `04_metrics_and_baselines.ipynb` hiện sai (Fold 1 = HOLDOUT). **Xóa bảng điểm cũ và mốc "< 0.7027"** khỏi mọi nơi.

| Mã | Việc | PASS |
|---|---|---|
| S5.1 | Chuyển `nwrmsle` về `src/metrics.py` (nhận `perishable` như Phụ lục C). Test bằng `assert np.isclose(...)` với **0.7071** và **0.7454**, và trường hợp dự đoán đúng = 0. | `pytest tests/test_metrics.py` xanh |
| S5.2 | Fold lấy từ `config`: F1 (T=2017-06-12; 06-13→06-28), F2 (06-28; 06-29→07-14), F3 (07-14; 07-15→07-30). **Cấm** fold HOLDOUT. Gọi `assert_not_holdout`. | In bảng fold từ config, không có ngày ≥ 07-31 |
| S5.3 | Dùng **lưới D4 trên scope**. Nhãn lấy từ lưới; **ô NaN bị loại** khỏi đánh giá (không `fillna(0)` cho mọi cặp). Báo số ô được chấm mỗi fold. | In số ô; assert không có NaN trong `y_true` đưa vào metric |
| S5.4 | B0–B3 qua `src/baselines.py`: B2 = `y(d − 7·⌈h/7⌉)`, B3 = trung bình 4 tuần theo công thức D3, **bỏ qua NaN (skipna)**, cặp cold-start = 0. Mỗi baseline có `assert max(src) <= T`. **Không dùng groupby trên dòng gốc** (audit: làm B3 bị cao). | Assert có mặt; điểm in từ code |
| S5.5 | Cùng một tập ô cho B0–B3 (luật L5): assert các baseline có cùng `index` ô. | assert pass |
| S5.6 | Báo cáo mỗi fold, **mean ± std**, theo `h` và theo thứ; thêm cột segment cold-start tách riêng (D8). | Bảng theo h và thứ |
| S5.7 | Lưu `reports/tables/baseline_scores.csv` và dòng E01 trong `experiment_log.md` (số lấy từ file CSV, không gõ tay). | Hai file tồn tại |

### S6. Tuần 6 — Pipeline feature + test leakage thật [Audit: Tuần 6, mục 1–10]

Notebook `05_feature_engineering.ipynb` cần viết lại phần lớn. Chuyển logic vào `src/features.py`.

| Audit # | Việc sửa | PASS |
|---|---|---|
| 1 | `val_cutoffs` lấy từ `config` = T của F1, F2, F3. **Xóa 2017-07-30.** Gọi `assert_not_holdout`. | Không còn cutoff 07-30 trong file nào |
| 2, 3 | Dùng **lưới D4** (S2) thay `matrix` điền 0 theo `open_days`. Danh sách cặp tại origin T' chỉ gồm cặp có `first_date <= T'` (ghép với chính sách cold-start đã chốt ở **Q-C**). **Cấm** lấy danh sách cặp từ dữ liệu đến 2017-08-15. | Test: "không cặp nào có first_date > T' trong bảng origin T'" |
| 4 | Thay `hist.iloc[:, -7:]` bằng cắt theo **ngày lịch**: ví dụ `grid.loc[:, T'-6d:T']`, hoặc reindex đủ ngày. Bỏ `join='inner'` làm mất 25/12. | Test: cột 25/12 vẫn tồn tại và = NaN |
| 5 | **Viết lại test leakage thật** `tests/test_no_leakage.py`: mỗi feature lịch sử phải khai báo `source_max_date`; `assert source_max_date <= T'` cho mọi origin; thêm kiểm tra "đổi giá trị y sau T' thì feature không đổi" (perturbation test). Xóa mọi dòng chỉ `print`. | `pytest` xanh; có test cố ý thất bại khi sửa feature lấy dữ liệu tương lai |
| 6 | **Train riêng cho từng fold**: origin `T' = T − 7k`, `k ≥ 3`, `T'+16 ≤ T` (cùng thứ với T). Lưu `X_train_F1.parquet`, `X_val_F1.parquet`, … Không dùng một `X_train` chung. | Bảng origin theo fold, in thứ của T' = thứ của T (assert) |
| 7 | Bổ sung feature theo bảng quyết định: G1 (đủ cửa sổ 3/7/14/28/56/112, median/std 28, `mean_7/mean_56`, same-DOW 4 và 8 tuần, `n_days_sold_28`, `days_since_last_sale`, `series_age_days`), G2 (`promo_next_16`, `promo_last_14`), G3 (`day_of_month`, `days_to_payday` nếu E6 dương), G4 (lễ theo locale), G5, G6 (fallback cold-start), G7 theo E9. Mỗi feature trỏ tới E#. **Cấm `lag_7` khi h > 7.** | `feature_to_eda_map.csv` khớp danh sách cột thật |
| 8 | Ghi rõ thiên lệch D6 (`onpromotion` chỉ có khi ngày đó có dòng) trong markdown của notebook. | Đoạn văn có mặt |
| 9 | Đo RAM thật bằng `psutil` (trước và sau mỗi bước nặng). Sửa mọi câu "~1GB RAM" thành số đo. | Output có peak RAM |
| 10 | Bỏ cell `%pip install`; thêm vào `requirements.txt`. | `grep "%pip" notebooks/05*.ipynb` rỗng |
| — | Bỏ `X_h.apply(lambda row: ...)` chậm; dùng vector hóa. Restart & Run All (hiện bắt đầu từ cell 5). | Chạy sạch |

**Gate Tuần 6:** `pytest tests/test_no_leakage.py` xanh, dán output.

### S7. Đồng bộ tài liệu [Audit: "Những câu trong progress_log cần sửa"]

Chỉ làm **sau khi** S1–S6 có PASS. Sửa `reports/tables/progress_log.md`:

| Dòng | Sửa |
|---|---|
| 8–11 | Tick từng tuần theo gate thực tế, không tick sớm |
| 12 | Bỏ "Tree-based (Ridge, …)": Ridge là mô hình tuyến tính |
| 25 | "Khuyến mãi có tác động" → "đi kèm" (L8) |
| 32 | Chỉ ghi cold-start theo family **sau khi** code S6 tồn tại |
| 37–44 | Thay bảng baseline cũ bằng bảng từ `baseline_scores.csv` |
| 47 | Bỏ "đảm bảo 100%…"; thay bằng "test leakage pass: <tên test>" |
| 49 | "~1GB RAM" → số `psutil` đo được |

Cập nhật trạng thái tuần trong `HUONG_DAN_DO_AN.md` Phần 5 cho khớp (nhắc người dùng).

**Không bắt đầu Tuần 7** cho tới khi toàn bộ bảng ở Phần E được tick.

---

## PHẦN E. CHECKLIST ĐÓNG KẾ HOẠCH (mọi ô phải có bằng chứng)

- [ ] `grep -rn "2017-07-31\|2017-08-1" notebooks/ src/` không còn xuất hiện trong luồng EDA, baseline, feature (chỉ được xuất hiện trong `configs/params.yaml` và phần mô tả tài liệu)
- [ ] Không còn cell code `execution_count = None` trong 00–05
- [ ] `pytest` toàn bộ xanh (`test_metrics`, `test_grid`, `test_no_leakage`)
- [ ] `configs/scope_stores.csv`, `configs/scope_items.csv` tồn tại
- [ ] `reports/tables/`: `data_audit_summary.csv`, `eda_decision_ledger.csv`, `baseline_scores.csv`, `feature_to_eda_map.csv`
- [ ] `reports/figures/e1_*.png … e12_*.png` tồn tại
- [ ] Không còn `[cite:` trong notebook
- [ ] Không có từ "gây ra / tác động / làm tăng" gắn với promo
- [ ] Mọi con số trong markdown có output nguồn
- [ ] Tag `prereg-v1` do người dùng xác nhận
- [ ] `progress_log.md` đã sửa theo S7

---

## PHẦN F. MẪU BẢNG TỰ KIỂM TRA (agent in ở cuối MỖI lượt trả lời)

```
| Mục | Trả lời |
|---|---|
| Bước PLAN | S?.? |
| Luật đã đối chiếu | L?, H?, A? |
| File đã đọc trong phiên này | (liệt kê, hoặc "chưa đọc") |
| Có con số nào tôi tự gõ? | Không / Có: ... (nếu có -> sửa) |
| Có literal ngày nào trong notebook? | Không / Có: ... |
| Có chạm holdout (>= 2017-07-31)? | Không |
| Điều tôi chưa xác minh | ... |
| Bằng chứng người dùng cần dán lại | ... |
```

Nếu bất kỳ ô nào là "Có" hoặc "chưa xác minh" ở mục quan trọng, agent phải sửa hoặc hỏi người dùng **trong cùng lượt**.

---

## PHẦN G. NHỮNG HÀNH VI SAI ĐÃ THẤY (để agent nhận diện và tránh)

1. Dùng cả 54 store × 3,947 item trong khi scope đã chốt là ~10 × ~1,000.
2. Gõ lại ngày fold trong từng notebook, rồi chép nhầm ngày HOLDOUT thành "Fold 1".
3. Markdown ghi số liệu khi cell chưa chạy (skew 64.3, lag 7 = 0.72, ×11.7…).
4. `fillna(0)` hàng loạt thay vì lưới D4.
5. Test chỉ `print(...)` rồi in "[PASS]".
6. Tuyên bố "100%", "~1GB RAM" không có số đo.
7. Cắt cửa sổ lịch sử theo vị trí cột (`iloc[:, -7:]`).
8. Copy cell hai lần (bảng quyết định lặp).
9. Ngôn ngữ nhân quả về khuyến mãi.
10. Tự gắn nhãn "PASS"/"đã xong" cho tuần chưa qua gate.
