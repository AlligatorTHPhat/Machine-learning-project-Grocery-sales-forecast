# Hướng dẫn thực hiện dự án: Dự báo doanh số bán hàng tạp hóa

> File này viết cho người mới học Machine Learning (ML). Mỗi giai đoạn có: **mục tiêu** (làm để làm gì), **checklist** (việc cần làm), **đầu ra** (file nào sẽ có sau khi xong) và **lưu ý** (lỗi hay gặp).
>
> Ký hiệu: `- [x]` = đã làm xong (dựa trên 3 notebook bạn đã gửi) · `- [ ]` = còn phải làm.
>
> Nên đọc kèm: `FINDINGS_REPORT.md` (báo cáo những gì đã phát hiện được và đề xuất hướng đi).
>
> Gợi ý đặt file: copy cả hai file vào thư mục `docs/` của dự án.

---

## Mục lục

1. [Bài toán bằng lời đơn giản](#1-bài-toán-bằng-lời-đơn-giản)
2. [Từ điển thuật ngữ](#2-từ-điển-thuật-ngữ)
3. [Thư mục nào dùng để làm gì](#3-thư-mục-nào-dùng-để-làm-gì)
4. [7 quy tắc vàng](#4-7-quy-tắc-vàng)
5. [Các giai đoạn và checklist](#5-các-giai-đoạn-và-checklist)
6. [Cách tạo file nộp kết quả (submission)](#6-cách-tạo-file-nộp-kết-quả-submission)
7. [Checklist cuối trước khi nộp](#7-checklist-cuối-trước-khi-nộp)
8. [Khi máy báo hết RAM](#8-khi-máy-báo-hết-ram)

---

## 1. Bài toán bằng lời đơn giản

- **Dữ liệu:** lịch sử bán hàng của chuỗi siêu thị (có vẻ là bộ dữ liệu *Corporación Favorita* ở Ecuador). Mỗi dòng trong `train` cho biết: ngày nào, cửa hàng nào, mặt hàng nào, bán được bao nhiêu (`unit_sales`), có khuyến mãi không (`onpromotion`).
- **Việc cần làm:** đoán `unit_sales` cho **16 ngày tiếp theo** (từ 2017-08-16 đến 2017-08-31) cho từng cặp *(cửa hàng, mặt hàng)* nằm trong file `test`.
- **Đây là bài toán "dự báo theo thời gian"** (time series forecasting). Điểm khác biệt quan trọng so với bài ML thông thường: **không được trộn ngẫu nhiên dữ liệu** để chia train/validation, vì như vậy là "nhìn trộm tương lai".

Hình dung đơn giản: bạn đứng ở ngày 15/08/2017, chỉ biết những gì đã xảy ra đến hết ngày đó, và phải đoán 16 ngày sau. Mọi bước trong dự án đều phải giữ đúng tình huống này.

Luồng làm việc (lấy từ README của bạn):

```
Hiểu & kiểm tra dữ liệu → EDA → Làm sạch → Tạo đặc trưng (feature)
→ Chia dữ liệu theo thời gian → Baseline → Huấn luyện mô hình
→ Đánh giá → Mô hình cuối + Báo cáo
```

---

## 2. Từ điển thuật ngữ

| Thuật ngữ | Giải thích dễ hiểu |
|---|---|
| **Target (mục tiêu)** | Con số cần dự đoán. Ở đây là `unit_sales`. |
| **Feature (đặc trưng)** | Thông tin đầu vào giúp mô hình đoán, ví dụ: thứ trong tuần, có khuyến mãi không, doanh số trung bình 7 ngày gần nhất. |
| **EDA** | Xem và vẽ biểu đồ dữ liệu để hiểu nó trước khi làm gì khác. |
| **Data cleaning** | Làm sạch: sửa kiểu dữ liệu, xử lý giá trị lỗi/thiếu. |
| **Missing value** | Ô dữ liệu bị trống. |
| **Leakage (rò rỉ dữ liệu)** | Mô hình vô tình "thấy" thông tin của tương lai khi học. Kết quả thử nghiệm đẹp nhưng khi dùng thật thì tệ. **Đây là lỗi nguy hiểm nhất của bài này.** |
| **Lag** | Giá trị của quá khứ. Ví dụ "lag 16 ngày" = doanh số của 16 ngày trước. |
| **Rolling mean** | Trung bình trượt. Ví dụ "trung bình 7 ngày gần nhất". |
| **Validation (kiểm định)** | Một phần dữ liệu để chấm thử mô hình trước khi nộp. Với dữ liệu thời gian, phần này phải **nằm sau** phần dùng để học. |
| **Baseline** | Cách đoán rất đơn giản (ví dụ: "ngày mai bán bằng trung bình tuần qua"). Mô hình ML phải thắng được baseline thì mới có giá trị. |
| **Overfitting** | Mô hình học "thuộc lòng" dữ liệu cũ nên đoán dữ liệu mới kém. |
| **Chunk** | Đọc file lớn từng phần nhỏ (ví dụ 100.000 dòng/lần) để không tràn RAM. |
| **Parquet** | Định dạng file dữ liệu gọn và nhanh hơn CSV, giữ nguyên kiểu dữ liệu. |
| **log1p / expm1** | `log1p(x) = log(1+x)` làm các số lớn "nhỏ lại" cho mô hình dễ học; `expm1` là phép ngược để đổi về số bán thật. |
| **RMSLE / NWRMSLE** | Thước đo sai số dựa trên log. NWRMSLE là bản có trọng số (hàng dễ hỏng nặng hơn). Xem giai đoạn 6. |
| **Cold start** | Mặt hàng/cửa hàng mới, chưa có lịch sử, nên khó đoán. |

---

## 3. Thư mục nào dùng để làm gì

| Thư mục / file | Dùng để | Quy tắc |
|---|---|---|
| `data/raw/` | Dữ liệu gốc | **Chỉ đọc, không sửa, không ghi đè.** |
| `data/processed/` | Dữ liệu đã làm sạch (`*_cleaned.parquet`) | Đầu ra của notebook 02. |
| `data/interim/` | File trung gian (ví dụ: bộ dữ liệu "gần đây" đã cắt nhỏ) | Có thể xóa và tạo lại. |
| `data/features/` | Bảng đặc trưng đã tạo | Đầu ra của notebook 03. |
| `notebooks/` | Nơi thử nghiệm, vẽ biểu đồ, ghi chú | Đánh số 00 → 07 theo thứ tự. |
| `src/` | Code Python gọn gàng để chạy lại | Chuyển dần code ổn định từ notebook sang đây. |
| `configs/` | Cấu hình (đường dẫn, ngày cắt, tham số mô hình) | Tránh "gõ cứng" số trong code. |
| `models/baseline`, `experiments`, `final` | Mô hình đã lưu | `final` chỉ chứa mô hình được chọn cuối cùng. |
| `reports/` | Bảng, hình, báo cáo | `reports/table`, `reports/figures`, `final_report.md`. |
| `tests/` | Kiểm tra tự động | Viết sau khi code trong `src/` ổn định. |
| `docs/` | Tài liệu | Đặt file hướng dẫn và báo cáo này vào đây. |

> Lưu ý: mình chưa xem nội dung các file trong `src/`, `configs/`, `tests/` và các notebook 03–07. Mình giả định chúng đang trống hoặc mới có khung. Nếu đã có code, hãy giữ lại và đối chiếu với checklist.

---

## 4. 7 quy tắc vàng

1. **Không sửa `data/raw/`.** Mọi thay đổi ghi sang thư mục khác.
2. **Không chia ngẫu nhiên.** Luôn chia theo thời gian: học ở quá khứ, kiểm tra ở tương lai.
3. **Mỗi feature phải trả lời được câu hỏi:** "Vào ngày 15/08/2017, mình có biết thông tin này cho ngày cần dự đoán không?" Nếu không → bỏ (đó là leakage).
4. **Không nạp cả `train` vào RAM.** Máy bạn có ~7.5 GB RAM nhưng thường chỉ còn ~2–2.7 GB trống, trong khi `train` đầy đủ cần ước tính ~6.8 GB.
5. **Ghi lại mọi quyết định và lý do** (một dòng trong notebook hoặc báo cáo là đủ).
6. **Làm nhỏ trước, làm lớn sau.** Chạy thử toàn bộ quy trình trên một mẫu nhỏ (vài triệu dòng) cho đến khi không còn lỗi, rồi mới tăng dữ liệu.
7. **Luôn so sánh với baseline.** Con số sai số đứng một mình không có ý nghĩa.

---

## 5. Các giai đoạn và checklist

### Giai đoạn 0 — Chuẩn bị môi trường

**Mục tiêu:** có môi trường chạy được, ai cũng dựng lại được.

- [x] Tạo môi trường: `conda create -n grocery_env python=3.11 -y` rồi `conda activate grocery_env`
- [x] Cấu trúc thư mục dự án đã dựng
- [x] Kiểm tra `requirements.txt` có đủ các thư viện sau (nếu thiếu thì thêm): `pandas`, `numpy`, `pyarrow`, `psutil`, `matplotlib`, `scikit-learn`, `lightgbm`, `jupyter`
- [ ] Cân nhắc cài thêm `duckdb` (đọc/lọc/gộp file Parquet lớn mà không tốn nhiều RAM): `pip install duckdb`
- [ ] Chạy lại từ đầu trong môi trường sạch để chắc chắn `requirements.txt` đủ
- [x] Kiểm tra `.gitignore` đã loại `data/`, `models/`, `*.parquet` (file rất nặng, không nên đưa lên Git)
- [ ] Cập nhật `README.md` gốc: thêm cách chạy từng notebook theo thứ tự

---

### Giai đoạn 1 — Phạm vi & môi trường (`00_scope_and_enviroment.ipynb`) ✔ đã xong

**Mục tiêu:** biết rõ bài toán và giới hạn máy.

- [x] Xác định bài toán: dự báo `unit_sales`
- [x] Ghi nhận máy: 8 nhân CPU, 7.5 GB RAM (trống ~2.4–2.7 GB), không có GPU, ~100 GB ổ đĩa trống
- [x] Thống kê 8 file dữ liệu (`train.csv` ~4.65 GB, 125.497.040 dòng)
- [x] Kết luận: **bắt buộc đọc `train` theo chunk**
- [x] Kết luận: chia dữ liệu theo thời gian (không chia ngẫu nhiên)
- [x] (Nhỏ) Sửa vài ghi chú cho khớp thực tế: file `transactions.csv` chỉ ~1.5 MB (không phải > 1 GB); mục "có cần tối ưu dtype không → không" nên đổi thành "có" vì notebook 02 đã ép kiểu `int32`
- [x] (Nhỏ) Tên file đang viết `enviroment` (thiếu chữ `n`), còn notebook 02 gọi là `environment`. Thống nhất một cách viết

**Hệ quả cho các bước sau:** máy không có GPU nên ưu tiên mô hình chạy tốt trên CPU (ví dụ LightGBM), tránh mạng nơ-ron sâu.

---

### Giai đoạn 2 — Kiểm tra dữ liệu & EDA (`01_eda.ipynb`) ✔ phần lớn đã xong

**Mục tiêu:** hiểu dữ liệu trông như thế nào, có vấn đề gì.

- [x] Kiểm tra schema (tên cột, kiểu dữ liệu) của 7 bảng
- [x] Đếm giá trị thiếu: `onpromotion` thiếu 17,26% (21.657.651 dòng) trong `train`; `oil.dcoilwtico` thiếu 43 dòng (3,53%)
- [x] Kiểm tra khóa tham chiếu: mọi `store_nbr`, `item_nbr` trong `train` đều có trong `stores`, `items`
- [x] Kiểm tra thời gian: `train` từ 2013-01-01 đến 2017-08-15, `test` từ 2017-08-16 đến 2017-08-31
- [x] Kiểm tra target: không có giá trị 0; có 7.795 dòng âm (0,006%)
- [x] Vẽ biểu đồ: phân phối target, doanh số theo ngày/thứ/tháng, theo cửa hàng, mặt hàng, khuyến mãi, giá dầu, ngày lễ
- [x] **Điền phần kết luận còn trống** ở mục 9.1–9.4 của notebook (đang ghi "Cập nhật sau"). Có thể dùng nội dung trong `FINDINGS_REPORT.md`
- [x] **Vẽ lại biểu đồ phân phối target và tính lại phân vị bằng mẫu ngẫu nhiên lấy từ toàn bộ file.** Hiện tại mẫu chỉ lấy từ 20 chunk đầu (~2 triệu dòng đầu, tức là những ngày đầu năm 2013), nên bị lệch
- [x] Kiểm tra `onpromotion` bị thiếu **vào những ngày nào** (đoán: tập trung ở giai đoạn đầu, cần xác nhận bằng đếm theo tháng)
- [x] Kiểm tra trong `test` có bao nhiêu mặt hàng và cặp (cửa hàng, mặt hàng) **chưa từng xuất hiện** trong `train` (cold start)
- [x] Kiểm tra mức độ "thưa" của dữ liệu: `train` chỉ chứa các dòng có bán, vậy bao nhiêu % tổ hợp (ngày × cửa hàng × mặt hàng) có dòng?
- [x] Vẽ doanh số trung bình theo `family` và theo `perishable` (hàng dễ hỏng) vì sau này dùng làm trọng số
- [ ] Kiểm tra bằng biểu đồ: doanh số có tăng vào ngày 15 và cuối tháng không (ngày trả lương)?

**Đầu ra:** notebook có kết luận đầy đủ; hình lưu vào `reports/figures/`.

---

### Giai đoạn 3 — Làm sạch dữ liệu (`02_data_cleaning.ipynb`) ✔ đã xong

**Mục tiêu:** chuẩn hóa dữ liệu, giữ nguyên số dòng, không "đoán" bừa.

- [x] Chuẩn hóa `date` thành kiểu ngày; `store_nbr`, `item_nbr` thành `int32`; `onpromotion` thành Boolean cho phép thiếu
- [x] Chứng minh **không có dòng trùng** theo khóa `(date, store_nbr, item_nbr)` (dữ liệu đã sắp xếp nên kiểm tra chunk đủ để kết luận)
- [x] Đã dựng "cổng kiểm tra" (validation gate): nếu có lỗi cấu trúc thì dừng, không ghi file
- [x] Ghi 7 file `*_cleaned.parquet` vào `data/processed/`; số dòng đầu vào = đầu ra ở mọi file
- [x] Lưu báo cáo `reports/02_cleaning_report.json` và các bảng trong `reports/table/`
- [x] Quyết định có chủ đích: **chưa** xóa dòng âm, **chưa** điền giá trị thiếu, **chưa** thêm ngày thiếu
- [x] Mở `reports/02_cleaning_report.json` và `reports/table/02_output_validation.csv` kiểm tra lần cuối: tất cả `status = OK`
- [x] Đối chiếu với file `docs/data_understanding_cleaning_report.md` đã có (mình chưa xem), tránh hai báo cáo nói khác nhau

**Quy tắc từ đây trở đi:** dữ liệu trong `data/processed/` coi như "chuẩn". Mọi xử lý mới (điền thiếu, cắt âm về 0, ...) làm ở bước tạo feature, **không** ghi đè file cleaned.

---

### Giai đoạn 4 — Tạo "bộ dữ liệu làm việc" (đầu notebook `03`)

**Mục tiêu:** có một bản dữ liệu đủ nhỏ để vừa RAM nhưng vẫn đủ để học.

**Vì sao cần:** `train_cleaned.parquet` nặng ~1.1 GB trên đĩa nhưng khi nạp vào RAM sẽ phình ra nhiều lần. Ngoài ra, dữ liệu 2013–2014 ít liên quan đến việc dự báo tháng 8/2017 và là nơi `onpromotion` thiếu nhiều (cần xác nhận).

- [ ] Đếm số dòng theo năm/tháng để biết "dữ liệu gần đây" nặng cỡ nào
- [ ] Chọn mốc bắt đầu, ví dụ **2016-08-01** (khoảng 1 năm trước ngày cắt). Có thể bắt đầu nhỏ hơn (từ 2017-01-01) rồi mở rộng nếu RAM cho phép
- [ ] Đọc có lọc (không cần đọc cả file), ví dụ:

```python
import pandas as pd

df = pd.read_parquet(
    "../data/processed/train_cleaned.parquet",
    columns=["date", "store_nbr", "item_nbr", "unit_sales", "onpromotion"],
    filters=[("date", ">=", pd.Timestamp("2016-08-01"))],
)
df["unit_sales"] = df["unit_sales"].astype("float32")   # tiết kiệm RAM
print(df.shape, df.memory_usage(deep=True).sum() / 1024**3, "GB")
```

- [ ] Kiểm tra RAM sau khi nạp (`psutil.virtual_memory().available`). Nếu còn dưới ~1 GB thì thu hẹp khoảng thời gian
- [ ] Lưu ra `data/interim/train_recent.parquet`
- [ ] Ghi vào notebook: mốc ngày đã chọn và lý do

**Lưu ý:** nếu vẫn thiếu RAM, dùng DuckDB để lọc và gộp trực tiếp trên file Parquet, rồi chỉ lấy kết quả đã gộp.

---

### Giai đoạn 5 — Tạo đặc trưng (`03_feature_engineering.ipynb`)

**Mục tiêu:** biến dữ liệu thô thành thông tin mà mô hình học được, **không rò rỉ tương lai**.

**Nguyên tắc cốt lõi:** bạn phải dự đoán cho các ngày 16/08 → 31/08. Với ngày 31/08, chỉ biết dữ liệu đến 15/08, tức là cách 16 ngày. Vậy **mọi feature về doanh số quá khứ phải dùng dữ liệu cách ít nhất 16 ngày so với ngày cần đoán**. Cách này đơn giản và an toàn: dùng cùng một công thức cho cả lúc học lẫn lúc dự đoán.

#### 5.1 Quyết định cần chốt trước

- [ ] **Ngày không có dòng nào** trong `train` có nghĩa là gì? Vì `train` không có dòng nào bằng 0, nhiều khả năng "không có dòng" = "không bán được" (nhưng cũng có thể là cửa hàng chưa mở hoặc mặt hàng chưa bán). Khi tính trung bình trượt, nên **thử cả hai cách** (coi là 0 / bỏ qua) và so sánh kết quả
- [ ] **Doanh số âm (trả hàng):** trong tập target dùng để học, cắt về 0 (`clip(lower=0)`), vì doanh số bán không thể âm và cuộc thi gốc cũng xử lý như vậy. Làm ở bước tạo target, không sửa file cleaned
- [ ] **`onpromotion` thiếu:** nếu thiếu tập trung ở giai đoạn đầu và bạn đã cắt bỏ giai đoạn đó ở giai đoạn 4 thì vấn đề gần như biến mất. Nếu còn thiếu, tạo cột đánh dấu "thiếu" riêng thay vì tự điền `False`
- [ ] **Biến đổi target:** học trên `log1p(unit_sales)` thay vì số gốc (vì phân phối rất lệch: trung vị ~4, nhưng tối đa 89.440)

#### 5.2 Nhóm đặc trưng nên tạo (làm từng nhóm một, kiểm tra từng nhóm)

**Nhóm A — Lịch (an toàn, luôn biết trước):**
- [ ] Thứ trong tuần, ngày trong tháng, tháng, tuần trong năm
- [ ] Cuối tuần hay không
- [ ] Gần ngày 15 / cuối tháng (kiểm tra bằng biểu đồ trước, ở giai đoạn 2)

**Nhóm B — Khuyến mãi (có sẵn cả trong `test`):**
- [ ] `onpromotion` của chính dòng đó
- [ ] (Tùy chọn) số mặt hàng đang khuyến mãi tại cửa hàng đó trong ngày

**Nhóm C — Thông tin cửa hàng và mặt hàng (không đổi theo thời gian):**
- [ ] Từ `stores`: `city`, `state`, `type`, `cluster`
- [ ] Từ `items`: `family`, `class`, `perishable`
- [ ] Biến chữ thành "category" cho LightGBM (không cần one-hot)

**Nhóm D — Lịch sử bán hàng (quan trọng nhất, và dễ gây leakage nhất):**
- [ ] Trung bình doanh số 7, 14, 28, 56 ngày, **tính đến thời điểm cách 16 ngày** trở về trước, theo từng cặp (cửa hàng, mặt hàng)
- [ ] Trung bình theo thứ trong tuần trong 4–8 tuần gần nhất (thứ Bảy bán khác thứ Năm rất nhiều, xem báo cáo)
- [ ] Số ngày kể từ lần bán gần nhất; tỷ lệ ngày có bán trong 28 ngày
- [ ] Doanh số cùng kỳ tuần trước (lag 21, lag 28 ngày)

> Mẹo thực hiện khi RAM ít: với khoảng 1 năm dữ liệu, tạo một ma trận NumPy kiểu `float32` có hàng = cặp (cửa hàng, mặt hàng), cột = ngày (điền 0 nếu không có dòng). Ước tính ~210.000 cặp × 365 ngày × 4 byte ≈ 0,3 GB, vừa RAM. Rồi tính trung bình trượt bằng phép cộng dồn (cumsum). Nhanh hơn nhiều so với `groupby().rolling()` trên bảng dài.

**Nhóm E — Ngày lễ (làm sau khi nhóm A–D chạy ổn):**
- [ ] Xử lý `holidays_events`: bỏ dòng `transferred = True` (ngày lễ bị dời, thực tế không nghỉ); giữ dòng `type = Transfer` (ngày thực tế được nghỉ); `Bridge` coi là ngày nghỉ; `Work Day` là ngày đi làm bù
- [ ] Ghép đúng phạm vi: `National` áp dụng mọi cửa hàng; `Regional` khớp `state`; `Local` khớp `city`
- [ ] Một ngày có thể có nhiều dòng, nên gộp lại (ví dụ: có ngày lễ quốc gia = 1/0)

**Nhóm F — Giá dầu (ưu tiên thấp, thử sau cùng):**
- [ ] Điền giá dầu thiếu bằng giá của ngày trước đó (forward-fill), ngày đầu tiên dùng giá ngày kế tiếp
- [ ] Thử mô hình có và không có feature này; chỉ giữ nếu thực sự cải thiện
- [ ] **Không dùng** hệ số tương quan −0,705 trong notebook 01 làm bằng chứng (xem giải thích trong báo cáo)

**Nhóm G — Số giao dịch `transactions` (khuyến nghị bỏ ở vòng đầu):**
- [ ] File này chỉ đến ngày 2017-08-15, nên **không có** cho các ngày cần dự đoán. Nếu dùng thì chỉ được dùng dạng quá khứ (lag ≥ 16 ngày). Giá trị mang lại có hạn, nên để sau

#### 5.3 Kiểm tra chống rò rỉ (bắt buộc)

- [ ] Với mỗi feature nhóm D, tự hỏi: "Ngày 15/08/2017, mình có tính được số này cho ngày 31/08/2017 không?"
- [ ] Viết một bài kiểm tra nhỏ: đổi giá trị doanh số của các ngày trong khoảng cần dự đoán thành số bất kỳ, rồi tạo lại feature. Nếu feature thay đổi nghĩa là đang rò rỉ
- [ ] Không dùng `shift` âm, không dùng `rolling` có `center=True`

**Đầu ra:** `data/features/train_features.parquet`, `val_features.parquet`, `test_features.parquet`; danh sách feature lưu trong `configs/features.yaml`.

---

### Giai đoạn 6 — Chia theo thời gian + Baseline (`04_baseline.ipynb`)

**Mục tiêu:** có thước đo công bằng và một mốc so sánh.

#### 6.1 Chia dữ liệu

`test` dài 16 ngày, nên validation cũng nên dài 16 ngày để giống thật:

```python
VAL_END   = pd.Timestamp("2017-08-15")
VAL_START = VAL_END - pd.Timedelta(days=15)      # 2017-07-31 → 16 ngày

train_part = df[df["date"] <  VAL_START]
val_part   = df[df["date"] >= VAL_START]
```

- [ ] Chia như trên: học đến 2017-07-30, kiểm tra 2017-07-31 → 2017-08-15
- [ ] Kiểm tra: ngày lớn nhất của `train_part` < ngày nhỏ nhất của `val_part`
- [ ] (Tùy chọn, khi đã ổn) thêm 1–2 lần kiểm tra lùi về trước (mỗi lần 16 ngày) để xem kết quả có ổn định không
- [ ] Lưu cách chia vào `configs/base.yaml` để dùng lại

**Chú ý:** feature của các dòng trong validation chỉ được dùng dữ liệu cách ≥ 16 ngày trước đó, **kể cả dữ liệu thuộc validation**. Ví dụ ngày 10/08 chỉ được dùng doanh số đến 25/07.

#### 6.2 Thước đo

Nếu bạn làm theo cuộc thi gốc (Kaggle *Favorita Grocery Sales Forecasting*), thước đo là **NWRMSLE**: sai số trên thang log, hàng dễ hỏng (`perishable = 1`) có trọng số 1,25, còn lại 1,0; doanh số âm được tính là 0. **Hãy xác nhận thước đo với đề bài/giảng viên của bạn.**

```python
import numpy as np

def nwrmsle(y_true, y_pred, perishable):
    y_true = np.clip(y_true, 0, None)
    y_pred = np.clip(y_pred, 0, None)
    w = np.where(perishable == 1, 1.25, 1.0)
    err = (np.log1p(y_pred) - np.log1p(y_true)) ** 2
    return np.sqrt((w * err).sum() / w.sum())
```

- [ ] Đặt hàm này vào `src/models/evaluate.py` để dùng chung
- [ ] Kiểm tra hàm bằng ví dụ nhỏ tính tay

#### 6.3 Baseline

- [ ] **Baseline 1:** dự đoán = trung bình doanh số 14 ngày gần nhất (theo cặp cửa hàng-mặt hàng)
- [ ] **Baseline 2:** dự đoán = trung bình cùng thứ trong tuần, 4 tuần gần nhất
- [ ] Tính điểm cả hai trên validation, ghi vào `reports/table/04_baseline_scores.csv`
- [ ] Quyết định mốc cần vượt: baseline tốt nhất
- [ ] Xử lý cặp chưa có lịch sử: dùng trung bình của cùng `family` + cửa hàng (hoặc 0)

---

### Giai đoạn 7 — Huấn luyện mô hình (`05_model_training.ipynb`)

**Mục tiêu:** tìm mô hình tốt hơn baseline.

- [ ] **Mô hình 1 (đơn giản, để so sánh):** hồi quy tuyến tính có regularization (ví dụ Ridge) trên một mẫu nhỏ
- [ ] **Mô hình 2 (chính):** `LightGBM` — chạy tốt trên CPU, xử lý được dữ liệu dạng bảng và biến phân loại
- [ ] (Tùy chọn) Mô hình 3: XGBoost hoặc RandomForest trên mẫu nhỏ để so sánh
- [ ] Học trên `log1p(max(unit_sales, 0))`; đổi về bằng `expm1` khi dự đoán, rồi `clip(lower=0)`
- [ ] **Chạy thử trên mẫu 1–2 triệu dòng** trước. Chỉ tăng dữ liệu khi quy trình đã không còn lỗi
- [ ] Dùng validation để dừng sớm (`early_stopping`), tránh overfitting
- [ ] Chỉnh tham số từng bước nhỏ (số cây, `learning_rate`, `num_leaves`, `min_child_samples`), ghi lại mỗi lần thử
- [ ] Đặt `n_jobs` khoảng 6 (máy có 8 nhân; chừa một ít cho hệ thống)
- [ ] Lưu mỗi lần thử vào `models/experiments/` kèm file ghi tham số và điểm
- [ ] Lưu tham số vào `configs/model.yaml`

**Lưu ý RAM:** nếu bị đầy RAM, giảm số dòng học (lấy mẫu các dòng của 3–6 tháng gần nhất), giảm số feature, hoặc dùng kiểu `float32`/`category`.

---

### Giai đoạn 8 — Đánh giá (`06_model_evaluation.ipynb`)

**Mục tiêu:** hiểu mô hình tốt ở đâu, kém ở đâu, chứ không chỉ nhìn một con số.

- [ ] Bảng so sánh: baseline vs Ridge vs LightGBM (cùng validation, cùng thước đo)
- [ ] Điểm theo **từng ngày dự đoán** (ngày thứ 1 … thứ 16): có xấu dần về cuối không?
- [ ] Điểm theo `family`, theo `perishable`, theo cửa hàng: nhóm nào kém nhất?
- [ ] Biểu đồ so sánh dự đoán và thực tế cho vài cặp tiêu biểu (bán nhiều, bán ít, có khuyến mãi)
- [ ] Biểu đồ mức quan trọng của feature (feature importance); feature nào lạ thì kiểm tra lại leakage
- [ ] Phân tích các dòng sai lệch lớn nhất
- [ ] Ghi 3–5 nhận xét bằng lời thường

---

### Giai đoạn 9 — Phân tích cuối + Báo cáo (`07_final_analysis.ipynb`, `reports/final_report.md`)

- [ ] Chọn mô hình cuối và nêu lý do (điểm, độ ổn định, thời gian chạy)
- [ ] Huấn luyện lại mô hình cuối trên **toàn bộ dữ liệu đến 2017-08-15** (gồm cả validation) với tham số đã chọn
- [ ] Tạo feature cho `test` đúng cách (xem mục 6), dự đoán, tạo file nộp
- [ ] Lưu mô hình vào `models/final/`
- [ ] Viết `reports/final_report.md`: bài toán, dữ liệu, phương pháp, kết quả, hạn chế, hướng cải thiện
- [ ] Mỗi kết luận trong báo cáo phải có số liệu hoặc biểu đồ đi kèm

---

### Giai đoạn 10 — Gọn code và kiểm thử (làm xen kẽ, hoàn thiện cuối cùng)

- [ ] `src/data/load.py`: hàm đọc Parquet có lọc ngày
- [ ] `src/data/preprocess.py`: các bước chuẩn hóa từ notebook 02
- [ ] `src/features/build.py`: các hàm tạo feature (cùng một hàm cho train, validation và test)
- [ ] `src/models/train.py`, `evaluate.py`, `predict.py`
- [ ] `tests/test_features.py`: kiểm tra không rò rỉ (xem 5.3)
- [ ] `tests/test_data.py`: kiểm tra số dòng, kiểu dữ liệu, không trùng khóa
- [ ] `tests/test_pipeline.py`: chạy cả quy trình trên mẫu rất nhỏ
- [ ] Chạy `pytest` và sửa lỗi

---

## 6. Cách tạo file nộp kết quả (submission)

File `sample_submission.csv` có 2 cột: `id` và `unit_sales`, đúng 3.370.464 dòng (bằng số dòng của `test`).

- [ ] Dự đoán trên thang log → `expm1` → `clip(lower=0)`
- [ ] Ghép theo `id` của `test`, không để mất hoặc thừa dòng nào
- [ ] Đủ 3.370.464 dòng, không có giá trị trống hoặc âm
- [ ] Thứ tự và tên cột giống `sample_submission.csv`
- [ ] Lưu vào `reports/` hoặc `data/` (ví dụ `submission.csv`), đừng ghi đè `data/raw/sample_submission.csv`

---

## 7. Checklist cuối trước khi nộp

- [ ] Chạy lại toàn bộ notebook từ trên xuống dưới không lỗi (`Restart & Run All`)
- [ ] `data/raw/` không bị thay đổi
- [ ] Validation nằm hoàn toàn **sau** dữ liệu học
- [ ] Không có feature nào dùng thông tin sau ngày 2017-08-15
- [ ] Mô hình thắng baseline trên validation
- [ ] File submission đúng định dạng
- [ ] Báo cáo cuối có số liệu và biểu đồ, viết bằng ngôn ngữ dễ hiểu
- [ ] `README.md` hướng dẫn được người khác chạy lại
- [ ] `requirements.txt` đầy đủ

---

## 8. Khi máy báo hết RAM

Thử lần lượt từ trên xuống:

1. Đóng trình duyệt và các ứng dụng nặng (máy bạn thường chỉ còn ~2 GB trống)
2. Khởi động lại kernel của notebook để giải phóng bộ nhớ
3. Chỉ đọc các cột cần thiết (`columns=[...]`)
4. Thu hẹp khoảng thời gian dữ liệu
5. Ép kiểu nhỏ hơn: `float32`, `int32`, `category`
6. Xóa DataFrame không cần nữa: `del df` rồi `import gc; gc.collect()`
7. Lấy mẫu một phần các dòng để học
8. Dùng DuckDB để lọc/gộp trực tiếp trên Parquet
9. Giảm `CHUNK_SIZE` (đang là 100.000)
