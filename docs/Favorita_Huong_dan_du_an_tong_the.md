# Hướng dẫn thực hiện dự án tổng thể

## Dự báo doanh số bán lẻ theo cửa hàng và mặt hàng bằng học máy trên bộ dữ liệu Corporación Favorita

> **Nguồn tổng hợp:** `Huong_dan_ML.md` (lộ trình 10 tuần và kỷ luật thực nghiệm), `00_scope_and_preregistration.ipynb`, `checking_stage.ipynb`, `notebook.ipynb` (EDA từng bảng).
> **Trạng thái** trong tài liệu được ghi theo output đã chạy trong các notebook, tính đến 24/09/2026. Bạn đang ở **cuối Tuần 1**: audit dữ liệu đã xong và qua cổng, nhưng phạm vi (scope) và pre-registration chưa được khóa.

---

## 0. Cách dùng tài liệu

| Ký hiệu | Ý nghĩa |
|---|---|
| `[x]` | Đã xong, có bằng chứng trong notebook (ghi nguồn trong ngoặc) |
| `[ ]` | Chưa xong |
| 🔒 **Gate** | Điều kiện bắt buộc trước khi sang bước sau |
| ⚠️ **D#** | Quyết định thiết kế phải chốt và ghi vào pre-registration trước khi modeling (mục 3) |
| 🧠 | Cổng kiến thức: bạn tự trả lời bằng lời của mình. Tài liệu chỉ liệt kê câu hỏi, không trả lời hộ |

**Quy tắc vận hành:**

1. Mỗi thay đổi thiết kế sau khi khóa pre-registration phải ghi vào experiment log (Phụ lục B).
2. Không chuyển sang tuần kế tiếp nếu Gate của tuần hiện tại chưa đạt.
3. Mọi con số trong báo cáo phải truy được về một file kết quả (`reports/`).

---

## 1. Tóm tắt dự án

| Hạng mục | Nội dung |
|---|---|
| Tên tiếng Việt | Dự báo doanh số bán lẻ theo cửa hàng và mặt hàng bằng học máy trên bộ dữ liệu Corporación Favorita |
| Tên tiếng Anh | Machine Learning Based Retail Sales Forecasting at the Store-Item Level Using the Corporación Favorita Dataset |
| Đơn vị dự báo (grain) | `store_nbr × item_nbr × date` |
| Biến mục tiêu | `unit_sales` của ngày tương lai (số lượng bán, không phải nhu cầu thật khi hết hàng) |
| Chân trời (horizon) | 16 ngày |
| Validation | 3 fold rolling-origin (mỗi fold 16 ngày) + 1 holdout cuối chưa nhìn tới |
| Metric chính | NWRMSLE / RMSLE theo định nghĩa cuộc thi; phụ: MAE, WAPE |
| Baseline | B0 zero, B1 last value, B2 seasonal naive, B3 mean các tuần cùng thứ |
| Mô hình | Ridge, HistGradientBoosting, Random Forest (mẫu có kiểm soát), **LightGBM (mô hình chính)** |
| Sản phẩm | Pipeline tái lập, bảng thí nghiệm, ablation, error analysis, báo cáo, slide |

### Ba câu hỏi nghiên cứu

| Mã | Câu hỏi | Bằng chứng cần có |
|---|---|---|
| RQ1 | Mô hình ML có vượt seasonal-naive/B3 không? | Điểm trung bình ± std và từng fold qua 3 fold |
| RQ2 | Nhóm feature (lịch sử, khuyến mãi, lịch, metadata, ngữ cảnh) đóng góp bao nhiêu? | Bảng ablation (bỏ từng nhóm, đo mức giảm hiệu năng) |
| RQ3 | Hiệu năng khác nhau thế nào theo segment (bán nhanh/chậm, có/không khuyến mãi, perishable, cold-start)? | Error analysis theo segment, không chỉ một con số tổng |

### Bảy nguyên tắc không thương lượng

1. **Không** dùng `train_test_split` ngẫu nhiên.
2. Feature chỉ dùng thông tin có tại thời điểm dự báo: dữ liệu ≤ cutoff, cộng các biến biết trước của ngày đích (lịch, `onpromotion` của ngày đó).
3. Mọi thống kê (encoder, median, mean encoding, chọn scope) chỉ fit trên phần train của từng fold.
4. Baseline và mô hình so sánh trên cùng dòng, cùng horizon, cùng tiền xử lý, cùng lượng thông tin.
5. Báo cáo mean ± std qua các fold, kèm kết quả từng fold.
6. Holdout cuối chỉ chạy **một lần** sau khi khóa thiết kế.
7. Không overclaim: không suy nhân quả từ khuyến mãi, không nói "tối ưu tồn kho", không đồng nhất `unit_sales` với nhu cầu thật, không dùng leaderboard làm bằng chứng khoa học.

---

## 2. Hiện trạng dự án

### 2.1 Bản đồ notebook

| File hiện tại | Đề xuất tên | Trạng thái | Còn thiếu |
|---|---|---|---|
| `00_scope_and_preregistration.ipynb` | giữ nguyên | ⏳ Một phần | Scope Decision còn ô trống; chưa có ngày fold; chưa chốt các dòng "Verify"; chưa freeze (chi tiết ở 2.3) |
| `checking_stage.ipynb` | `01_data_checking.ipynb` | ✅ Hoàn tất | 11/11 kiểm tra bắt buộc PASS |
| `notebook.ipynb` | `02_eda_raw_tables.ipynb` | ⏳ Một phần | Mới là EDA từng bảng (phân bố, missing, heatmap); thiếu phân tích quan hệ theo yêu cầu Tuần 3 |
| (mới) | `03_metric_and_baselines.ipynb` | ⬜ | Tuần 2 và 4 |
| (mới) | `04_feature_pipeline.ipynb` | ⬜ | Tuần 5 |
| (mới) | `05_models_v1.ipynb`, `06_lightgbm_tuning.ipynb` | ⬜ | Tuần 6 và 7 |
| (mới) | `07_ablation_error_analysis.ipynb`, `08_final_holdout_and_figures.ipynb` | ⬜ | Tuần 8 và 9 |

### 2.2 Phát hiện dữ liệu then chốt và hệ quả thiết kế

Các số liệu dưới đây lấy từ output đã chạy trong `00_...`, `checking_stage.ipynb` và `notebook.ipynb`.

| # | Phát hiện | Hệ quả | Xử lý ở |
|---|---|---|---|
| F1 | `train.csv` 125,497,040 dòng, 4,765.94 MB. Máy: 8 CPU logic, RAM 7.50 GB tổng nhưng chỉ **2.10 GB khả dụng**, không GPU. Sample 100k dòng đầu tốn 10.2 MB → đọc ngây thơ ≈ 12–13 GB | Không thể load toàn bộ; cần scope và cache Parquet | **D1** |
| F2 | Train: 54 store, 4,036 item, 174,685 series. Test: 54 × 3,901 = 210,654 cặp × 16 ngày = 3,370,464 dòng | Test là lưới Cartesian đầy đủ | D4 |
| F3 | Train **thưa**: 0 dòng `unit_sales == 0`. Ở 16 ngày cuối train, chỉ 46.8%–52.6% số cặp trong lưới test có dòng dữ liệu mỗi ngày | Dòng thiếu ≠ dòng bằng 0 một cách hiển nhiên; phải có quy tắc rõ | **D4** |
| F4 | 4 ngày thiếu toàn bộ: 25/12 các năm 2013–2016 (Navidad). Ngày 2013-01-01 chỉ có 1 store | `shift`/`rolling` phải theo **ngày lịch**, không theo vị trí dòng | D4, Tuần 5 |
| F5 | Trong validation, tất cả các ngày đều có đủ mặt 54 store tham gia bán hàng | Cả 54 cửa hàng đều mở cửa trong tập validation | D4, Tuần 3 |
| F6 | 7,795 dòng âm (0.0062%); min −15,372, max 89,440; percentile 99.5% ≈ 100 (ước lượng từ 1% mẫu) | Quy tắc xử lý âm và đuôi nặng | **D5** |
| F7 | `onpromotion` NaN 21,657,651 dòng (17.26%); True 7,810,622 (6.22%). Các ngày đầu 2013 NaN hoàn toàn, các ngày 2017 không còn NaN. Test có `onpromotion` đầy đủ | Không `fillna(False)` bừa; xác định mốc bắt đầu có dữ liệu | **D6** |
| F8 | 60 item chỉ có trong test; 44,441 cặp store-item chưa từng xuất hiện trong train (21.10% số cặp test) | Cold-start là phần đáng kể; baseline lịch sử cho 0 | **D8** |
| F9 | `transactions.csv` kết thúc 2017-08-15, không có cho horizon | Chỉ được dùng dạng lag/rolling ≤ cutoff, hoặc bỏ | **D7** |
| F10 | `oil.csv`: 43 NaN (3.53%), chỉ có ngày thường (không có T7/CN); file phủ tới 2017-08-31 | Giá dầu sau cutoff là tương lai, dù file có sẵn | **D7** |
| F11 | `holidays_events.csv`: 350 dòng, 38 ngày có nhiều sự kiện cùng ngày (khác locale/mô tả) | Join theo locale (National/Regional/Local), không join chỉ theo `date` | **D7** |
| F12 | Notebook checking cố định **một** cửa sổ 2017-07-31 → 2017-08-15; hướng dẫn yêu cầu 3 fold + holdout | Cần định nghĩa lại vai trò cửa sổ này | **D2** |
| F13 | Validation có Thứ Hai/Thứ Ba ×3; test có Thứ Tư/Thứ Năm ×3 (cửa sổ 16 ngày luôn lệch 2 thứ) | Báo cáo thêm metric theo thứ trong tuần | D2 |

### 2.3 Việc cần sửa ngay trong notebook hiện có

**`00_scope_and_preregistration.ipynb`**

- [x] Điền **Scope Decision** (đang là `[FULL DATA / X STORES / TOP-N ITEMS]`). Thay `Available RAM: 1,53 GB` bằng số đo thực và ghi rõ thời điểm đo (output đã chạy cho 2.10 GB).
- [x] Mục 3: ghi ngày cụ thể của F1/F2/F3 và holdout (đề xuất ở D2).
- [x] Mục 4: **viết lại B1–B3 theo cutoff**. Công thức `ŷ(t) = y(t-7)` chỉ đúng khi horizon ≤ 7 (xem D3).
- [x] Mục 5: thay `[model to be specified]` bằng mô hình ML đơn giản đã chọn (đề xuất: Ridge).
- [x] Mục 7: chốt các dòng "Depends / Verify / Must verify" (đề xuất ở D7).
- [x] Mục 11 và 12 đang **trùng nhau** (cùng là "Test Set Protection"). Gộp thành một mục và định nghĩa rõ "independent test set" là gì (Kaggle test không có nhãn, xem D2).
- [x] Thêm mục cho các quy tắc D4 (thưa/zero), D5 (âm), D6 (promo NaN), D8 (cold-start).
- [x] Điền cột Date của Experiment Log; **freeze** (git commit/tag `prereg-v1`).

**`notebook.ipynb` (EDA)**

- [x] Cell cuối trỏ tới `checking_stage_01_05.ipynb`, nhưng file thực tế là `checking_stage.ipynb`. Sửa cho khớp (hoặc đổi tên theo bảng 2.1).
- [x] Các bảng nhỏ (`items`, `stores`, `oil`, `holidays`) đang đọc theo chunk 100k dòng dù chỉ vài trăm đến vài nghìn dòng. Không sai nhưng thừa. Đọc trực tiếp cho gọn.
- [x] Bổ sung phân tích quan hệ cho Tuần 3 (mục 5, Tuần 3).

**`checking_stage.ipynb`**

- [x] Hoàn tất. Nên lưu thêm bảng tóm tắt F1–F13 ở trên vào `reports/tables/data_audit_summary.md` để báo cáo trích dẫn.
- [x] Notebook quét toàn bộ `train.csv` nhiều lần (mỗi lần đọc 4.7 GB). Sau khi có bản Parquet của scope (Tuần 1–2), các bước sau **không** đọc lại CSV gốc.

---

## 3. Quyết định thiết kế D1–D9

Đây là phần quan trọng nhất của tài liệu. Các quyết định dưới đây xuất phát từ số liệu audit và từ một vài điểm mà công thức trong hướng dẫn gốc chưa khớp với bài toán dự báo 16 ngày. Mỗi mục có **khuyến nghị**; bạn quyết định cuối cùng và ghi vào pre-registration.

### D1. Phạm vi dữ liệu (scope)

**Bằng chứng:** F1. RAM khả dụng 2.10 GB; ngay cả dtype gọn (≈ 12 byte/dòng) `train` cũng ≈ 1.4 GiB chưa tính feature.

**Khuyến nghị:** *X cửa hàng + mẫu mặt hàng phân tầng*, cửa sổ lịch sử 12–18 tháng.

- Chọn cửa hàng phân tầng theo `type × cluster` (và `city/state`), không chọn ngẫu nhiên thuần.
- Chọn mặt hàng phân tầng theo `family × mức doanh số × perishable` thay vì **top-N thuần**. Top-N chỉ giữ hàng bán nhanh và làm RQ3 (hàng bán chậm) mất bằng chứng.
- Việc chọn scope **chỉ dùng dữ liệu ≤ cutoff của F1** để không rò rỉ. Cả 3 fold và holdout dùng chung một scope.
- Lưu danh sách đã chọn ra `configs/scope_stores.csv`, `configs/scope_items.csv` cùng seed.
- (Tùy chọn) giữ một nhóm nhỏ series "mới" (lần bán đầu gần cutoff) để phân tích cold-start.
- Nếu cửa sổ train phủ tháng 04/2016, cân nhắc cờ sự kiện động đất 16/04/2016 (theo mô tả dữ liệu cuộc thi, hãy tự đối chiếu).

**Ước lượng kích thước** (dùng để chốt scope, không đoán):

```
số dòng train  ≈ số_cặp × số_origin × 16          (số_origin ≈ số_ngày_lịch_sử / origin_stride)
RAM ma trận    ≈ số dòng × số_feature × 4 byte    (float32)
Mục tiêu       : ma trận feature ≤ 0.5–0.7 GB, vì pandas thường tạo 2–3 bản sao trung gian
```

Ví dụ: 10 store × 500 item = 5,000 cặp; 12 tháng với `origin_stride = 7` → ≈ 52 origin; 5,000 × 52 × 16 ≈ **4.2 triệu dòng**; 30 feature → ≈ **0.5 GB**. Đây là điểm khởi đầu hợp lý; tăng dần sau pilot.

**Checklist D1**

- [ ] Chạy pilot 1% (hoặc 1–2 store) qua pipeline, đo peak RAM (`psutil`) và thời gian.
- [ ] Chọn scope sao cho peak RAM ≤ ~60% RAM khả dụng.
- [ ] Ghi vào pre-registration: scope, lý do (RAM/thời gian), thứ được giữ (thứ tự thời gian, nhiều series, biến thiên promo, dị biệt store/item) và hạn chế (mẫu có thể không đại diện toàn quần thể).
- [ ] Chuyển `train.csv` → Parquet (dtype gọn: `int16` store, `int32` item, `float32` sales, `int8` promo) **chỉ cho scope**. Có thể dùng Polars lazy/DuckDB để lọc mà không load hết.

### D2. Fold, holdout và vai trò của "test"

**Bằng chứng:** F12. `test.csv` của Kaggle (2017-08-16 → 08-31) **không có nhãn**, nên không thể làm "independent test set" cho đánh giá cục bộ. Cửa sổ 2017-07-31 → 08-15 mà notebook checking gọi là "validation" chính là 16 ngày cuối của train.

**Khuyến nghị:** dùng cửa sổ 16 ngày cuối train làm **FINAL HOLDOUT** (đổi tên trong pre-registration), và đặt 3 fold lùi về trước, liền kề nhau, không chồng lấn:

| Fold | Train đến hết | Validation | Thứ của cutoff |
|---|---|---|---|
| F1 | 2017-06-12 | 2017-06-13 → 2017-06-28 | Thứ Hai |
| F2 | 2017-06-28 | 2017-06-29 → 2017-07-14 | Thứ Tư |
| F3 | 2017-07-14 | 2017-07-15 → 2017-07-30 | Thứ Sáu |
| **HOLDOUT** (chỉ chạy ở E40) | 2017-07-30 | 2017-07-31 → 2017-08-15 | Chủ Nhật |
| Kaggle test (tùy chọn, chỉ để nộp bài) | 2017-08-15 | 2017-08-16 → 2017-08-31 | Thứ Ba |

- Train mở rộng (expanding window): train của F2 gồm cả validation của F1, v.v.
- Kaggle test chỉ dùng để tạo file nộp cuối cùng; điểm leaderboard **không** là bằng chứng khoa học.
- Cutoff rơi vào các thứ khác nhau là bình thường; hãy ghi lại và báo cáo thêm metric theo thứ (F13).
- Nếu bạn muốn dùng 07-31 → 08-15 làm F3 thì sẽ **không còn holdout chưa nhìn tới**; khi đó phải viết lại Mục 11 của pre-registration cho phù hợp.
- (Tùy chọn) Cả 4 cửa sổ đều nằm trong mùa hè 2017, ít ngày lễ quốc gia. Nếu muốn RQ2/RQ3 nói được về ngày lễ, thêm một fold bổ sung chứa ngày lễ (báo cáo riêng, không trộn vào mean ± std chính).

### D3. Baseline và feature phải neo theo cutoff

**Vấn đề:** Gọi `T` là ngày cuối cùng đã quan sát, `d` là ngày cần dự báo, `h = d − T ∈ {1..16}`. Công thức `ŷ(t) = y(t-7)` và `lag 1, 7, 14` trong hướng dẫn gốc chỉ đúng khi dự báo một bước. Với dự báo 16 ngày, `y(d-7)` ở `h > 7` là một ngày **chưa xảy ra**: dùng nó là rò rỉ tương lai (hoặc ra NaN).

**Baseline theo cutoff** (missing → 0 theo D4):

| Mã | Công thức |
|---|---|
| B0 | `ŷ = 0` |
| B1 | `ŷ = y(T)` (giá trị cuối cùng đã quan sát của cùng cặp) |
| B2 | `ŷ = y(d − 7·⌈h/7⌉)` |
| B3 | `ŷ = mean_{k=0..3} y(d − 7·(⌈h/7⌉ + k))` |

Độ lùi của B2 theo `h`: `h = 1–7` → lùi 7; `h = 8–14` → lùi 14; `h = 15–16` → lùi 21. Nguồn luôn ≤ `T`. Mã tham chiếu ở Phụ lục C.

**Thiết kế ML "direct-anchored" (khuyến nghị):** mỗi dòng huấn luyện là `(origin T', store, item, h)`.

- *Biết trước của ngày đích `d`:* calendar(d), `onpromotion(d)`, holiday(d), `h`.
- *Lịch sử ≤ `T'`:* `y(T')`, các lag cùng thứ `y(d − 7·(⌈h/7⌉ + k))` với `k = 0..3` (chính là đầu vào của B2/B3), rolling mean/std 7/14/28 ngày kết thúc tại `T'`, số ngày có bán trong 28 ngày, số ngày kể từ lần bán cuối.
- *Metadata:* item, store.
- *Origin huấn luyện:* `T' ≤ T − 16` để mọi nhãn `d = T' + h` đều nằm trong train của fold. Đây chính là chốt chặn chống leakage.
- *`origin_stride`:* tham số cấu hình; bắt đầu bằng 7, giảm nếu RAM cho phép. Số dòng = số_cặp × số_origin × 16.

**Phương án dự phòng "safe-lag" (mỗi dòng một ngày đích, mọi lag ≥ 16, dùng `shift(16)`):** rẻ hơn về RAM nhưng ML **kém thông tin hơn** B2/B3 (B2 dùng `y(d-7)` với `h ≤ 7`, còn ML chỉ thấy tới `d-16`). Nếu dùng phương án này phải ghi rõ, và thêm B2-safe/B3-safe (lùi 21, 28, 35, 42) để so sánh công bằng; không được tuyên bố "vượt B3 chuẩn" dựa trên so sánh lệch thông tin.

- [ ] Viết lại Mục 4 của pre-registration theo công thức trên.
- [ ] Chọn direct-anchored hoặc safe-lag, ghi lý do.
- [ ] 🧠 Tự giải thích: vì sao `shift(1).rolling(7)` không đủ an toàn cho dự báo 16 ngày trong một lần chạy?

### D4. Dữ liệu thưa và "zero ngầm"

**Bằng chứng:** F3, F4, F5. Train không có dòng bằng 0; nguồn dữ liệu cuộc thi cũng nêu dòng không bán bị bỏ và không có thông tin tồn kho. Nghĩa là dòng thiếu có thể là 0 bán, hết hàng, chưa bán mặt hàng đó, hoặc store không hoạt động hôm đó.

**Khuyến nghị (ghi vào pre-registration):**

1. Với mỗi cặp trong scope, dựng lưới ngày liên tục từ **ngày có dòng đầu tiên của cặp** (≤ cutoff) đến cutoff. Không điền số 0 cho giai đoạn trước khi cặp xuất hiện.
2. Điền `0` cho dòng thiếu **chỉ ở store-day có ít nhất một dòng** (store có hoạt động ghi nhận hôm đó). Store-day không có dòng nào (F4, F5) được gắn cờ và loại khỏi nhãn huấn luyện/đánh giá; đừng biến ngày đóng cửa thành "bán 0".
3. Đánh giá **chính** trên lưới đầy đủ theo quy tắc (1)–(2). Báo cáo **phụ** trên "chỉ dòng có bản ghi" để thấy độ nhạy.
4. (Nếu đủ tài nguyên) ablation độ nhạy: có/không điền zero.
5. **Cấm** `shift` theo vị trí dòng trên dữ liệu thưa: một cặp thiếu ngày sẽ bị lệch lịch. Luôn dựng lưới ngày hoặc tra cứu theo khóa `(store, item, date)`.

- [ ] Đo tỷ lệ store-day không có dòng nào trong scope, theo tháng (đưa vào EDA Tuần 3).
- [ ] 🧠 Tự giải thích: vì sao coverage ≈ 50% khiến baseline "zero" mạnh hơn bạn tưởng, và vì sao điều này ảnh hưởng RMSLE?

### D5. Biến mục tiêu và dòng âm

**Bằng chứng:** F6.

- Kẹp `unit_sales < 0` về `0` cho cả nhãn và feature lịch sử (dòng âm là hoàn trả, chỉ 0.0062%). Ghi số dòng bị kẹp theo từng fold.
- Huấn luyện trên `log1p(y_clipped)`, dự đoán `expm1`, kẹp dự đoán ≥ 0. Cách này khớp với RMSLE.
- Không xóa outlier lớn (max 89,440): `log1p` đã nén; nếu cân nhắc winsorize thì ghi thành một thí nghiệm riêng.
- Với NWRMSLE, dùng `sample_weight = 1.25` cho hàng perishable khi huấn luyện để khớp hàm mục tiêu (xác minh trọng số ở D9).

### D6. `onpromotion` bị thiếu

**Bằng chứng:** F7.

- [ ] Ở EDA (Tuần 3): xác định **ngày đầu tiên** `onpromotion` không còn NaN, và tỷ lệ NaN theo tháng.
- [ ] Cửa sổ huấn luyện nên nằm hoàn toàn sau mốc đó (cửa sổ 12–18 tháng gần cutoff thường thỏa điều kiện này; hãy kiểm chứng bằng số).
- [ ] Không `fillna(False)` toàn cục. Nếu còn NaN ở biên cửa sổ: dùng cờ `promo_missing` cho Ridge/HGB; LightGBM xử lý NaN nguyên bản.
- [ ] `onpromotion(d)` của ngày đích dùng được vì `test.csv` cung cấp cho cả 16 ngày, và trong fold thì lấy từ train. Ghi giả thiết này vào pre-registration: *"kế hoạch khuyến mãi của 16 ngày tới đã biết tại cutoff"*.
- [ ] Không diễn giải hệ số/importance của promo như hiệu ứng nhân quả (mục Out of Scope).

### D7. Bảng phụ: cái gì dùng được tại thời điểm dự báo

Đây là bản chốt cho các dòng "Verify" trong Mục 7 của pre-registration.

| Nguồn | Có tại cutoff `T`? | Quy tắc |
|---|---|---|
| Calendar của ngày `d` | Có | Dùng |
| Store/item metadata | Có | Dùng |
| `onpromotion(d)` trong horizon | Có (theo D6) | Dùng cho ngày `d`; lịch sử promo chỉ ≤ `T'` |
| Sales ≤ `T` | Có | Dùng qua feature neo cutoff |
| Sales > `T` | Không | **Cấm tuyệt đối** |
| `transactions` | Chỉ tới `T` (file dừng 2017-08-15) | Chỉ lag/rolling ≤ `T'` ở mức store; hoặc bỏ. Không dùng giá trị ngày `d` |
| Giá dầu | File có tới 2017-08-31, nhưng giá sau `T` là tương lai | Chỉ dùng giá trị ≤ `T'`, forward-fill **từ quá khứ**; NaN ngày thường là ngày nghỉ của thị trường dầu |
| `holidays_events` | Lịch công bố trước | Dùng, join theo locale khớp `store` (National khớp mọi store; Regional khớp `state`; Local khớp `city`); xử lý `transferred` có chủ đích |

- [ ] 🧠 Tự giải thích: vì sao `onpromotion` của ngày dự báo hợp lệ còn `transactions` của ngày dự báo thì không?

### D8. Cold-start

**Bằng chứng:** F8 (21.10% cặp test chưa có lịch sử; 60 item chỉ có trong test).

- Định nghĩa cold-start **theo từng fold**: cặp không có dòng nào ≤ `T`. Với cặp này B0–B3 đều cho 0.
- Chính sách dự phòng cho ML (fit trên train của fold): thống kê phân cấp theo `item` ở các store khác, rồi `family × store`, rồi 0.
- Luôn báo cáo cold-start thành segment riêng (RQ3), đừng để nó nằm lẫn trong con số tổng.

### D9. Metric

- Metric chính: **NWRMSLE** = `sqrt( Σ wᵢ·(log(1+ŷᵢ) − log(1+yᵢ))² / Σ wᵢ )`, `wᵢ = 1.25` nếu perishable, ngược lại `1.0`. Hãy **tự xác minh** trọng số và cách xử lý giá trị âm trên trang Evaluation chính thức của cuộc thi (`Huong_dan_ML.md` yêu cầu việc này).
- Kẹp `y` và `ŷ` về ≥ 0 trước khi tính log.
- Tính metric trên toàn bộ dòng của một fold, rồi báo cáo mean ± std qua 3 fold. Metric phụ: MAE, WAPE ở cấp tổng hợp. **Không** dùng MAPE (mẫu số 0).
- Viết unit test cho công thức (giá trị tham chiếu ở mục 8 và Phụ lục C).

