# Báo cáo: Những gì đã phát hiện được và hướng xử lý tiếp theo

**Phạm vi:** tổng hợp từ 3 notebook `00_scope_and_enviroment`, `01_eda`, `02_data_cleaning`.
**Cách đọc:** viết cho người mới học ML. Từ khó có giải thích trong bảng thuật ngữ của `PROJECT_GUIDE.md`.
**Lưu ý trung thực:** các con số bên dưới lấy từ kết quả đã chạy trong notebook của bạn. Phần nào là *nhận định của mình* (chưa được notebook chứng minh) thì mình ghi rõ là "cần kiểm tra".

---

## 1. Tóm tắt trong 1 phút

- Dự án dự báo `unit_sales` (số lượng bán) cho **16 ngày** (2017-08-16 → 2017-08-31) cho từng cặp cửa hàng – mặt hàng.
- Dữ liệu gồm **125,5 triệu dòng** lịch sử bán hàng (2013-01-01 → 2017-08-15), 54 cửa hàng, ~4.000 mặt hàng.
- **Dữ liệu khá sạch:** không trùng dòng, không lỗi ngày, không khóa tham chiếu bị sai. Vấn đề chính là `onpromotion` thiếu 17,26% và có 7.795 dòng doanh số âm.
- **Thử thách lớn nhất là tài nguyên máy, không phải chất lượng dữ liệu:** `train` đầy đủ ước tính ~6,8 GB trong RAM, trong khi máy chỉ còn ~2–2,7 GB trống.
- **Việc đã xong:** hiểu bài toán, khảo sát dữ liệu, làm sạch và lưu thành file Parquet (số dòng khớp 100%).
- **Bước tiếp theo quan trọng nhất:** chốt vài quyết định (xem mục 6), cắt dữ liệu về khoảng thời gian gần đây cho vừa RAM, rồi tạo feature **không rò rỉ tương lai**.

---

## 2. Những gì đã làm được

| Notebook | Đã làm | Kết quả chính |
|---|---|---|
| **00 – Scope & Environment** | Xác định bài toán, đo máy, thống kê 8 file | Bài toán = dự báo chuỗi thời gian; máy 8 nhân, 7,5 GB RAM, không GPU; cần đọc `train` theo chunk |
| **01 – EDA** | Kiểm tra schema, thiếu, trùng, khóa, thời gian, target; vẽ biểu đồ | Dữ liệu hợp lệ; thấy xu hướng tăng và chu kỳ theo tuần |
| **02 – Data Cleaning** | Chuẩn hóa kiểu dữ liệu, chứng minh không trùng, ghi Parquet, kiểm tra đầu ra | 7 file `*_cleaned.parquet`, số dòng vào = ra, báo cáo JSON đã lưu |

Điểm tốt của cách làm hiện tại (nên giữ):

- Không sửa `data/raw/`.
- Không xóa hay điền dữ liệu một cách vội vàng; mỗi quyết định có chính sách ghi rõ.
- Có "cổng kiểm tra" dừng lại khi dữ liệu có vấn đề cấu trúc.
- Ghi file qua file tạm rồi mới đổi tên, nên lỗi giữa chừng không để lại file hỏng.

---

## 3. Những phát hiện về dữ liệu

### 3.1 Các bảng dữ liệu

| File | Số dòng | Nội dung | Ghi chú |
|---|---:|---|---|
| `train.csv` | 125.497.040 | Doanh số theo ngày × cửa hàng × mặt hàng | ~4,65 GB (CSV); ~1,08 GB (Parquet) |
| `test.csv` | 3.370.464 | Các dòng cần dự đoán, 16 ngày | Có `onpromotion`, không thiếu |
| `sample_submission.csv` | 3.370464 | Mẫu file nộp | Cột `id`, `unit_sales` |
| `transactions.csv` | 83.488 | Số giao dịch của cửa hàng theo ngày | Chỉ đến 2017-08-15 |
| `items.csv` | 4.100 | `family`, `class`, `perishable` | Thông tin mặt hàng |
| `oil.csv` | 1.218 | Giá dầu theo ngày | Thiếu 43 ngày (3,53%) |
| `holidays_events.csv` | 350 | Ngày lễ và sự kiện | 312 ngày khác nhau (một ngày có thể có nhiều dòng) |
| `stores.csv` | 54 | `city`, `state`, `type`, `cluster` | Thông tin cửa hàng |

### 3.2 Chất lượng dữ liệu

| Hạng mục | Kết quả | Đánh giá |
|---|---|---|
| Cột bắt buộc | Đủ ở cả 7 bảng | Tốt |
| Ngày không hợp lệ | 0 | Tốt |
| `store_nbr`, `item_nbr` không có trong bảng tham chiếu | 0 | Tốt |
| Dòng trùng theo khóa `(date, store_nbr, item_nbr)` | 0, đã chứng minh toàn cục (dữ liệu sắp xếp theo khóa) | Tốt |
| `onpromotion` thiếu trong `train` | 21.657.651 dòng (17,26%) | **Cần xử lý** |
| `onpromotion` thiếu trong `test` | 0 | Tốt |
| `unit_sales` âm | 7.795 dòng (0,006%) | Cần quyết định |
| `unit_sales = 0` | 0 dòng | **Đáng chú ý** (xem 3.3) |
| Giá dầu thiếu | 43 / 1.218 | Nhỏ, xử lý được |
| Ngày thiếu trong lịch | 4 ngày, đều là 25/12 mỗi năm (2013–2016) | Bình thường (cửa hàng nghỉ) |

### 3.3 Phát hiện đáng chú ý về target `unit_sales`

- **Trung bình ≈ 8,55, độ lệch chuẩn ≈ 23,6, nhỏ nhất −15.372, lớn nhất 89.440.** Nghĩa là phần lớn dòng bán ít, nhưng có vài dòng bán rất nhiều.
- **Phân phối lệch mạnh về bên phải** (biểu đồ gần như chỉ có một cột sát số 0, đuôi dài tới hàng nghìn). Học trực tiếp trên số gốc sẽ bị vài giá trị lớn chi phối; nên học trên `log1p`.
- **Không có dòng nào bằng 0.** Điều này gợi ý rằng `train` chỉ ghi những lần **có bán**; khi không bán thì không có dòng. Mình thử tính: nếu mọi tổ hợp đều có dòng thì sẽ có khoảng 54 × 4.036 × 1.684 ≈ 367 triệu dòng, nhưng thực tế chỉ có 125,5 triệu (~34%). *Đây là nhận định cần kiểm tra thêm*, vì phần chênh lệch có thể gồm cả cửa hàng chưa mở hoặc mặt hàng chưa ra mắt. Dù lý do nào, nó ảnh hưởng trực tiếp đến cách tính trung bình trượt (xem mục 6).
- **Doanh số âm** rất ít (0,006%), nhiều khả năng là hàng trả lại. Với mục tiêu dự đoán lượng bán, hợp lý là cắt về 0 khi tạo target học.
- **Sai số của phân vị:** các phân vị trong notebook (trung vị 4, 75% là 9, 90% là 19, 99% là 66) được tính từ mẫu **chỉ lấy ở 20 chunk đầu** (~2 triệu dòng đầu, tức các ngày đầu năm 2013), nên chỉ mang tính tham khảo. Cần tính lại bằng mẫu ngẫu nhiên từ toàn bộ file.

### 3.4 Phát hiện từ biểu đồ

| Chủ đề | Quan sát | Ý nghĩa cho bước sau |
|---|---|---|
| **Xu hướng** | Tổng doanh số mỗi ngày tăng dần: khoảng 0,3–0,5 triệu/ngày (2013) lên khoảng 0,7–1,2 triệu/ngày (2016–2017). Có vẻ có bước tăng khoảng giữa năm 2015 | Dùng dữ liệu **gần đây** để học; dữ liệu 2013 phản ánh quy mô cũ |
| **Chu kỳ theo tuần** | Rất rõ. **Chủ nhật cao nhất** (~820 nghìn/ngày), **thứ Bảy** (~770 nghìn), **thứ Năm thấp nhất** (~505 nghìn) | Feature "thứ trong tuần" gần như chắc chắn quan trọng |
| **Ngày đầu năm** | Doanh số rơi gần 0 vào ngày 1/1 mỗi năm; cuối năm thường có đỉnh | Cần feature ngày lễ và cuối năm; ngày 1/1 là ngoại lệ |
| **Cửa hàng** | Doanh số tập trung vào một số cửa hàng (44, 45, 47, 3, 49 đứng đầu; cửa hàng 44 ≈ 62 triệu) | Thông tin cửa hàng (`cluster`, `type`) và lịch sử theo cửa hàng sẽ hữu ích |
| **Mặt hàng** | Một số mặt hàng bán vượt trội (item 1503844 ≈ 6,26 triệu) và có đuôi dài các mặt hàng bán ít | Mặt hàng bán ít khó dự đoán; cần thước đo theo log để không bị mặt hàng lớn lấn át |
| **Khuyến mãi** | Có khuyến mãi: trung bình **13,47** / dòng; không khuyến mãi: **8,09** / dòng | Cho thấy có liên quan, **chưa chứng minh khuyến mãi là nguyên nhân** (có thể mặt hàng bán chạy mới được khuyến mãi) |

### 3.5 Bảng ngày lễ

- 350 dòng: `Holiday` 221, `Event` 56, `Additional` 51, `Transfer` 12, `Bridge` 5, `Work Day` 5.
- Phạm vi: `National` 174, `Local` 152, `Regional` 24.
- 12 dòng `transferred = True` (ngày lễ bị dời sang ngày khác).
- Hệ quả: phải xử lý có quy tắc (ngày lễ bị dời, ngày lễ chỉ áp dụng cho một thành phố/tỉnh), không thể đánh dấu "là ngày lễ" một cách đơn giản.

---

## 4. Những điểm cần lưu ý và rủi ro

### 4.1 Rủi ro lớn (ảnh hưởng đến kết quả)

| # | Rủi ro | Vì sao quan trọng | Cách xử lý đề xuất |
|---|---|---|---|
| R1 | **Thiếu RAM** | `train` đầy đủ ~6,8 GB (ước tính), máy còn ~2–2,7 GB | Cắt khoảng thời gian gần đây, ép `float32`, lọc Parquet khi đọc, cân nhắc DuckDB |
| R2 | **Rò rỉ tương lai (leakage)** | Test cách ngày cuối của train 1–16 ngày; feature lịch sử dùng dữ liệu quá gần sẽ không có thật khi dự đoán | Mọi feature doanh số quá khứ dùng dữ liệu cách ≥ 16 ngày |
| R3 | **Tương quan giá dầu −0,705 gây hiểu lầm** | Cả giá dầu và doanh số đều có xu hướng theo năm (giá dầu giảm, doanh số tăng) nên trông như có quan hệ, nhưng có thể chỉ là trùng hợp của hai xu hướng | Không dùng con số đó làm bằng chứng; thử feature giá dầu bằng thực nghiệm |
| R4 | **`transactions` không có cho giai đoạn test** | File kết thúc 2017-08-15 | Không dùng số giao dịch cùng ngày; ở vòng đầu bỏ qua bảng này |
| R5 | **Dòng "không có dữ liệu" bị hiểu sai** | Nếu coi ngày không có dòng là "không tính", trung bình trượt sẽ cao hơn thực tế | Thử cả hai cách (coi là 0 / bỏ qua) và so sánh bằng validation |
| R6 | **Mặt hàng/cặp mới trong test** | Không có lịch sử nên dự đoán kém | Kiểm tra số lượng; dùng giá trị trung bình theo `family` + cửa hàng |

### 4.2 Điểm cần sửa trong các notebook (nhỏ nhưng nên làm)

1. **Notebook 01, mục 9.1–9.4 còn trống** ("Cập nhật sau"). Nên điền bằng nội dung ở mục 3 của báo cáo này.
2. **Mẫu để vẽ phân phối và tính phân vị bị lệch** (chỉ 20 chunk đầu, xem 3.3).
3. **Notebook 01 viết "target là `sales`"** trong phần mô tả, nhưng tên cột thật là `unit_sales`. Nên thống nhất.
4. **Notebook 00 có hai ghi chú chưa khớp thực tế:** `transactions.csv` chỉ ~1,5 MB (không cần chunk); mục "tối ưu dtype → không" mâu thuẫn với notebook 02 (đã ép `int32`).
5. **Tên file `enviroment`** (thiếu `n`) và notebook 02 nhắc `environment`. Nên đổi cho thống nhất.
6. **Notebook 01 kiểm tra trùng theo dòng, theo chunk** (chưa đủ chứng minh). Điều này đã được notebook 02 bù lại bằng kiểm tra khóa toàn cục, nên không còn là vấn đề; chỉ cần ghi chú để tránh hiểu nhầm.
7. **Báo cáo `docs/data_understanding_cleaning_report.md`** đã có trong dự án (mình chưa đọc). Nên đối chiếu để hai báo cáo không mâu thuẫn.
8. **Các thư mục `src/`, `configs/`, `tests/`** chưa thấy nội dung được dùng. Mình giả định đang trống; nên chuyển dần code ổn định từ notebook sang đó.

---

## 5. Cách bài toán này nên được "đặt" cho đúng

Đây là điểm dễ sai nhất với người mới, nên mình nói riêng.

- Bạn cần dự đoán **16 ngày liên tiếp** sau ngày 15/08/2017.
- Với ngày 16/08, bạn biết dữ liệu đến 15/08 (cách 1 ngày). Với ngày 31/08, bạn cũng chỉ biết dữ liệu đến 15/08 (cách 16 ngày).
- Nếu tạo feature như "doanh số hôm qua" cho mọi dòng, mô hình sẽ học rất tốt khi thử (vì hôm qua luôn có sẵn), nhưng khi dự đoán thật cho ngày 31/08 thì "hôm qua" (30/08) **không tồn tại**.
- **Cách đơn giản và an toàn:** chỉ dùng thông tin cách ≥ 16 ngày trước ngày cần đoán, cho **cả** lúc học, lúc kiểm tra và lúc dự đoán. Hơi "thiệt" một chút về độ chính xác của ngày gần, nhưng đúng với tình huống thật.

---

## 6. Đề xuất hướng xử lý tiếp theo

### 6.1 Các quyết định cần chốt (kèm đề xuất)

| Vấn đề | Đề xuất | Lý do | Cách kiểm chứng |
|---|---|---|---|
| Khoảng thời gian dùng để học | Bắt đầu từ **2016-08-01** (hoặc 2017-01-01 nếu thiếu RAM) | Vừa RAM; dữ liệu gần phản ánh đúng hiện tại; tránh vùng `onpromotion` thiếu (cần xác nhận) | So điểm validation khi mở rộng khoảng thời gian |
| Doanh số âm | Cắt về 0 khi tạo target học; không sửa file cleaned | Chỉ 0,006%; lượng bán không âm | Điểm validation không đổi đáng kể |
| `onpromotion` thiếu | Đếm thiếu theo tháng. Nếu tập trung ở giai đoạn đầu thì bỏ giai đoạn đó. Nếu còn thiếu thì tạo cột "đã thiếu" | Không tự giả định thiếu = `False` | Biểu đồ thiếu theo thời gian |
| Ngày không có dòng | Thử cả "coi là 0" và "bỏ qua" | Chưa chắc ý nghĩa thật của dòng vắng mặt | So điểm validation hai cách |
| Biến đổi target | Học trên `log1p`, dự đoán rồi `expm1` | Phân phối rất lệch | So điểm có và không `log1p` |
| Cách chia dữ liệu | Validation = 16 ngày cuối (2017-07-31 → 2017-08-15) | Giống đúng độ dài của test | Kiểm tra ngày train < ngày validation |
| Thước đo | NWRMSLE (nếu theo cuộc thi gốc) | Dựa trên log, hàng dễ hỏng nặng hơn | **Hỏi lại đề bài/giảng viên** |
| Giá dầu | Forward-fill, thử có/không | Tương quan hiện có đáng ngờ | So điểm validation |
| Bảng `transactions` | Bỏ ở vòng đầu | Không có cho giai đoạn test | Thử lag ≥ 16 ngày ở vòng sau (tùy chọn) |

### 6.2 Thứ tự ưu tiên

**Ưu tiên 1 — Làm ngay (hoàn thiện nền móng)**
1. Điền kết luận còn trống ở notebook 01 và sửa mẫu lệch.
2. Kiểm tra thêm: `onpromotion` thiếu vào thời gian nào; cặp mới trong `test`; tỷ lệ tổ hợp có dòng.
3. Kiểm tra `requirements.txt` có `lightgbm`, `pyarrow`, `scikit-learn`.

**Ưu tiên 2 — Chuẩn bị dữ liệu làm việc**
4. Cắt dữ liệu gần đây và lưu `data/interim/train_recent.parquet`.
5. Đo RAM sau khi nạp để biết còn bao nhiêu chỗ.

**Ưu tiên 3 — Validation và baseline (trước khi làm feature phức tạp)**
6. Chia theo thời gian, viết hàm thước đo.
7. Tính 2 baseline đơn giản. Đây là mốc để biết mô hình có "đáng công" hay không.

**Ưu tiên 4 — Feature, rồi mô hình**
8. Tạo feature theo từng nhóm, thêm một nhóm thì chấm điểm lại một lần (nhóm lịch → khuyến mãi → thông tin cửa hàng/mặt hàng → lịch sử bán → ngày lễ → giá dầu).
9. Huấn luyện LightGBM trên mẫu nhỏ, rồi tăng dần.
10. Đánh giá chi tiết, viết báo cáo cuối.

> Vì sao làm baseline **trước** feature: nếu baseline đã khá tốt thì ta biết cần nhắm đến mức nào; và nếu mô hình ML không thắng nổi baseline thì có thể đang có lỗi (thường là lỗi feature hoặc leakage).

### 6.3 Khoảng điểm nên đặt kỳ vọng

Mình **không** nêu con số điểm mục tiêu vì notebook chưa có kết quả mô hình hay baseline nào. Hãy chạy baseline trước, rồi đặt mục tiêu theo mức đó (ví dụ: "giảm sai số so với baseline tốt nhất").

---

## 7. Câu hỏi cần bạn xác nhận

1. **Thước đo đánh giá** của đề bài là gì? (NWRMSLE giống cuộc thi Kaggle, hay RMSE/MAE thông thường?)
2. **Dự án có bắt buộc nộp file dự đoán cho `test`** không, hay chỉ cần đánh giá trên validation?
3. **Yêu cầu của môn học:** có cần so sánh nhiều mô hình không (ví dụ tối thiểu 3)? Nếu có, kế hoạch ở `PROJECT_GUIDE.md` đã gồm baseline, Ridge và LightGBM.
4. **Nội dung hiện có** của `src/`, `configs/`, `tests/` và `docs/data_understanding_cleaning_report.md`: nếu bạn gửi, mình có thể đối chiếu và chỉnh lại checklist cho khớp.

---

## 8. Kết luận

Phần khó của giai đoạn đầu (hiểu dữ liệu, làm sạch an toàn, chứng minh không trùng) đã được làm cẩn thận. Dữ liệu đủ tốt để mô hình hóa. Hai việc quyết định thành bại ở giai đoạn sau là: **(1) giữ dữ liệu vừa RAM** và **(2) không để feature nhìn trộm tương lai**. Nếu làm đúng hai việc này và luôn so sánh với baseline, dự án có nền tảng vững để đi tiếp.
