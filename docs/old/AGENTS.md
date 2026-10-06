# AGENTS.md — Hướng dẫn cho AI agent hỗ trợ đồ án Favorita

> File này dành cho AI coding agent (Claude Code, Codex, Cursor, Copilot…). Nếu dùng Claude Code, copy hoặc đổi tên thành `CLAUDE.md` để agent tự đọc.
> Người dùng đọc `HUONG_DAN_DO_AN.md` (bản đơn giản). Hai file phải **thống nhất**; nếu thay đổi quyết định ở đây, nhắc người dùng cập nhật file kia.
> Cập nhật lần cuối: 01/10/2026.

---

## 1. Bối cảnh

- **Đồ án môn Machine Learning**, báo cáo khoảng cuối 12/2026 đến đầu 01/2027.
- **Đề tài:** Dự báo `unit_sales` 16 ngày tới cho từng `store_nbr × item_nbr × date` trên dữ liệu thật Corporación Favorita (Kaggle, Ecuador, 01/2013 → 08/2017).
- **Giảng viên** thích dữ liệu thật và **chấm rất kỹ phần phân tích dữ liệu dẫn tới quyết định** xây feature và chọn mô hình. Mọi feature/mô hình phải truy được về một phân tích EDA (mã E#) hoặc quyết định (D#).
- **Người dùng (sinh viên):** mới học ML, tiếp thu chậm, lần đầu làm với dữ liệu lớn. Xem mục 3 về cách làm việc với người dùng.
- **Máy người dùng:** Linux, 8 CPU logic, RAM 7.5 GB nhưng **chỉ 2–3 GB khả dụng**, không GPU. Python 3.11. Đường dẫn gốc dự án trên máy người dùng: `.../Grocery-sales-forecast/`, notebook nằm trong `notebooks/`, dữ liệu trong `data/raw/`.

### Dữ liệu

| File | Kích thước | Ghi chú |
|---|---|---|
| `train.csv` | 4,766 MB, 125,497,040 dòng | `id, date, store_nbr, item_nbr, unit_sales, onpromotion` |
| `test.csv` | 3,370,464 dòng | 54 store × 3,901 item × 16 ngày (2017-08-16 → 08-31), **không có nhãn** |
| `items.csv` | 4,100 dòng | `family` (33), `class` (337), `perishable` (24% là 1) |
| `stores.csv` | 54 dòng | `city` (22), `state` (16), `type` A–E, `cluster` 1–17 |
| `transactions.csv` | 83,488 dòng | Dừng 2017-08-15 |
| `oil.csv` | 1,218 dòng | Chỉ ngày thường, 43 NaN, phủ tới 2017-08-31 |
| `holidays_events.csv` | 350 dòng | `type`, `locale` (National/Regional/Local), `transferred` |
| `sample_submission.csv` | 3,370,464 dòng | Định dạng nộp Kaggle |

---

## 2. Luật bắt buộc (không vi phạm, kể cả khi người dùng yêu cầu — khi đó giải thích lý do)

1. **Không** `train_test_split` ngẫu nhiên hay k-fold xáo trộn. Chỉ chia theo thời gian.
2. Feature chỉ dùng dữ liệu ≤ ngày cắt (cutoff/origin), cộng biến **biết trước** của ngày đích: lịch, ngày lễ, `onpromotion(d)`.
3. Không `shift()`/`rolling()` theo vị trí dòng trên dữ liệu thưa. Luôn dựng lưới ngày liên tục (ma trận cặp × ngày) rồi mới tính.
4. Mọi thống kê học từ dữ liệu (encoder, target encoding, chọn scope, ngưỡng phân nhóm, chuẩn hóa) chỉ fit trên phần train của từng fold.
5. Baseline và mô hình phải đánh giá trên **cùng tập ô**, cùng horizon, cùng tiền xử lý.
6. **HOLDOUT (2017-07-31 → 2017-08-15) chỉ chạy một lần** sau khi khóa thiết kế. Không dùng để chọn feature, tune, early stopping hay so mô hình. Nếu người dùng muốn "xem thử holdout", từ chối và giải thích.
7. **Không bao giờ load toàn bộ `train.csv` vào pandas.** Dùng Parquet + đọc cột cần thiết, `pyarrow` `iter_batches`, Polars lazy hoặc DuckDB. Ước lượng RAM trước khi chạy cell nặng; mục tiêu peak ≤ 60% RAM khả dụng.
8. Không overclaim trong code comment, markdown hay báo cáo: không nói promo "gây ra" tăng bán (chỉ là liên hệ); không gọi `unit_sales` là "nhu cầu"; không nói "tối ưu tồn kho"; không dùng điểm Kaggle leaderboard làm bằng chứng.
9. Không gõ tay số liệu vào báo cáo/markdown kết luận nếu có thể sinh từ code; kết quả phải lưu vào `reports/`.
10. Không chép số liệu, kết luận từ `demand-forecasting-favorita-grocery-sales.ipynb` (notebook Kaggle) vào đồ án. Chỉ tham khảo ý tưởng (Phụ lục D).

---

## 3. Cách làm việc với người dùng

Người dùng phải **tự hiểu và tự trình bày được** mọi thứ trước giảng viên. Agent là người kèm, không phải người làm thay.

- Trả lời bằng **tiếng Việt đơn giản**, câu ngắn. Thuật ngữ tiếng Anh giữ nguyên nhưng giải thích lần đầu xuất hiện (ví dụ: "leakage — rò rỉ thông tin tương lai vào lúc huấn luyện").
- **Mỗi lần một bước nhỏ.** Viết một cell, giải thích nó làm gì, để người dùng chạy và đọc output, rồi mới sang cell tiếp. Không sinh cả notebook 30 cell một lúc.
- Code ngắn, dễ đọc, có comment tiếng Việt ở các dòng quan trọng. Ưu tiên pandas/numpy rõ ràng hơn là one-liner khó hiểu. Đặt hàm dùng lại vào `src/`.
- Sau mỗi kết quả, **hỏi lại người dùng một câu kiểm tra hiểu** (ví dụ: "Vì sao ở đây mình lấy ngày 25/12 là NaN chứ không phải 0?"). Nếu trả lời sai, giải thích lại bằng ví dụ khác.
- Với mỗi biểu đồ EDA, hướng dẫn người dùng **tự viết** ô markdown 3 dòng: **Quan sát** (có số) → **Giải thích/giả thuyết** → **Quyết định** (D# hoặc feature). Agent góp ý, sửa câu, không viết thay toàn bộ.
- Khi người dùng đề xuất điều vi phạm mục 2, không làm im lặng; giải thích vì sao sai và đưa phương án đúng.
- Trước cell tốn RAM/thời gian, báo trước ước lượng ("cell này đọc ~X triệu dòng, khoảng Y phút, Z GB RAM").
- Khi người dùng hỏi "làm gì tiếp", trả lời theo kế hoạch tuần (mục 9) và Gate hiện tại.
- Nhắc ghi experiment log (Phụ lục B) mỗi khi có thay đổi thiết kế hoặc kết quả mới.

---

## 4. Hiện trạng (01/10/2026)

| Notebook | Trạng thái | Việc còn lại |
|---|---|---|
| `00_scope_and_preregistration.ipynb` | ~80% | Xem 4.1 |
| `01_data_checking.ipynb` | ~95%, 11/11 kiểm tra PASS | Xem 4.2 (có lỗi làm sai số liệu) |
| `02_eda_raw_tables.ipynb` | ~40% | EDA mô tả từng bảng; phần "quan hệ" có 6 biểu đồ **chưa có kết luận**. Xem 4.3 |
| `03`–`09` | Chưa có | |

### 4.1 Sửa `00_scope_and_preregistration`

- [ ] Scope Decision đang ghi **"FULL DATA"**: sai với RAM hiện có. Đổi theo D1; điền các dòng "..." (RAM ước tính, thời gian pilot).
- [x] RAM khả dụng: output in 3.08 GB, markdown ghi 2.10 GB. Ghi cả hai kèm thời điểm, thiết kế theo số thấp.
- [x] Mục 3: thêm cột "thứ của cutoff"; ghi chú F13.
- [x] Mục 12: bổ sung D5, D6 (promo của ô điền 0), D7.
- [ ] Thêm mục "Feature groups dự kiến và E# biện minh" (bảng mục 7) để ablation được đăng ký trước.
- [ ] Freeze: `git init`, commit, tag `prereg-v1` (repo chưa có git).

### 4.2 Sửa `01_data_checking`

- [x] **Lỗi cell 16 (CELL 16 — Validation window & coverage):** `n_stores`/`n_items` gộp giữa các chunk bằng `max()` thay vì hợp tập → ngày vắt qua 2 chunk bị đếm thiếu (in ra 39 store ngày 2017-08-01, 42 store ngày 2017-08-11). Thực tế cả hai ngày có 54 store (cell 06 và `transactions.csv` xác nhận). Sửa bằng `set` theo ngày. (Đã sửa)
- [ ] **Thiếu bước tạo dữ liệu trung gian:** notebook đọc `data/processed/train.parquet`, notebook 02 đọc `data/interim/train_series_profile.csv`, nhưng không notebook nào tạo ra. Tạo `01a_convert_to_parquet.ipynb` (CSV → Parquet theo chunk, dtype gọn) và cell lưu `train_series_profile.csv` (`store_nbr, item_nbr, first_date, last_date, n_days`).
- [ ] `execution_count` không theo thứ tự → Restart & Run All.
- [ ] Đổi tên "validation" của cửa sổ 07-31 → 08-15 thành "HOLDOUT".
- [ ] Lưu bảng F1–F14 ra `reports/tables/data_audit_summary.csv`.

### 4.3 Sửa `02_eda_raw_tables`

- [ ] Không đưa vào báo cáo: histogram `item_nbr`, `store_nbr`, `cluster` (mã định danh), "Distribution of Stores by store_nbr", heatmap chất lượng `sample_submission`.
- [ ] Bảng nhỏ đang đọc kiểu `for chunk in [pd.read_csv(...)]`: đọc thẳng.
- [ ] File `heatmap_*.md` đang ghi vào thư mục notebook → chuyển vào `reports/tables/`.
- [ ] Không quét `train.csv` thêm nữa; dùng Parquet hoặc profile.
- [ ] Phần "quan hệ" chuyển sang `03_eda_for_decisions.ipynb` và sửa:

| Biểu đồ hiện có | Vấn đề | Sửa |
|---|---|---|
| Tổng sales theo thời gian | Nhiễu bởi số store/item tăng (F14) | Thêm sales/dòng và số series hoạt động; đánh dấu 25/12, 01/01, 16/04/2016 (E1) |
| Tỷ lệ âm | Thiếu kết luận | Thêm kết luận → D5 |
| "Promo effect" | Chỉ đếm dòng promo, không đo hiệu ứng | Giữ làm bằng chứng F7; làm E7 |
| Boxplot theo thứ | Trộn các năm có mức khác nhau | Chuẩn hóa theo tuần (E5) |
| Heatmap store × family | Thiếu kết luận | Thêm Pareto (E10) |
| Boxplot lễ vs ngày thường | Nhiễu bởi mùa (lễ dồn vào tháng 12) và xu hướng | So với đường nền cùng thứ ±2 tuần, tách `type`/`locale` (E8) |

---

## 5. Phát hiện đã xác minh (F1–F14)

| # | Phát hiện | Hệ quả |
|---|---|---|
| F1 | 125.5M dòng, 4.7 GB; RAM khả dụng 2–3 GB | Phải giới hạn scope (D1) |
| F2 | Train 54 store, 4,036 item, 174,685 series; test lưới đầy đủ 210,654 cặp × 16 | D4 |
| F3 | **0 dòng** `unit_sales == 0`; 48–52% cặp test có dòng mỗi ngày ở 16 ngày cuối | Dòng thiếu cần quy tắc điền 0 (D4) |
| F4 | Thiếu 25/12 (2013–2016); 01/01 gần như không có dòng (1–2 store) | Đóng cửa → mặt nạ, không điền 0 |
| F5 | Store 20, 21, 22, 29, 42, 52, 53 mở muộn (52 mở ~04/2017); store 12, 18, 24, 25 có tháng trống | Mặt nạ store hoạt động; cold-start cấp store |
| F6 | 7,795 dòng âm (0.0062%), min −15,372, max 89,440, p99.5 ≈ 100 | Kẹp âm, log1p (D5) |
| F7 | `onpromotion` NaN 17.26%, NaN hoàn toàn tới khoảng 04/2014; promo tăng mạnh từ 2016 | Cửa sổ sau 04/2014 (D6). **Cần in ngày chính xác** |
| F8 | 60 item chỉ có trong test; 44,441 cặp test (21.10%) không có lịch sử | Cold-start (D8) |
| F9 | `transactions` dừng 2017-08-15 | Chỉ dùng ≤ T (D7) |
| F10 | `oil` chỉ ngày thường, 43 NaN, có tới 2017-08-31 | Sau T là tương lai (D7) |
| F11 | Holidays: 38 ngày nhiều sự kiện; có Transfer/Bridge/Additional/Work Day | Join theo locale (D7) |
| F12 | Động đất 2016-04-16 (`Terremoto Manabi` + các ngày sau) | Cờ hoặc loại khỏi cửa sổ (D1) |
| F13 | Holdout có lễ quốc gia chuyển ngày 2017-08-10 → 08-11; F1–F3 chỉ có lễ địa phương (06-23, 06-25, 07-03, 07-23..25 Guayaquil) | Holdout khó hơn ở khía cạnh lễ; ghi trước |
| F14 | Tổng sales tăng 2013→2017 song song với số dòng/ngày (~40k → ~105k) | Chuẩn hóa khi phân tích xu hướng |

---

## 6. Quyết định thiết kế D1–D9

Trạng thái: **[khuyến nghị]** = chờ người dùng chốt và ghi vào pre-registration; **[đã chốt]** = đã có trong notebook 00.

### D1. Scope [khuyến nghị]

- Lịch sử từ **2016-06-01** (sau động đất, đủ ≥140 ngày trước origin huấn luyện đầu tiên 2017).
- Store phân tầng theo `type` và `city` (~10 store). Item phân tầng theo `family × tam phân vị doanh số × perishable` (~1,000 item). **Không top-N** (mất hàng bán chậm, hỏng RQ3).
- Chọn scope chỉ bằng dữ liệu ≤ 2017-06-12. Lưu `configs/scope_stores.csv`, `configs/scope_items.csv` + seed.
- Ước lượng: `số dòng nhãn ≈ số_cặp × số_origin × 16`; `RAM ≈ số dòng × số_feature × 4 byte`. 10,000 cặp × 20 origin × 16 × 40 feature ≈ 0.5 GB. Mục tiêu ma trận ≤ 0.5–0.7 GB.
- Pilot 1–2 store trước, đo peak RAM bằng `psutil`.

### D2. Fold và holdout [đã chốt]

| Cửa sổ | Train đến hết (T) | Đánh giá | Thứ của T |
|---|---|---|---|
| F1 | 2017-06-12 | 06-13 → 06-28 | Thứ Hai |
| F2 | 2017-06-28 | 06-29 → 07-14 | Thứ Tư |
| F3 | 2017-07-14 | 07-15 → 07-30 | Thứ Sáu |
| HOLDOUT | 2017-07-30 | 07-31 → 08-15 | Chủ Nhật |
| Kaggle test (tùy chọn) | 2017-08-15 | 08-16 → 08-31 | Thứ Ba |

Expanding window. Báo cáo từng fold + mean ± std, thêm metric theo `h` và theo thứ.

### D3. Neo theo cutoff [đã chốt baseline; khuyến nghị thiết kế ML]

`T` = ngày cuối đã quan sát, `d` = ngày đích, `h = d − T ∈ 1..16`.

| Baseline | Công thức |
|---|---|
| B0 | `ŷ = 0` |
| B1 | `ŷ = y(T)` (giá trị gần nhất đã quan sát) |
| B2 | `ŷ = y(d − 7·⌈h/7⌉)` |
| B3 | `ŷ = mean_{k=0..3} y(d − 7·(⌈h/7⌉ + k))` |

Thiết kế ML "direct, neo origin":
- Mẫu = `(origin T', store, item)`; nhãn `y(T'+1..T'+16)`.
- Feature lịch sử chỉ ≤ `T'`; feature ngày đích chỉ là biến biết trước.
- Origin huấn luyện `T' = T − 7k`, `k ≥ 3` (cùng thứ với T; `T'+16 ≤ T`).
- Chọn (a) một mô hình có feature `h`, hoặc (b) 16 mô hình theo `h` dùng chung X (ít RAM hơn). Ghi lý do.
- Cấm `lag_7 = y(d−7)` khi `h > 7`.

### D4. Zero ngầm [đã chốt]

1. Lưới ngày cho mỗi cặp từ **ngày có dòng đầu tiên** tới T; không điền trước đó.
2. Điền 0 chỉ khi store hoạt động hôm đó (mặt nạ E12); store-ngày không hoạt động → NaN, loại khỏi nhãn và đánh giá.
3. Đánh giá chính trên lưới đầy đủ; phụ trên dòng có bản ghi.

### D5. Target [khuyến nghị]

Kẹp âm về 0 (nhãn và lịch sử); train trên `log1p(y)`; dự đoán `expm1`, kẹp ≥ 0; `sample_weight = 1.25` cho perishable; không xóa outlier.

### D6. Promo [đã chốt một phần]

- In ngày cuối còn NaN; cửa sổ D1 nằm sau mốc này.
- Ô được điền 0 không có `onpromotion` gốc → điền `False`, ghi giả định và thiên lệch (ước lượng lift bị phóng đại vì ngày promo không bán được thì mất dòng).
- `onpromotion(d)` dùng được (test cho đủ 16 ngày) — giả định "kế hoạch promo đã biết tại cutoff".
- Không diễn giải nhân quả.

### D7. Bảng phụ [đã chốt]

| Nguồn | Quy tắc |
|---|---|
| Lịch, metadata | Dùng |
| `onpromotion(d)` | Dùng cho ngày đích; lịch sử promo ≤ T |
| Sales > T | Cấm |
| `transactions` | Chỉ lag/rolling ≤ T cấp store, hoặc loại (theo E9) |
| Oil | Chỉ ≤ T, forward-fill từ quá khứ; giữ/loại theo E9 |
| Holidays | National → mọi store; Regional → `state`; Local → `city`; xử lý `transferred` theo E8 |

### D8. Cold-start [đã chốt]

Theo từng fold: cặp không có dòng ≤ T. Baseline cho 0. ML: thống kê item ở store khác → `family × store` → 0, hoặc để mô hình xử lý NaN lịch sử với feature cấp item/family. Báo cáo segment riêng.

### D9. Metric [đã chốt]

NWRMSLE = `sqrt(Σ wᵢ (log1p ŷᵢ − log1p yᵢ)² / Σ wᵢ)`, `w = 1.25` nếu perishable. Kẹp ≥ 0. Phụ: MAE, WAPE. Không MAPE. Có unit test (Phụ lục C).

---

## 7. Spec EDA E1–E12 (notebook `03_eda_for_decisions.ipynb`)

Mỗi E# = câu hỏi → cách tính → nhìn gì → nhánh quyết định (viết nhánh **trước** khi chạy). EDA tổng quan (E1, E5, E6, E8, E9, E12) dùng profile tổng hợp trên toàn dữ liệu; EDA cấp series dùng dữ liệu scope **≤ 2017-06-12**. Mỗi E# xong phải có hình trong `reports/figures/e#_*.png`, dòng trong `reports/tables/eda_decision_ledger.csv`, và ô markdown Quan sát → Giải thích → Quyết định do người dùng viết.

| E# | Câu hỏi | Cách tính | Nhánh quyết định |
|---|---|---|---|
| E1 | Sales tăng do mỗi series bán nhiều hơn hay do thêm store/item? | `sales_sum`, `n_rows`, `sales_sum/n_rows`, số store hoạt động; rolling 28 | Mức/series ổn định gần đây → cửa sổ ngắn (D1); có xu hướng → feature tỷ lệ mean_7/mean_56. Ngày đóng cửa → mặt nạ. Động đất → cờ/loại |
| E2 | Bao nhiêu series gián đoạn? | ADI/CV² (Syntetos–Boylan, ngưỡng 1.32/0.49) trên lưới D4, 6 tháng trước F1; tỷ lệ series và tỷ lệ doanh số mỗi nhóm | Biện minh log1p + RMSLE; nếu đa số intermittent/lumpy → thêm thí nghiệm `tweedie`; nhóm = segment RQ3; feature `n_days_sold_28`, `days_since_last_sale` |
| E3 | Phân phối target? | Histogram `log1p(y)` trên lưới đã điền 0; quantile theo family | D5; chuẩn hóa theo mức series nếu family khác thang |
| E4 | Quá khứ nào dự báo tốt nhất cho từng `h`? | ACF `log1p(y)` lag 1–56 (~200 series phân tầng); corr giữa `y(T+h)` và `y(T)`, cùng thứ gần nhất, mean_7, mean_28 cho h=1..16 | Đỉnh bội 7 → B2/B3 và feature cùng thứ; đường corr theo h → biện minh direct multi-horizon |
| E5 | Hiệu ứng thứ có khác giữa nhóm? | sales / trung bình 7 ngày quanh nó; theo thứ × store type/family | Khác nhau → mô hình cây + `dow_ratio` của series; giống → chỉ `dayofweek` |
| E6 | Có hiệu ứng ngày lương (15 và cuối tháng)? | Tỷ lệ đã khử thứ theo `day_of_month`, CI bootstrap | Có → `days_to_payday`; không → ghi kết quả âm |
| E7 | Promo gắn với mức bán cao hơn bao nhiêu? | Trong cùng series: `mean log1p(y|promo) − mean log1p(y|no promo)`, trung vị theo family; hiệu ứng sau promo; phân bố promo train vs test | `onpromotion(d)`, `promo_next_16`, `promo_last_14`; tương tác promo × family qua mô hình cây. Nêu thiên lệch (D6) |
| E8 | Ngày lễ ảnh hưởng thế nào sau khi khử xu hướng? | sales / trung vị cùng thứ ±2 tuần (không lễ); theo `type`, `locale`; lễ Local/Regional chỉ so ở store đúng city/state; kiểm tra ngày `transferred=True` | Có/không các feature lễ; xử lý transferred. Báo trước: F1–F3 ít lễ quốc gia → nhóm lễ có thể đóng góp ít trong ablation |
| E9 | Oil và transactions có tín hiệu? | Corr giữa **thay đổi** 28 ngày của oil và của sales/dòng, lag 0–90 (không corr hai chuỗi có xu hướng); transactions mean 28 ≤ T có giải thích thêm so với sales store không | Dự kiến loại oil (phải có hình chứng minh); transactions giữ dạng lag hoặc loại |
| E10 | Dị biệt store/item? | Pareto doanh số theo item/store; boxplot mức bán series theo type, cluster, family, perishable | Chọn scope phân tầng (D1); metadata categorical; target encoding trong fold nếu dùng |
| E11 | Vòng đời series, cold-start theo fold? | Histogram `first_date` theo tháng; tỷ lệ cold-start mỗi fold; mức bán 30 ngày đầu | D8; feature `series_age_days`; quy tắc series "đã chết" |
| E12 | Store-ngày không có dòng = đóng cửa? | So "store có ≥1 dòng" với "store có transactions > 0" theo ngày | Định nghĩa mặt nạ `active(store, date)` cho D4 |

**Gate EDA:** E1–E12 có hình + kết luận; ledger đầy đủ; mỗi feature ở mục 8 trỏ tới ≥1 E#; ≥5 phát hiện có số cho slide; không phân tích cấp series nào dùng dữ liệu sau 2017-06-12.

---

## 8. Feature và mô hình

### Nhóm feature (dùng cho ablation RQ2)

| Nhóm | Feature | E# | Leakage |
|---|---|---|---|
| G1 Lịch sử | `last_value`, mean 3/7/14/28/56/112, median/std 28, mean_7/mean_56; trung bình cùng thứ với ngày đích 4 và 8 tuần; `n_days_sold_28`, `days_since_last_sale`, `series_age_days` | E1, E2, E4, E5, E11 | Cửa sổ kết thúc tại T' |
| G2 Promo | `onpromotion(d)`, `promo_next_16`, `promo_last_14` | E7 | `onpromotion(d)` biết trước |
| G3 Lịch | `h`, `dayofweek(d)`, `day_of_month(d)`, `days_to_payday(d)` nếu E6 dương | E5, E6 | |
| G4 Lễ | `is_national_holiday(d)`, `is_local_holiday(d, store)`, `days_to_next_holiday` | E8 | |
| G5 Metadata | `store_nbr`, `type`, `cluster`, `city`, `family`, `class`, `perishable` | E10 | Encoding fit trong fold |
| G6 Tổng hợp cấp cao | mean 28 của item mọi store, của `family × store` | E10, E11 | ≤ T' |
| G7 Ngữ cảnh (tùy E9) | transactions mean 28 của store, thay đổi oil 28 ngày | E9 | ≤ T' |

Ablation: đầy đủ; bỏ G2; bỏ G3+G4; bỏ G5; bỏ G6; bỏ G7; "chỉ G1".

### Mô hình

| Mô hình | Vai trò | Lý do gắn với dữ liệu |
|---|---|---|
| B0–B3 | Mốc | E4: chu kỳ tuần mạnh → B2/B3 là đối thủ khó |
| Ridge (log1p) | ML tuyến tính | Đo phần tín hiệu tuyến tính |
| HistGradientBoosting | Đối chứng cây | NaN/categorical tự nhiên; kiểm kết quả không phụ thuộc thư viện |
| **LightGBM** (L2 trên log1p, weight perishable) | Chính | Tương tác dow × family, promo × family, dị biệt store/item (E5, E7, E10); nhẹ trên CPU |
| LightGBM `tweedie` | Một thí nghiệm | Chỉ khi E2 cho thấy đa số gián đoạn |

Không làm: ARIMA/Prophet từng series (174k series thưa, cold-start), deep learning (không GPU, RAM thấp), Random Forest (chậm, tốn RAM, không thêm góc nhìn).

Tuning: ngân sách cố định (~30 cấu hình), chọn theo mean NWRMSLE 3 fold. Early stopping dùng đoạn cuối **trong train của fold**, không dùng cửa sổ đánh giá.

### Kiểm thử bắt buộc

- `tests/test_metrics.py`: giá trị tham chiếu ở Phụ lục C.
- `tests/test_no_leakage.py`: với mọi origin T', mọi cột ngày dùng để tính feature lịch sử ≤ T'; baseline có `assert max(src) <= T`.
- Lưới D4: không ô nào trước ngày đầu tiên bị điền; mọi ô 25/12 là NaN; tổng sales lưới = tổng sales gốc cùng khoảng.

---

## 9. Kế hoạch tuần và Gate

| Tuần | Thời gian | Việc | Notebook | Gate |
|---|---|---|---|---|
| 1 | 05–11/10 | Sửa 00, 01; tạo 01a; `git init`; chọn scope pilot | 00, 01, 01a | 01 chạy sạch; scope ghi trong 00 |
| 2 | 12–18/10 | Lưới D4 + mặt nạ; E1, E12 | 03 | Lưới qua kiểm thử |
| 3 | 19–25/10 | E2–E8 | 03 | Mỗi E# có kết luận |
| 4 | 26/10–01/11 | E9–E11; ledger; freeze `prereg-v1` | 03, 00 | Gate EDA |
| 5 | 02–08/11 | Metric + test; B0–B3 trên 3 fold | 04_metric_and_baselines | Bảng baseline |
| 6 | 09–15/11 | Pipeline feature + test leakage | 05_feature_pipeline | test_no_leakage pass |
| 7 | 16–22/11 | Ridge, HGB, LightGBM mặc định | 06_models_v1 | So với baseline |
| 8 | 23–29/11 | Tuning có ngân sách; (tùy) tweedie | 07_lightgbm_tuning | Khóa cấu hình |
| 9 | 30/11–06/12 | Ablation; error analysis theo segment (ADI/CV², promo, perishable, cold-start, h, thứ) | 08_ablation_error_analysis | Bảng RQ2, RQ3 |
| 10 | 07–13/12 | Holdout một lần; hình cuối | 09_final_holdout_and_figures | Ghi log E40 |
| 11 | 14–20/12 | Báo cáo | — | Bản nháp |
| 12 | 21–27/12 | Slide, tập bảo vệ | — | Tập 2 lần |

Ngày bảo vệ là giả định; nếu người dùng cho ngày thật, giãn/nén lịch tương ứng.

### Cấu trúc thư mục

```
configs/      scope_stores.csv, scope_items.csv, params.yaml
data/raw/     CSV gốc (không sửa)
data/interim/ train_daily_profile.csv, train_series_profile.csv
data/processed/ train.parquet, scope.parquet, grid_*.parquet
notebooks/    00 → 09
src/          metrics.py, baselines.py, grid.py, features.py
tests/        test_metrics.py, test_no_leakage.py
reports/      figures/, tables/, experiment_log.md
```

### Cấu trúc báo cáo

1. Giới thiệu (RQ, phạm vi) · 2. Dữ liệu và kiểm định chất lượng (F1–F14) · 3. **EDA → quyết định** (E1–E12 + ledger, ~25–30% độ dài) · 4. Phương pháp (D1–D9, feature, mô hình, chống leakage) · 5. Kết quả (RQ1–3, holdout) · 6. Thảo luận và hạn chế · 7. Kết luận.

---

## Phụ lục B. Experiment log (`reports/experiment_log.md`)

| ID | Ngày | Thay đổi | Lý do (E#/D#) | F1 | F2 | F3 | Mean ± std | Quyết định |
|---|---|---|---|---|---|---|---|---|
| E00 | 2026-09-29 | Chốt scope | D1 | | | | | |
| E01 | | B0–B3 | D3 | | | | | |
| E40 | | Holdout (một lần) | | — | — | — | Holdout: | |

## Phụ lục C. Code tham chiếu (đã chạy thử trên dữ liệu giả)

```python
import math
import numpy as np
import pandas as pd

def nwrmsle(y_true, y_pred, perishable):
    w = np.where(np.asarray(perishable) == 1, 1.25, 1.0)
    yt = np.clip(np.asarray(y_true, dtype=float), 0, None)
    yp = np.clip(np.asarray(y_pred, dtype=float), 0, None)
    err = (np.log1p(yp) - np.log1p(yt)) ** 2
    return np.sqrt((w * err).sum() / w.sum())

def test_nwrmsle():
    e1 = np.e - 1                      # log1p(e - 1) = 1
    assert np.isclose(nwrmsle([0, e1], [0, 0], [0, 0]), np.sqrt(1 / 2))        # 0.7071
    assert np.isclose(nwrmsle([0, e1], [0, 0], [0, 1]), np.sqrt(1.25 / 2.25))  # 0.7454
    assert nwrmsle([3, 5], [3, 5], [1, 0]) == 0


def build_grid(df, active, start, T):
    """Lưới D4. df: dạng dài (store_nbr, item_nbr, date, unit_sales), đã kẹp âm.
    active: DataFrame bool, index=store_nbr, columns=ngày. NaN = không hoạt động/chưa xuất hiện."""
    dates = pd.date_range(start, T, freq="D")
    df = df[(df["date"] >= dates[0]) & (df["date"] <= dates[-1])]
    w = (df.pivot_table(index=["store_nbr", "item_nbr"], columns="date",
                        values="unit_sales", aggfunc="sum")
           .reindex(columns=dates))
    vals = w.to_numpy(dtype="float32")
    has = ~np.isnan(vals)
    first = np.where(has.any(1), has.argmax(1), vals.shape[1])
    after_first = np.arange(vals.shape[1])[None, :] >= first[:, None]
    store_on = (active.reindex(index=w.index.get_level_values("store_nbr"), columns=dates)
                      .fillna(False).to_numpy(dtype=bool))
    vals[(~has) & after_first & store_on] = 0.0
    return pd.DataFrame(vals, index=w.index, columns=dates)


def seasonal_baseline(grid, T, horizon=16, n_weeks=1):
    """n_weeks=1 → B2, n_weeks=4 → B3. Bỏ qua ngày NaN."""
    T = pd.Timestamp(T)
    out = {}
    for h in range(1, horizon + 1):
        d = T + pd.Timedelta(days=h)
        back = 7 * math.ceil(h / 7)
        src = [d - pd.Timedelta(days=back + 7 * k) for k in range(n_weeks)]
        assert max(src) <= T            # chốt chặn leakage
        out[d] = grid.reindex(columns=src).mean(axis=1, skipna=True).fillna(0)
    return pd.DataFrame(out)

def last_value_baseline(grid, T, horizon=16):  # B1
    T = pd.Timestamp(T)
    last = grid.loc[:, :T].ffill(axis=1).iloc[:, -1].fillna(0)
    return pd.DataFrame({T + pd.Timedelta(days=h): last for h in range(1, horizon + 1)})


def demand_class(grid_window):
    """ADI/CV² cho E2. Tách trước các series có 0–1 ngày bán (cv2 không có nghĩa)."""
    x = grid_window.to_numpy()
    n_obs = np.sum(~np.isnan(x), axis=1)
    pos = np.where(x > 0, x, np.nan)
    n_pos = np.sum(~np.isnan(pos), axis=1)
    adi = np.where(n_pos > 0, n_obs / np.maximum(n_pos, 1), np.inf)
    with np.errstate(invalid="ignore", divide="ignore"):
        cv2 = (np.nanstd(pos, axis=1) / np.nanmean(pos, axis=1)) ** 2
    cls = np.select(
        [(adi < 1.32) & (cv2 < 0.49), (adi >= 1.32) & (cv2 < 0.49), (adi < 1.32) & (cv2 >= 0.49)],
        ["smooth", "intermittent", "erratic"], default="lumpy")
    return pd.DataFrame({"adi": adi, "cv2": cv2, "class": cls}, index=grid_window.index)


def ratio_to_baseline(daily, exclude_dates=()):
    """E5/E6/E8: giá trị ngày / trung bình 4 ngày cùng thứ (±7, ±14), bỏ ngày trong exclude_dates khỏi nền."""
    s_clean = daily.where(~daily.index.isin(pd.to_datetime(list(exclude_dates))))
    base = sum(s_clean.shift(k) for k in (-14, -7, 7, 14)) / 4
    return daily / base
```

## Phụ lục D. Notebook Kaggle tham khảo

`demand-forecasting-favorita-grocery-sales.ipynb` mô phỏng feature của lời giải hạng 1. **Lấy ý tưởng:** thống kê nhiều cửa sổ (1, 3, 7, 14, 30, 60, 140 ngày) tại ngày tham chiếu; thống kê nhiều cấp (store×item, item, store×class); promo 16 ngày tới; tổ chức theo ngày tham chiếu (= D3). **Không lấy:** số liệu/điểm; `fillna(0)` mọi feature (trái D4); câu "chỉ 2017 hữu ích" như sự thật (cần E1); cách validation một cửa sổ. Trong báo cáo ghi là "tham khảo lời giải cộng đồng Kaggle".

## Phụ lục E. Decision ledger (`reports/tables/eda_decision_ledger.csv`)

Cột: `E#, câu hỏi, bằng chứng (file hình/bảng, có số), kết luận, quyết định, ảnh hưởng tới (D#/G#/RQ)`.
