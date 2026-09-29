# Data Audit Summary (F1-F13)

Các số liệu dưới đây lấy từ output đã chạy trong `00_scope_and_preregistration.ipynb`, `checking_stage.ipynb` và `notebook.ipynb`.

| # | Phát hiện | Hệ quả | Xử lý ở |
|---|---|---|---|
| F1 | `train.csv` 125,497,040 dòng, 4,765.94 MB. Máy: 8 CPU logic, RAM 7.50 GB tổng nhưng chỉ **2.10 GB khả dụng**, không GPU. Sample 100k dòng đầu tốn 10.2 MB → đọc ngây thơ ≈ 12–13 GB | Không thể load toàn bộ; cần scope và cache Parquet | **D1** |
| F2 | Train: 54 store, 4,036 item, 174,685 series. Test: 54 × 3,901 = 210,654 cặp × 16 ngày = 3,370,464 dòng | Test là lưới Cartesian đầy đủ | D4 |
| F3 | Train **thưa**: 0 dòng `unit_sales == 0`. Ở 16 ngày cuối train, chỉ 46.8%–52.6% số cặp trong lưới test có dòng dữ liệu mỗi ngày | Dòng thiếu ≠ dòng bằng 0 một cách hiển nhiên; phải có quy tắc rõ | **D4** |
| F4 | 4 ngày thiếu toàn bộ: 25/12 các năm 2013–2016 (Navidad). Ngày 2013-01-01 chỉ có 1 store | `shift`/`rolling` phải theo **ngày lịch**, không theo vị trí dòng | D4, Tuần 5 |
| F5 | Trong validation, ngày 2017-08-01 chỉ có 39/54 store, ngày 2017-08-11 chỉ có 42/54 store | Có store-day không có dòng nào (đóng cửa hoặc thiếu dữ liệu?) | D4, Tuần 3 |
| F6 | 7,795 dòng âm (0.0062%); min −15,372, max 89,440; percentile 99.5% ≈ 100 (ước lượng từ 1% mẫu) | Quy tắc xử lý âm và đuôi nặng | **D5** |
| F7 | `onpromotion` NaN 21,657,651 dòng (17.26%); True 7,810,622 (6.22%). Các ngày đầu 2013 NaN hoàn toàn, các ngày 2017 không còn NaN. Test có `onpromotion` đầy đủ | Không `fillna(False)` bừa; xác định mốc bắt đầu có dữ liệu | **D6** |
| F8 | 60 item chỉ có trong test; 44,441 cặp store-item chưa từng xuất hiện trong train (21.10% số cặp test) | Cold-start là phần đáng kể; baseline lịch sử cho 0 | **D8** |
| F9 | `transactions.csv` kết thúc 2017-08-15, không có cho horizon | Chỉ được dùng dạng lag/rolling ≤ cutoff, hoặc bỏ | **D7** |
| F10 | `oil.csv`: 43 NaN (3.53%), chỉ có ngày thường (không có T7/CN); file phủ tới 2017-08-31 | Giá dầu sau cutoff là tương lai, dù file có sẵn | **D7** |
| F11 | `holidays_events.csv`: 350 dòng, 38 ngày có nhiều sự kiện cùng ngày (khác locale/mô tả) | Join theo locale (National/Regional/Local), không join chỉ theo `date` | **D7** |
| F12 | Notebook checking cố định **một** cửa sổ 2017-07-31 → 2017-08-15; hướng dẫn yêu cầu 3 fold + holdout | Cần định nghĩa lại vai trò cửa sổ này | **D2** |
| F13 | Validation có Thứ Hai/Thứ Ba ×3; test có Thứ Tư/Thứ Năm ×3 (cửa sổ 16 ngày luôn lệch 2 thứ) | Báo cáo thêm metric theo thứ trong tuần | D2 |
