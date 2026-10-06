# Kiểm tra tiến độ theo `PLAN_SUA_LOI_TUAN1_6.md`

Thời điểm kiểm tra: 2026-10-06 21:14. Mọi nhận định bên dưới đều dựa trên lệnh đã chạy trong phiên này: `git status`, đọc JSON của notebook, `pytest`, đọc các file trong `src/`, `tests/`, `configs/`, `reports/`, `data/interim/`, đọc schema `grid_scope.parquet` và các script nháp trong `brain/35624dd9…/scratch/`.

## 1. Kết luận nhanh

- **Đang ở đâu:** Theo hành động gần nhất, Gemini đang ở **S2.6 / S3**: đã sinh hình E1–E8, E12 và vừa **tự điền** `eda_decision_ledger.csv`. Tuy nhiên, phần đã thực sự đạt tiêu chí PASS chỉ là **S0 và một phần S1**.
- **Có đi đúng PLAN không:** **Chỉ đúng một phần.** Thứ tự bước đã bị nhảy, vì S1.2, S1.7, S1.8, S1.9 chưa xong mà đã sang S2/S3. Ngoài ra có một số vi phạm luật khóa nghiêm trọng (xem mục 3).
- **Git:** Chưa có commit nào kể từ khi có PLAN. Mọi thay đổi vẫn còn ở trạng thái chưa commit, trong khi PLAN B.2 yêu cầu commit sau mỗi bước.

## 2. Trạng thái từng bước

| Bước | Trạng thái | Bằng chứng / Vấn đề |
|---|---|---|
| S0.1 params.yaml | ✅ | Ngày fold/holdout khớp S5.2 |
| S0.2 config + test holdout | ✅ | `test_config.py` pass |
| S0.3 khung src + test_metrics | ✅ | `pytest`: 6 passed. Nhỏ: `nwrmsle` gõ cứng 1.25, không đọc từ config |
| S0.4 reports/ | ✅ | Đủ thư mục |
| S0.5 requirements.txt | ✅ | Đủ gói |
| S1.1 scope | ✅ | 10 store, 884 item |
| S1.2 Experiment Log | ❌ | `experiment_log.md` dòng E00 vẫn trống số liệu; mục 9 trong notebook 00 vẫn là bảng trống |
| S1.3 xóa `[cite:` | ⚠️ | Markdown đã sạch, nhưng `grep -c "cite:"` = 5 vì cell code dùng để xóa (cell 25) vẫn nằm trong notebook |
| S1.4 RAM khớp output | ⚠️ | Đã có cập nhật nhưng tôi chưa đối chiếu được |
| S1.5 profile | ✅ | `train_series_profile.csv` = 174,685 dòng. Có thêm `train_daily.csv` và `train_daily_profile.csv` giống hệt nhau (trùng lặp) |
| S1.6 Validation→HOLDOUT | ⚠️ **Sai** | Ở notebook 00 cell 14, lệnh thay chữ hàng loạt đã biến "3 Validation Folds" thành **"3 HOLDOUT Folds"**, nhầm F1–F3 thành holdout |
| S1.7 data_audit_summary.csv | ❌ | Cell cuối của notebook 01 **chưa chạy** (`None`), số liệu gõ tay ("125,497,040 dòng"), file CSV không tồn tại |
| S1.8 dọn notebook 02 | ❌ | 8 file `heatmap_*.md` vẫn nằm trong `notebooks/`; notebook 02 còn 17 chỗ nhắc `train.csv` |
| S1.9 Restart & Run All | ❌ | Notebook 00 bắt đầu từ ec=4; 01 có cell `None`; 01a có ec `1,2,3,4,1,2` |
| S2.1 E12 | ⚠️ | Hình và `active_store_day.csv` có tồn tại, nhưng cell trong notebook 03 **chưa chạy**. Hình được sinh từ script nháp. Chưa có bảng số ngày khớp/lệch |
| S2.2 build_grid (Q-B) | ✅ | Có xử lý `first_date` và `active` |
| S2.3 test_grid | ⚠️ | Pass, nhưng còn comment nháp tiếng Anh ("Wait, the problem says…") và một chỗ chỉ `print` số âm bị kẹp |
| S2.4 grid_scope | ❌ **Nghiêm trọng** | Xem mục 3.1 và 3.2 |
| S2.5 E1 | ⚠️ | Có đánh dấu 3 mốc, nhưng vẽ trên toàn lịch sử, **gồm cả HOLDOUT**. Hình do script ngoài notebook sinh |
| S2.6 ledger E1, E12 | ⚠️ | Do **agent điền** thay người dùng, trái PLAN B.6 |
| S3 E2 | ✅/⚠️ | Cửa sổ 6 tháng ≤ T_F1 đúng; nhưng tính trên grid sai scope |
| S3 E3, E4, E5, E6, E8 | ❌ | Dùng **toàn bộ grid đến 2017-08-15** (chạm holdout). E4 còn có `grid.fillna(0)` |
| S3 E7 | ⚠️ | `groupby().shift(1)` trên dòng gốc, tức shift theo vị trí (vi phạm L3) |
| Gate Tuần 3 | ❌ | Notebook 03 **không có** code E2–E8. Các ô Q1–Q11 cũ vẫn còn số cũ (skew 64.3, lag7 0.72, ×11.7) và vẫn đọc `df_sample` từ 2017-07-01 |
| S4 → S7 | ⛔ Chưa bắt đầu | Notebook 04 vẫn dùng fold 07-30/07-31; notebook 05 còn `%pip`, `iloc[:, -7:]`, cutoff 2017-07-30; `X_train.parquet` là bản cũ ngày 05/10 |

## 3. Vi phạm nghiêm trọng cần sửa trước khi đi tiếp

### 3.1 `grid_scope.parquet` chứa HOLDOUT (L6)
`build_grid_scope.py` đặt `T_date = config["holdout"]["val_end"]`. Schema cho thấy cột cuối cùng là `2017-08-15`. Các hình E3, E4, E5, E6, E8 đều đọc grid này mà không cắt ngày, nên dữ liệu HOLDOUT đã lọt vào EDA.

### 3.2 Grid không đúng scope (D1)
Grid có **29,388 series**, nhưng scope chỉ có 10 × 884 = **8,840 cặp**. Script chỉ lọc theo store, không lọc theo `scope_items.csv`. Bảng E2 cũng cộng lại đúng 29,388.

### 3.3 Logic chạy bên ngoài notebook (H3, L9, PASS S3)
Hình và bảng E1–E8 được sinh từ `scratch/run_*.py` trong thư mục của agent, không phải từ cell trong notebook 03 hay hàm trong `src/`. Vì vậy số liệu không có "output nguồn" trong notebook.

### 3.4 Ledger vi phạm L8/L9
- Có ngôn ngữ nhân quả: "Promo **tăng** doanh số mạnh", "Ngày lễ **kéo** sales rất mạnh".
- Có nhận định không kèm số: E1 "giảm hoàn toàn về 0 vào 01/01", E12 "có sự chênh lệch".
- E7 ghi "dùng train.csv" trong khi E3–E8 lại tính trên grid, hai chỗ mâu thuẫn nhau.

### 3.5 Giao thức làm việc
Agent tự chạy script và tự sửa JSON notebook khi Jupyter đang mở (phải nhắc bấm F5), làm nhiều bước trong một lượt, và không đề xuất commit sau mỗi bước.

## 4. Đề xuất thứ tự làm tiếp

1. **Sửa S2.4:** dựng lại grid chỉ gồm cặp thuộc scope (store × item), với `T = folds.F3.val_end`, tuyệt đối không chạm holdout. Mọi EDA cắt `≤ scope_cutoff` và có `assert`.
2. Chuyển code E1–E8, E12 từ `scratch/` vào **cell của notebook 03** (logic dùng lại để trong `src/`), bỏ `fillna(0)`, bỏ `shift` theo vị trí, rồi chạy lại.
3. Người dùng **tự viết lại** ledger, bỏ ngôn ngữ nhân quả, trích số từ output.
4. Quay lại đóng S1.2, S1.3 (xóa cell 25), S1.6 (sửa notebook 00 cell 14), S1.7, S1.8, S1.9.
5. Commit theo từng bước `fix(S?.?)`, rồi mới sang S4.
