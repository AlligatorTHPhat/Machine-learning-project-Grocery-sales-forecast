# Báo cáo tổng hợp Data Scope, Validation, EDA và Cleaning

**Dự án:** Machine Learning Final Project — Favorita Grocery Sales Forecasting  
**Phạm vi:** Tổng hợp kết quả đã ghi trong `00_scope_and_enviroment.ipynb`, `01_eda.ipynb` và `02_data_cleaning.ipynb`.  
**Trạng thái:** Đã hoàn tất khảo sát dữ liệu và bước chuẩn hóa ban đầu; sẵn sàng làm cơ sở cho `03_feature_engineering.ipynb` sau khi lưu ý các điểm cần theo dõi trong báo cáo này.

> **Quy ước về độ tin cậy:** Các số liệu dưới đây được lấy từ output đã lưu trong ba notebook được cung cấp. Những kết luận chưa có output trực tiếp hoặc chưa được diễn giải trong notebook sẽ được ghi rõ là giới hạn/công việc tiếp theo, không được xem như kết quả đã xác nhận.

## 1. Tóm tắt điều hành

1. Bộ dữ liệu gồm **8 file CSV**, trong đó `train.csv` là file lớn nhất: khoảng **4,765.94 MB** và **125,497,040 dòng**.
2. Đây là bài toán dự báo chuỗi thời gian. Tên target đúng theo dataset đang dùng là **`unit_sales`**, không phải `sales`.
3. Các vấn đề chất lượng dữ liệu chính được xác nhận là missing ở `train.onpromotion` (**21,657,651 dòng; 17.2575%**) và `oil.dcoilwtico` (**43/1,218 dòng; 3.5304%**), cùng **7,795 giá trị `unit_sales` âm** (**0.0062%**).
4. Notebook `02` đã chạy qua validation gate. Kiểm tra báo cáo cho thấy không có ngày/ID/khóa store-item không hợp lệ, không có token promotion không nhận diện được và không phát hiện duplicate của khóa nghiệp vụ `(date, store_nbr, item_nbr)` ở train/test theo phương pháp kiểm tra có điều kiện về thứ tự dữ liệu.
5. Bước `02` **không xóa dòng, không tự điền missing, không xóa target âm và không tự thêm các ngày thiếu**. Tất cả 7 bảng được xuất đều giữ nguyên số dòng đầu vào.
6. Dữ liệu đã được chuẩn hóa và ghi sang `data/processed/` dưới dạng Parquet. `data/raw/` được giữ làm nguồn đầu vào không biến đổi.

## 2. Phạm vi bài toán và quy ước dữ liệu

- **Bài toán:** Forecasting doanh số theo thời gian.
- **Target chuẩn:** `unit_sales` trong `train.csv`.
- **Khóa quan sát ở cấp bản ghi bán hàng:** `(date, store_nbr, item_nbr)`.
- **Mục tiêu của bước 00–02:** kiểm kê và đánh giá dữ liệu, phát hiện rủi ro, chuẩn hóa cách biểu diễn và tạo đầu ra ổn định; chưa thực hiện feature engineering, imputation theo thời gian hoặc huấn luyện mô hình.
- **Nguyên tắc bảo toàn:** không sửa trực tiếp file trong `data/raw/`; các phép biến đổi được ghi sang `data/processed/`.

### Lưu ý về tính nhất quán tên target

Notebook `00` vẫn còn `TARGET_COLUMN = "sales"` và một số ghi chú ở `01` vẫn dùng từ “sales” như tên cột. Tuy nhiên, code chạy thành công trong `01` và schema đầy đủ ở `02` đều xác nhận cột dữ liệu thực tế là **`unit_sales`**. Cần cập nhật cấu hình/diễn giải còn sót trong `00` và Markdown của `01` để tránh nhầm lẫn ở các notebook sau.

## 3. Môi trường thực thi và giới hạn tài nguyên

Các output ghi nhận được:

| Thuộc tính | Giá trị ghi nhận |
|---|---:|
| Hệ điều hành | Linux x86_64 (`Linux-7.0.0-30-generic`) |
| Python | 3.11.16 |
| CPU | 8 logical CPU cores |
| RAM tổng | Khoảng 7.50 GiB |
| GPU NVIDIA | Không phát hiện |
| Dung lượng đĩa trống tại lần đo trong `00` | Khoảng 101.70 GiB |
| Chunk size dùng trong EDA/cleaning | 100,000 dòng |
| Ước tính bộ nhớ DataFrame đầy đủ của `train.csv` trong `00` | Khoảng 6.78 GB |

RAM khả dụng thay đổi theo thời điểm chạy: notebook ghi nhận khoảng 1.64 GiB, 3.02 GiB và 3.62 GiB ở những lần đo khác nhau. Vì vậy, không nên coi một con số RAM khả dụng là hằng số. Ước tính 6.78 GB cho DataFrame train đầy đủ, cộng với các intermediate copies và tiến trình khác, cho thấy **không nên nạp toàn bộ `train.csv` vào RAM**. Chiến lược chunk processing được sử dụng trong `01` và `02` là phù hợp với môi trường hiện tại.

## 4. Kiểm kê dataset

| File | Số dòng | Số cột | Kích thước gần đúng trong `00` | Vai trò / ghi chú |
|---|---:|---:|---:|---|
| `train.csv` | 125,497,040 | 6 | 4,765.94 MB | Dữ liệu train, target `unit_sales` |
| `test.csv` | 3,370,464 | 5 | 120.32 MB | Dữ liệu cần dự báo |
| `sample_submission.csv` | 3,370,464 | 2 | 38.57 MB | Mẫu định dạng submission |
| `transactions.csv` | 83,488 | 3 | 1.48 MB | Transactions theo ngày và store |
| `items.csv` | 4,100 | 4 | 0.097 MB | Metadata item |
| `holidays_events.csv` | 350 | 6 | 0.021 MB | Holiday/event, một ngày có thể có nhiều event |
| `oil.csv` | 1,218 | 2 | 0.020 MB | Giá dầu theo ngày |
| `stores.csv` | 54 | 5 | 0.001 MB | Metadata store |

Tổng cộng **8 file CSV**. `02_data_cleaning` đã chuẩn hóa 7 file dữ liệu chính (train, test và 5 bảng tham chiếu); `sample_submission.csv` chưa được xuất thành file `_cleaned` trong bước này và nên được giữ làm tham chiếu cho bước tạo submission cuối cùng.

### Schema

Bảng schema trong `01` đánh dấu một số cột thành `extra_columns` vì cấu hình lúc đó chỉ khai báo các cột tối thiểu cần kiểm tra. Những trường như `id`, `city`, `cluster`, `state`, `type`, `family`, `class`, `perishable`, `dcoilwtico`, `description`, `locale`, `locale_name`, `transferred` và `transactions` là cột hợp lệ của dataset, **không phải bằng chứng rằng raw data có lỗi**.

`02` sử dụng schema đầy đủ cho 7 file được xử lý. Kết quả cho thấy mỗi file có đủ cột yêu cầu, không có `extra_columns`, và trạng thái đều là `OK`.

## 5. Phát hiện về chất lượng dữ liệu từ `01` và raw scan trong `02`

### 5.1 Missing values

| File / cột | Missing | Tỷ lệ | Nhận định |
|---|---:|---:|---|
| `train.csv: onpromotion` | 21,657,651 | 17.257499% | Missing đáng kể; không tự suy ra là `False` |
| `train.csv: unit_sales` | 0 | 0% | Không có missing target được phát hiện |
| `train.csv: id`, `date`, `store_nbr`, `item_nbr` | 0 ở từng cột | 0% | Không ghi nhận missing ở các trường này |
| `test.csv` — tất cả 5 cột | 0 ở từng cột | 0% | Không ghi nhận missing |
| `oil.csv: dcoilwtico` | 43/1,218 | 3.530378% | Cần chính sách xử lý chuỗi thời gian riêng |
| `stores.csv`, `items.csv`, `holidays_events.csv`, `transactions.csv` | 0 ở từng cột | 0% | Không ghi nhận missing ở các bảng này |

### 5.2 Giá trị target và trường khóa

Các thống kê tính trên toàn bộ `train.csv` trong `01`:

| Chỉ số | Kết quả |
|---|---:|
| Số target không missing | 125,497,040 |
| Mean `unit_sales` | 8.554865 |
| Standard deviation | 23.60515 |
| Minimum | -15,372 |
| Maximum | 89,440 |
| Số `unit_sales == 0` | 0 |
| Số `unit_sales < 0` | 7,795 (0.006211%) |
| `store_nbr` không dương | 0 |
| `item_nbr` không dương | 0 |

`02` xác nhận thêm: không có `unit_sales` không chuyển được sang số, không có ngày/ID/store/item không hợp lệ, không có giá trị promotion khác các biểu diễn True/False được chấp nhận và không có sai lệch tham chiếu store/item.

**Quyết định đã áp dụng:** giữ nguyên target âm. Các giá trị âm hiếm có thể phản ánh trả hàng hoặc điều chỉnh; chỉ dựa vào dấu âm là chưa đủ để loại bỏ chúng. Cũng không tự coi target bằng 0 là invalid, dù bộ train hiện tại không phát hiện dòng nào có target đúng bằng 0.

> **Giới hạn của quantile trong `01`:** các quantile được tính từ mẫu bounded, không phải thống kê exact trên toàn bộ tập. Cách lấy mẫu hiện tại lấy mẫu từ 20 chunk đầu tiên, nên có thể thiên lệch theo thời gian nếu dữ liệu được sắp theo thứ tự ngày. Các quantile này chỉ nên xem là tham khảo sơ bộ và nên được tính lại bằng mẫu trải đều trên toàn bộ train trước khi ra quyết định về target transformation hoặc clipping.

### 5.3 Duplicates và tính duy nhất của khóa nghiệp vụ

- Trong `01`, kiểm tra duplicate train chỉ xác nhận **0 duplicate trong từng chunk**; bản thân kết quả này chưa đủ kết luận về duplicate xuyên qua ranh giới chunk.
- Trong `02`, raw scan ghi nhận train/test đều có `business_key_order_is_monotonic = True`, 0 duplicate trong chunk, 0 duplicate tại ranh giới chunk và 0 duplicate của khóa nghiệp vụ theo kiểm tra tổng thể có điều kiện đó.
- Các khóa duy nhất của bảng tham chiếu cũng không có duplicate:
  - `stores.csv`: `store_nbr` — 0 duplicate.
  - `items.csv`: `item_nbr` — 0 duplicate.
  - `oil.csv`: `date` — 0 duplicate.
  - `transactions.csv`: `(date, store_nbr)` — 0 duplicate.
- `holidays_events.csv` có 350 dòng và 312 ngày khác nhau; nhiều event trên cùng ngày là tình huống hợp lệ, do đó không đặt `date` làm khóa duy nhất.
- Kiểm tra duplicate toàn dòng trên các bảng nhỏ trong `01` cũng không ghi nhận duplicate.

### 5.4 Referential integrity

| Quan hệ | Kết quả |
|---|---|
| `train.store_nbr → stores.store_nbr` | 0 khóa không tìm thấy; `OK` |
| `train.item_nbr → items.item_nbr` | 0 khóa không tìm thấy; `OK` |
| Số store phân biệt trong train | 54 |
| Số item phân biệt trong train | 4,036 |
| Số item trong `items.csv` | 4,100 |

Việc số item xuất hiện trong train ít hơn tổng số item trong metadata không tự thân là lỗi; kiểm tra tham chiếu không phát hiện item lạ.

## 6. Phát hiện về thời gian và EDA

### 6.1 Phạm vi thời gian

| Dataset | Ngày bắt đầu | Ngày kết thúc |
|---|---|---|
| `train.csv` | 2013-01-01 | 2017-08-15 |
| `test.csv` | 2017-08-16 | 2017-08-31 |
| `transactions.csv` | 2013-01-01 | 2017-08-15 |
| `oil.csv` | 2013-01-01 | 2017-08-31 |
| `holidays_events.csv` | 2012-03-02 | 2017-12-26 |

Train có 1,684 ngày quan sát duy nhất trong khoảng lịch gồm 1,688 ngày. Bốn ngày không xuất hiện là:

- 2013-12-25
- 2014-12-25
- 2015-12-25
- 2016-12-25

Bốn ngày đều là ngày 25/12. Đây là bằng chứng để điều tra/nghiên cứu thêm, **không phải căn cứ để tự thêm ngày với `unit_sales = 0`**. Train kết thúc ngày 2017-08-15 và test bắt đầu ngay ngày 2017-08-16.

### 6.2 Các phân tích EDA đã chạy

`01` đã tạo các phân tích/đồ thị sau:

- Histogram và boxplot của `unit_sales` trên mẫu.
- Tổng `unit_sales` theo ngày.
- Trung bình doanh số theo thứ trong tuần.
- Tổng doanh số theo tháng.
- Top 20 store và top 20 item theo tổng `unit_sales`.
- So sánh `unit_sales` theo trạng thái promotion.
- Giá dầu theo thời gian và tương quan với tổng doanh số theo ngày.
- Kiểm tra cấu trúc `holidays_events.csv` và phân bố các cột event.

Một số kết quả số cụ thể được lưu trong output:

- Store có tổng `unit_sales` cao nhất trong bảng xếp hạng là `store_nbr = 44`, khoảng **62.09 triệu**.
- Item có tổng `unit_sales` cao nhất trong bảng xếp hạng là `item_nbr = 1503844`, khoảng **6.264 triệu**.
- Với các dòng có trạng thái promotion được ghi rõ: `onpromotion = True` có 7,810,622 dòng và mean `unit_sales` khoảng **13.4667**; `False` có 96,028,767 dòng và mean khoảng **8.0881**. 21,657,651 dòng có promotion missing không thuộc hai nhóm này. Đây là so sánh mô tả, **không chứng minh promotion gây ra doanh số cao hơn**.
- Tương quan giữa tổng `unit_sales` và `dcoilwtico` khi ghép theo ngày và chỉ dùng các ngày có đủ cặp giá trị là khoảng **-0.7050**. Đây là tương quan aggregate có thể chịu ảnh hưởng của trend, seasonality, promotion và các yếu tố khác; không diễn giải thành quan hệ nhân quả.

Các cell đồ thị đã được chạy, nhưng một số Markdown interpretation/conclusion trong `01` vẫn là mẫu hướng dẫn để người viết điền. Vì vậy, chưa nên coi mọi kết luận định tính về trend/seasonality đã được ghi thành kết quả chính thức nếu chưa được bổ sung vào notebook.

## 7. Những gì `02_data_cleaning` đã thực sự làm

### 7.1 Validation gate

`02` báo:

`VALIDATION GATE PASSED: có thể chuyển sang chuẩn hóa và ghi dữ liệu.`

Gate xác nhận schema bắt buộc, các trường ngày/khóa quan trọng, referential integrity, token promotion và điều kiện kiểm tra khóa nghiệp vụ đạt yêu cầu để tiếp tục chuẩn hóa. Điều này **không có nghĩa** missing đã được giải quyết hoặc mọi giá trị đã hoàn hảo về mặt ngữ nghĩa.

### 7.2 Chính sách xử lý được thực hiện

| Hạng mục | Hành động trong `02` |
|---|---|
| `date` | Parse/chuẩn hóa sang kiểu ngày |
| Khóa số | Chuẩn hóa/ép kiểu thích hợp cho store/item/ID |
| `onpromotion` | Chuẩn hóa thành Boolean nullable khi ghi Parquet; giữ missing |
| Text của bảng nhỏ | Trim whitespace đầu/cuối |
| `unit_sales < 0` | Giữ nguyên và thống kê |
| Missing `onpromotion` | Giữ nguyên; không tự đổi thành `False` |
| Missing `dcoilwtico` | Giữ nguyên; không forward-fill/interpolate trong bước này |
| Ngày thiếu khỏi train | Không tự thêm bản ghi |
| Duplicate | Phát hiện nhưng không tự xóa |

### 7.3 Đầu ra đã ghi và xác nhận số dòng

| File đầu vào | File đầu ra | Số dòng đầu vào | Số dòng đầu ra | Trạng thái |
|---|---|---:|---:|---|
| `train.csv` | `data/processed/train_cleaned.parquet` | 125,497,040 | 125,497,040 | `OK` |
| `test.csv` | `data/processed/test_cleaned.parquet` | 3,370,464 | 3,370,464 | `OK` |
| `stores.csv` | `data/processed/stores_cleaned.parquet` | 54 | 54 | `OK` |
| `items.csv` | `data/processed/items_cleaned.parquet` | 4,100 | 4,100 | `OK` |
| `oil.csv` | `data/processed/oil_cleaned.parquet` | 1,218 | 1,218 | `OK` |
| `holidays_events.csv` | `data/processed/holidays_events_cleaned.parquet` | 350 | 350 | `OK` |
| `transactions.csv` | `data/processed/transactions_cleaned.parquet` | 83,488 | 83,488 | `OK` |

Tất cả file trên có `row_count_match = True`, không thiếu cột yêu cầu trong output validation. Đầu ra train giữ lại 21,657,651 missing `onpromotion` và 7,795 giá trị `unit_sales` âm như chủ đích. Notebook cũng ghi `reports/02_cleaning_report.json` để lưu báo cáo tổng hợp.

**Cách hiểu chính xác:** bước 02 hiện là **validation + normalization + xuất dữ liệu có kiểm chứng**, chưa phải imputation hay sửa các vấn đề nghiệp vụ. Đây là lựa chọn thận trọng phù hợp vì các quyết định điền/xóa dữ liệu chưa có đủ căn cứ.

## 8. Vấn đề còn mở và lưu ý trước `03_feature_engineering`

1. **Thống nhất tên target:** cập nhật `TARGET_COLUMN` và ghi chú còn sót trong `00`/`01` từ `sales` thành `unit_sales`.
2. **Chính sách `onpromotion`:** 17.26% dữ liệu train bị missing, còn test không missing. Giữ nullable trong dữ liệu cleaned là đúng ở giai đoạn hiện tại; feature engineering cần quyết định cách biểu diễn missing, dựa trên ngữ nghĩa và quy trình tạo feature, thay vì mặc định điền `False`.
3. **Chính sách giá dầu:** `dcoilwtico` có 43 missing. Nếu tạo feature được điền theo thời gian, mọi phép fill phải tôn trọng thời điểm dự báo và không sử dụng thông tin tương lai ngoài quy trình được phép.
4. **Transactions không bao phủ test:** `transactions.csv` kết thúc ngày 2017-08-15, trong khi test là 2017-08-16 đến 2017-08-31. Không thể dùng transaction thực tế của chính ngày test như feature có sẵn tại thời điểm dự báo. Nếu dùng lịch sử transactions, chỉ dùng dữ liệu quá khứ hoặc feature lag hợp lệ.
5. **Holiday/event có thể nhiều dòng mỗi ngày:** khi join vào dữ liệu bán hàng, không join thẳng theo `date` nếu điều đó tạo nhân bản bản ghi. Cần thiết kế phép biến đổi/aggregate theo ngày và locale phù hợp.
6. **Giữ đúng grain của dữ liệu bán hàng:** kiểm soát khóa `(date, store_nbr, item_nbr)` sau mọi phép join và feature transformation; row count hoặc unique key không được tăng ngoài chủ đích.
7. **Giữ `id` để tạo submission:** không bỏ mất quan hệ giữa `id` của test và từng dòng dự báo; `sample_submission.csv` chưa được chuẩn hóa ở `02`.
8. **Temporal validation:** không dùng random split cho bài toán forecasting. Chọn khoảng validation theo thời gian và tạo lag/rolling sao cho mỗi dự báo chỉ dùng thông tin thực sự có sẵn tại thời điểm đó. Test có horizon 16 ngày, đây là tham khảo hữu ích khi thiết kế validation, nhưng split cuối cùng cần được ghi rõ trong notebook/modeling plan.
9. **Quantile của target:** mẫu hiện tại lấy từ các chunk đầu; tính lại bằng mẫu trải đều trên toàn khoảng thời gian trước khi dùng quantile làm căn cứ quyết định mô hình hoặc biến đổi target.
10. **Ghi diễn giải EDA:** hoàn thiện các Markdown interpretation còn để trống trong `01` nếu muốn xem đó là báo cáo EDA cuối cùng; riêng các số liệu kiểm tra/cleaning đã có output có thể sử dụng ngay.

## 9. Đề xuất hợp đồng đầu vào cho `03_feature_engineering`

- Đọc các file `*_cleaned.parquet` trong `data/processed/`, không đọc lại raw để tạo feature chính.
- Dùng `unit_sales` làm target lịch sử; giữ `id` của test để khôi phục đúng thứ tự submission.
- Giữ grain bán hàng `(date, store_nbr, item_nbr)` và xác minh uniqueness sau khi ghép bảng.
- Tạo calendar features từ `date`; tạo lag/rolling chỉ từ target quá khứ, với nhóm theo store/item phù hợp và quy định rõ cách khởi tạo ở biên thời gian.
- Với `stores` và `items`, kiểm soát cardinality khi join theo khóa duy nhất. Với holidays, tránh join many-to-many ngoài ý muốn.
- Xác định rõ availability-time của `transactions` và oil; không dùng dữ liệu chưa sẵn có tại thời điểm dự báo.
- Lưu quy tắc imputation/encoding trong `configs/features.yaml` khi đã ra quyết định, và giữ cho các bước build feature có thể tái lập.
- Trước khi xuất bộ feature, lưu báo cáo: số dòng, số cột, missing theo feature quan trọng, unique business key, khoảng thời gian, dtype và kiểm tra leakage cơ bản.

## 10. Kết luận

Ba notebook đã đưa dự án từ **xác định phạm vi và giới hạn tài nguyên** (`00`), qua **data validation và khám phá dữ liệu** (`01`), đến **chuẩn hóa dữ liệu và xác nhận đầu ra** (`02`). Những vấn đề chính đã được lượng hóa; các quyết định cleaning hiện tại có chủ đích bảo toàn dữ liệu thay vì tự động xóa hoặc điền giá trị khi chưa đủ bằng chứng.

Dự án đã có thể chuyển sang `03_feature_engineering`. Các quyết định về `onpromotion` missing, dầu thiếu, holiday joins, transactions availability và temporal leakage cần được ghi rõ khi triển khai feature. Báo cáo này phản ánh kết quả notebook đã có, không thay thế cho việc chạy lại kiểm tra sau khi code hoặc dữ liệu thay đổi.
