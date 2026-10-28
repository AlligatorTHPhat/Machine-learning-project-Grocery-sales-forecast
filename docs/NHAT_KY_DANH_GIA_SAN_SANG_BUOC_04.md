# Nhật ký phát hiện và đánh giá mức sẵn sàng sang bước 04 (baseline)

Tài liệu này tổng hợp những gì đọc được từ kết quả bạn vừa gửi: notebook 03 đã chạy trên dữ liệu thật, `03_feature_engineering_report.json`, `03_config.json`, `03_encodings.json`, `02_cleaning_report.json`, `00_environment_snapshot.json`, `02_raw_manifest.json` và ba notebook 00, 01, 02 chạy lại.

Quy ước: **[Đo]** là con số đọc trực tiếp từ output; **[Ước]** là con số tôi tính nhẩm từ output và dựa trên giả định nêu rõ; **[Suy luận]** là kết luận chưa được kiểm chứng bằng code.

---

## 1. Kết luận

**Có thể sang bước 04, nhưng có điều kiện.** Pipeline feature đúng và an toàn về rò rỉ thời gian; vướng mắc còn lại nằm ở **cách đánh giá và cách xử lý các nhóm cặp**, không phải ở chất lượng dữ liệu.

| Cổng kiểm tra | Kết quả | Căn cứ |
|---|---|---|
| Dữ liệu đã làm sạch đúng và nguyên vẹn | **Đạt** | 02: 11/11 mục checklist PASS, 22/22 checksum khớp, raw không đổi |
| Feature tính đúng | **Đạt** | 03: sai khác lớn nhất so với cài đặt brute-force độc lập là 1,6e-7 |
| Không rò rỉ thông tin tương lai vào feature | **Đạt** | 03: nhiễu `unit_sales` sau ngày mốc, feature không đổi, nhãn đổi |
| Định dạng đầu ra khớp bài nộp | **Đạt** | 03: 61/61 kiểm tra cấu trúc PASS; `id` test trùng khớp 3.370.464 dòng |
| Có mốc so sánh (baseline luật) | **Đạt** | Tốt nhất: trung bình 28 ngày, NWRMSLE **0,6386** trên V1 |
| Nhãn khuyến mãi tương lai không làm lộ đáp án | **Không đạt, cần xử lý bằng thực nghiệm** | `P(y>0 \| promo_t=1)` = **100%** ở train và V1 |
| Đánh giá phản ánh cơ cấu của test | **Đạt có điều kiện** | V1 chỉ có cặp cũ; test có ≈ 21% cặp mới |
| Vừa RAM khi huấn luyện | **Đạt có điều kiện** | Dữ liệu huấn luyện ≈ 0,7–0,8 GB so với RAM khả dụng ≈ 1,3 GB |

**Năm điều kiện bắt buộc khi bắt đầu bước 04** (chi tiết ở mục 6):

1. Chỉ dùng mô hình chính cho cặp **có lịch sử** (`pair_status` 0 và 1); cặp mới (2 và 3) xử lý riêng.
2. Luôn so sánh mô hình **có và không có** nhóm *khuyến mãi tương lai*.
3. **Loại giá dầu** (`oil_*`) khỏi mô hình đầu tiên.
4. Báo cáo kết quả theo `pair_status` và tính thêm ước lượng theo cơ cấu của test; không chỉ báo một con số V1.
5. Giảm kích thước dữ liệu nạp (cửa sổ từ 2016 trở đi, hoặc bỏ nhóm rủi ro) để không hết RAM.

---

## 2. Trạng thái các file đã nhận

| File | Trạng thái |
|---|---|
| `00_scope_and_environment_77d9f4`, `01_eda_ba669f`, `02_data_cleaning_86ecf4` | Mã nguồn **giống hệt** bản đã giao; chạy không lỗi |
| `03_feature_engineering` | Chạy hết 26/26 cell code, không lỗi; **10,2 phút** (tôi ước vài chục phút) |
| `03_feature_engineering_report.json`, `03_config.json`, `03_encodings.json` | Hợp lệ, đầy đủ |
| `02_cleaning_report.json`, `00_environment_snapshot.json`, `02_raw_manifest.json` | Hợp lệ; khớp các kết quả đã biết |

Không có lỗi runtime nào trong cả bốn notebook.

---

## 3. Nhật ký phát hiện từ notebook 03 (dữ liệu thật)

### 3.1 Vận hành

| Mục | Kết quả [Đo] |
|---|---|
| Thời gian dựng toàn bộ | 10,2 phút (test 57 giây; V1 33 giây; V2 18 giây; 54 mốc huấn luyện ≈ 500 giây) |
| Số mốc huấn luyện | 54 (6 mốc bị loại vì chạm giai đoạn động đất); từ 2015-06-11 đến 2017-07-17 |
| Tỷ lệ lấy mẫu cặp mỗi mốc huấn luyện | 2,32% |
| Số dòng | Huấn luyện **2.970.590**; V1 2.786.240; V2 1.172.130; test 3.370.464 |
| Kích thước file | Huấn luyện 277 MB; V1 193 MB; V2 88 MB; test 196 MB |
| Số feature | 75 (cộng 8 cột meta/nhãn, tổng 83 cột) |
| RAM khả dụng | 1,67 GB lúc bắt đầu, 1,33 GB lúc kết thúc |

### 3.2 Tính đúng đắn

- Cấu trúc đầu ra: 61/61 kiểm tra PASS (`date − origin = h`, không có ngày mục tiêu ≤ mốc, `id` duy nhất, nhãn có đủ).
- Brute-force độc lập: `mean_7/28/112`, `act_28`, `days_since_last_sale`, `pair_age_days`, `promo_days_28`, `dow_mean_4/8` đều khớp (sai khác ≤ 2e-7, tức chỉ là sai số float32).
- Kiểm thử nhiễu tương lai: ma trận lịch sử, khuyến mãi lịch sử, feature cơ bản và feature cùng thứ **giống hệt**; nhãn đã thay đổi (phép nhiễu có tác dụng).
- Cơ cấu cặp của test khớp **chính xác** với notebook 01 (mục 7.16): 71,07% cặp cũ còn bán, 7,84% cặp ngừng bán, 19,56% cặp mới có item đã biết, 1,54% item mới. Điều này xác nhận cả cache ở 01 lẫn cách phân loại ở 03.

### 3.3 Nhãn

| Tập | Dòng | Tỷ lệ `y = 0` | `mean y_log` | Tỷ lệ cặp ngừng bán |
|---|---|---|---|---|
| train (mẫu) | 2.970.590 | 35,3% | 1,112 | 9,4% |
| V1 | 2.786.240 | 39,9% | 1,013 | **13,5%** |
| V2 | 1.172.130 | 35,7% | 1,070 | 9,3% |

Khoảng 35–40% dòng có nhãn 0, phù hợp với fill rate 66% ở notebook 01 cộng thêm các cặp ngừng bán. V1 có tỷ lệ cặp ngừng bán và tỷ lệ nhãn 0 cao hơn các mốc khác; điều này ảnh hưởng đến cách đọc baseline (mục 5).

### 3.4 Độ lệch phân phối train so với test (PSI, chỉ tính cặp có lịch sử)

| Feature | PSI | Đọc |
|---|---|---|
| `store_promo_share_t` | **0,944** | Lệch lớn, **chưa có lời giải thích** (xem mục 6, dòng 8) |
| `pair_age_days` | 0,724 | Bình thường: tuổi cặp tăng theo thời gian (trung bình 856 so với 1.121 ngày) |
| `store_mean_28`, `store_tx_28` | 0,051; 0,049 | Ổn định |
| `promo_h_sum` | 0,027 | Ổn định |
| Mọi feature còn lại | < 0,011 | Ổn định (`mean_28`, `act_28`, `dow_mean_8`, `promo_t`, `item_promo_share_t`...) |

Phần lớn feature ổn định giữa train và test. Tỷ lệ thiếu của test ≈ 21,1% (đúng bằng tỷ lệ cặp mới) ở mọi feature lịch sử, và 1,5% ở `item_mean_28` (các item chưa từng bán).

### 3.5 Khuyến mãi tương lai: hai phát hiện trái chiều

**[Đo] Tỷ lệ khuyến mãi trên mẫu số dày (mọi cặp-ngày) là như nhau:**

| Tập | `promo_t = 1` |
|---|---|
| train | 5,75% |
| V1 | 6,08% |
| test | 5,82% |

Vì vậy phát hiện ở notebook 01 (test 5,89% so với 10,74% ở 28 ngày cuối train) **phần lớn là hiệu ứng mẫu số**, không phải thay đổi thật. Xem mục 4, dòng 1.

**[Đo] Nhưng rò rỉ qua sự tồn tại của dòng là thật và hoàn toàn:**

| Tập | `P(y>0 \| promo_t=1)` | `P(y>0 \| promo_t=0)` |
|---|---|---|
| train | **100,0%** | 62,7% |
| V1 | **100,0%** | 57,5% |

Trong dữ liệu huấn luyện và kiểm định, `promo_t = 1` luôn đi kèm `y > 0`, vì khuyến mãi chỉ được ghi khi có dòng bán. Mô hình sẽ học `promo_t` như một chỉ báo "chắc chắn có bán". Ở test, khoảng 5,8% dòng (≈ 196.000 dòng) có `promo_t = 1`, nhưng ta không biết bao nhiêu trong số đó thực sự bán được. Kết quả: sai số kiểm định của mô hình dùng nhóm này có thể **lạc quan**, và dự báo trên các dòng khuyến mãi của test có thể **thiên cao**.

### 3.6 Baseline luật trên V1 (chỉ cặp có lịch sử) [Đo]

| Baseline | NWRMSLE |
|---|---|
| **B2c trung bình dày 28 ngày** | **0,6386** |
| B2b trung bình 14 ngày | 0,6462 |
| B2a trung bình 7 ngày | 0,6555 |
| B4 `dow_mean_8` (dự phòng `item_mean_28`) | 0,6606 |
| B3b `dow_mean_8` | 0,6610 |
| B3a `dow_mean_4` | 0,6757 |
| B1 giá trị cùng thứ gần nhất | 0,8175 |
| B0 hằng số | 1,0715 |
| B-zero (dự báo 0) | 1,4820 |

Phân tách (baseline B2c):

| Nhóm | Dòng | NWRMSLE |
|---|---|---|
| Cặp cũ, có bán trong 28 ngày | 2.409.184 | **0,6827** |
| Cặp cũ, ngừng bán > 28 ngày | 377.056 | 0,1965 |
| Perishable = 0 | 2.154.208 | 0,6371 |
| Perishable = 1 | 632.032 | 0,6426 |

Theo `h`: thấp nhất ở `h = 1` (≈ 0,608), cao nhất ở `h = 7` (≈ 0,669), dao động trong khoảng 0,63–0,65 ở các bước còn lại. Đây là đánh giá trên **một mốc duy nhất**, nên các đỉnh theo `h` có thể chỉ là nhiễu.

**Nhận xét:**

- Các baseline dựa trên cùng thứ trong tuần (`dow_mean_*`) **không** vượt trung bình 28 ngày. Ở mức từng cặp, dữ liệu thưa và nhiều số 0 nên tín hiệu thứ-trong-tuần yếu hơn kỳ vọng; vẫn nên giữ làm feature vì mô hình kết hợp được với feature khác.
- Hằng số (B0) và dự báo 0 kém xa; xác nhận lịch sử cặp mang tín hiệu lớn.
- Sai số gần như **không khác** giữa perishable và không perishable.
- Tập cặp ngừng bán có sai số thấp (0,197), kéo con số tổng xuống: **V1 có nhiều cặp ngừng bán hơn cơ cấu test**, nên 0,6386 hơi lạc quan (xem mục 5).

### 3.7 Tương quan feature với `y_log` (Spearman, mẫu V1) [Đo]

Mạnh nhất: `mean_28` (0,768), `mean_14` (0,767), `mean_56` và `mean_7` (0,760), `dow_mean_8` (0,748), `mean_112` (0,746), `act_28` (0,738). Các feature lịch sử cặp và cùng thứ chiếm hầu hết 20 vị trí đầu. Tỷ lệ thiếu thấp: `dow_*` 0,047%; `meanpos_28` 13,6% (cặp không có ngày bán trong cửa sổ, hợp lý).

---

## 4. Đính chính các kết luận trước đó

| # | Điều đã nói | Điều số liệu thật cho thấy | Việc cần làm |
|---|---|---|---|
| 1 | 01, mục 7.10 và bảng 9.5: test có tỷ lệ khuyến mãi 5,89% so với 10,74% cuối train, "lệch phân phối" | Trên mẫu số dày: 5,82% (test) so với 5,75% (train). Độ lệch phần lớn là hiệu ứng mẫu số | Sửa kết luận ở 01; rủi ro thật của khuyến mãi là rò rỉ qua sự tồn tại của dòng (mục 3.5), không phải lệch tỷ lệ |
| 2 | 03, mục 11.2: các feature khuyến mãi "có PSI đáng chú ý" | Chỉ `store_promo_share_t` cao (0,944); `promo_t` 0,004, `item_promo_share_t` 0,004, `promo_h_sum` 0,027 | Sửa văn bản; tìm nguyên nhân của `store_promo_share_t` |
| 3 | 03: "mỗi lần chạy có thể mất vài chục phút" | 10,2 phút | Không cần hành động |
| 4 | 03: chạy 03 "sẽ ổn với RAM ≈ 2 GB" | Dữ liệu huấn luyện đầy đủ ≈ 0,83 GB so với 1,33 GB khả dụng; chính notebook cảnh báo thiếu RAM | Xem điều kiện 5 |

Hai mục 1 và 2 là lỗi diễn giải của tôi ở các bước trước; số liệu thật đã sửa lại chúng.

Về `02_cleaning_report.json`: các mục decision log vẫn ghi `Open` cho DL02, DL07, DL08, DL10, DL11. Phần lớn nay đã được xử lý trong 03:

| Mục | Nội dung | Trạng thái thực tế sau 03 |
|---|---|---|
| DL02 | Dòng ngầm bằng 0 | Xử lý: tensor dày, ngày vắng = 0, mẫu số theo tuổi cặp |
| DL07 | Giá dầu thiếu | Xử lý: điền tiến, chỉ giá trị trước mốc; nhóm exploratory |
| DL08 | Outlier `unit_sales` | **Vẫn mở**: chỉ dùng `log1p`, chưa cắt giá trị cực lớn |
| DL10 | Holidays trùng khóa | Xử lý: 350 dòng gom thành 343 nhóm trước khi nối |
| DL11 | `transactions` thiếu | Xử lý: để NaN, không điền 0 (thiếu 0,035%) |

---

## 5. Ước lượng sai số theo cơ cấu của test [Ước]

Con số 0,6386 chỉ áp dụng cho cặp có lịch sử với cơ cấu của V1. Test có cơ cấu khác (mục 3.2). Tính lại bằng cách trộn bình phương sai số của hai nhóm đã đo (0,6827² cho cặp đang bán, 0,1965² cho cặp ngừng bán; bỏ qua khác biệt về trọng số):

| Cơ cấu | NWRMSLE ước |
|---|---|
| V1 (đúng như đã đo) | 0,639 |
| Chỉ cặp cũ nhưng theo tỷ lệ của test (90,1% đang bán, 9,9% ngừng bán) | **≈ 0,651** |
| Thêm 21,1% cặp mới, sai số bình phương bằng 1 lần cặp đang bán | ≈ 0,658 |
| Như trên, 1,5 lần | ≈ 0,694 |
| Như trên, 2 lần | ≈ 0,729 |

**Giả định chưa kiểm chứng:** sai số của cặp mới. Đây là phần lớn nhất chưa biết. Vì vậy cần một cách đo riêng cho cặp mới (điều kiện 4), không thể suy ra từ V1.

Hệ quả thực tế: mô hình tốt hơn baseline cần so sánh trên **cùng cơ cấu**; một mức cải thiện nhỏ trên V1 có thể bị nuốt bởi sai số của cặp mới.

---

## 6. Rủi ro và điều kiện chi tiết cho bước 04

| # | Mức độ | Vấn đề | Điều kiện / việc cần làm |
|---|---|---|---|
| 1 | **Cao** | **Cặp mới không có trong dữ liệu huấn luyện.** Tập huấn luyện chỉ chứa cặp đã có lịch sử, nên các feature lịch sử **không có giá trị thiếu** trong huấn luyện. Khi dự báo cặp mới (feature NaN), LightGBM mặc định coi giá trị thiếu ở feature chưa từng thiếu khi huấn luyện như 0 [Suy luận], và 0 ở `pair_age_days` nghĩa là "cặp vừa ra mắt" chứ không phải "chưa có lịch sử". Kết quả không đáng tin | Định tuyến theo `pair_status`: trạng thái 0 và 1 vào mô hình chính; trạng thái 2 và 3 dùng quy tắc/ mô hình riêng dựa trên `item_*`, `store_*`, `family_store_mean_28`, `class_mean_28`. Kiểm chứng bằng cách dự báo thử |
| 2 | **Cao** | **Rò rỉ nhãn qua khuyến mãi tương lai** (`promo_t`, `promo_h_sum`, `item_promo_share_t`, `store_promo_share_t`) | Huấn luyện hai bản: có và không có nhóm này; coi mức cải thiện trên kiểm định là cận trên. Ưu tiên `promo_days_*` và `promo_share_sale_days_28` (tính tại mốc, an toàn) |
| 3 | **Cao** | **Đánh giá chưa phản ánh test**: V1 không có cặp mới, có nhiều cặp ngừng bán hơn test | Báo cáo theo `pair_status`; thêm ước lượng theo cơ cấu test (mục 5); đo cặp mới bằng kiểm định giả lập: lấy ngẫu nhiên khoảng 20% cặp ở V1, che lịch sử của chúng (đặt feature lịch sử là NaN, tính lại feature phân cấp không có chúng), rồi dự báo bằng quy tắc cho cặp mới |
| 4 | Trung bình | **Giá dầu** (`oil_T`, `oil_chg_28`) là hằng số theo mốc: với 54 mốc huấn luyện, mô hình có thể dùng chúng để "nhận ra" mốc và học hiệu ứng thời kỳ không tổng quát hóa. Notebook 01 cũng cho thấy tương quan giá dầu là giả | Loại khỏi mô hình đầu tiên; chỉ thử lại như một thí nghiệm riêng |
| 5 | Trung bình | **RAM**: 2,97 triệu dòng × 75 feature ≈ 0,83 GB, chưa tính bản nhị phân hóa của LightGBM, so với 1,33 GB khả dụng | Phương án đã đo: từ 2016-01-01 → 2,21 triệu dòng (≈ 0,51 GB với 62 feature); từ 2016-07-01 → 1,78 triệu (≈ 0,41 GB); từ 2017-01-01 → 0,99 triệu (≈ 0,23 GB). Loại thêm nhóm rủi ro, nạp trực tiếp thành mảng float32, đóng ứng dụng khác. Đây cũng là thí nghiệm cửa sổ huấn luyện mà notebook 01 đã đề xuất |
| 6 | Trung bình | **Một mốc kiểm định duy nhất** (V1) cho nhiều nhiễu; chưa có baseline cho V2 | Tính baseline trên cả V2; dùng V1 và V2 cùng lúc để so sánh mô hình; tránh tối ưu siêu tham số sát V1 |
| 7 | Trung bình | **Các mốc huấn luyện chồng lấn**: các dòng không độc lập | Chia fold và chọn siêu tham số theo mốc, không theo dòng; dùng quy tắc `T_tr + 16 ≤ T_F` đã lưu trong `03_config.json` |
| 8 | Trung bình | `store_promo_share_t` có PSI 0,944 chưa giải thích được (trung bình giống nhau: 0,0570 so với 0,0591) | Vẽ phân phối theo vai trò trước khi dùng. [Suy luận] Chỉ có 54 cửa hàng nên phân phối rời rạc và phụ thuộc cách chia nhóm, nhưng chưa loại trừ khác biệt thật |
| 9 | Thấp | `promo_known_t` hằng số (luôn 1 với mọi mốc từ 2015-06) | Bỏ khỏi mô hình |
| 10 | Thấp | Các feature chỉ khác 0 ở cuối năm (`is_jan1`, `is_dec24_31`, `days_to_xmas`) không có tác dụng ở test (ngày 16–31/08) | Có thể giữ (vô hại) hoặc bỏ để giảm bộ nhớ |
| 11 | Thấp | DL08: outlier chưa xử lý | `log1p` đã giảm ảnh hưởng; thử cắt theo item/family nếu sai số bị chi phối bởi giá trị cực lớn |
| 12 | Thấp | Các biểu đồ nhỏ ở 01 có nhãn trục chồng nhau | Chỉ ảnh hưởng hiển thị |

---

## 7. Kế hoạch đề xuất cho bước 04

Thứ tự gợi ý, mỗi bước có tiêu chí xong:

| Bước | Việc | Xong khi |
|---|---|---|
| 04.1 | Tính lại các baseline luật trên **V1 và V2**, theo `pair_status`, `perishable`, `h` | Có bảng baseline cho cả hai fold; chọn baseline luật tốt nhất mỗi fold |
| 04.2 | Baseline cho cặp mới theo luật phân cấp (ví dụ trung bình item hoặc cửa hàng); đo bằng kiểm định giả lập ở điều kiện 3 | Có sai số ước lượng cho `pair_status` 2 và 3, cập nhật bảng ở mục 5 |
| 04.3 | Mô hình LightGBM đầu tiên cho cặp có lịch sử: `y_log`, trọng số `w`, nhóm feature "an toàn" (lịch sử, cùng thứ, phân cấp, lịch, metadata, khuyến mãi lịch sử), **không** có khuyến mãi tương lai, **không** có giá dầu | Mô hình vượt baseline tốt nhất trên cả V1 và V2 |
| 04.4 | Thí nghiệm ablation: thêm nhóm *khuyến mãi tương lai*; thêm giá dầu; đổi cửa sổ huấn luyện (toàn bộ, từ 2016, từ 2016-07) | Bảng so sánh NWRMSLE từng cấu hình, ghi rõ bản nào dùng nhóm rủi ro |
| 04.5 | Bản nộp thử v0: mô hình chính cho cặp cũ, luật cho cặp mới; ghi file theo thứ tự `id` của `sample_submission.csv` (`y = expm1(dự báo log)`, cắt ≥ 0) | File nộp hợp lệ, số dòng và `id` khớp 3.370.464 |

**Mốc cần vượt:** NWRMSLE **0,6386** (V1, cặp có lịch sử) hoặc ≈ **0,651** khi quy về tỷ lệ cặp của test (mục 5). Mô hình cho kết quả không tốt hơn mốc này thì chưa có giá trị.

**Quy ước ghi kết quả ở 04:** mỗi con số cần ghi rõ fold, nhóm cặp và cấu hình có hay không có nhóm khuyến mãi tương lai.

---

## 8. Phụ lục: tham số và đường dẫn cần dùng ở 04

Từ `03_config.json`:

- `horizon = 16`, `lookback = 224`, `origin_step_days = 13`, `seed = 42`; nhãn `y_log = log1p(clip(unit_sales, 0))`, trọng số `w` (perishable 1,25).
- Fold: **V1_last16** (mốc 2017-07-30) và **V2_seasonal** (mốc 2016-08-15). Quy tắc: mốc huấn luyện `T_tr` dùng cho fold `F` khi `T_tr + 16 ≤ T_F`. V1 có 53 mốc huấn luyện (2.905.662 dòng); V2 có 26 mốc (1.301.649 dòng).
- `baseline_best`: `B2c mean_28`, NWRMSLE 0,63859.
- Nhóm feature: lịch sử cặp (20), cùng thứ trong tuần (6), phân cấp/cửa hàng (13), lịch/ngày lễ (17), metadata (7), khuyến mãi lịch sử (3), **khuyến mãi tương lai (5, rủi ro)**, exploratory (2), trạng thái cặp (2).

Đường dẫn: `data/processed/features/<role>_<YYYYMMDD>.parquet` với `role` ∈ `train`, `valid_last16`, `valid_seasonal`, `test`. Hàm đọc dữ liệu: `load_features(roles, columns, origin_min, origin_max, max_rows)` trong notebook 03.

Báo cáo `03_feature_engineering_report.json` chứa: manifest (57 file), kết quả kiểm tra cấu trúc, sai khác brute-force, nhãn theo vai trò, cơ cấu cặp của test, baseline, PSI, tóm tắt fold và ước tính bộ nhớ huấn luyện (0,83 GB).

---

## 9. Phạm vi kiểm chứng của tài liệu này

- Mọi con số **[Đo]** lấy từ output đã chạy của bạn. Tôi không có các file Parquet đặc trưng nên không tự chạy lại phép đo nào trên dữ liệu thật.
- Các con số **[Ước]** (mục 5) phụ thuộc vào giả định nêu kèm; chúng là kịch bản để định hướng, không phải kết quả.
- Điều 1 ở mục 6 về cách LightGBM xử lý giá trị thiếu là suy luận từ hành vi mặc định, chưa được thử trên dữ liệu của bạn.
- Lý do `store_promo_share_t` có PSI cao chưa được xác định.
