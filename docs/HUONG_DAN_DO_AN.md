# Hướng dẫn làm đồ án: Dự báo doanh số siêu thị Favorita

> Đọc từ trên xuống, không cần đọc một lần hết. Mỗi tuần mở lại **Phần 5** để biết tuần đó làm gì.
> Phần kỹ thuật chi tiết nằm trong `AGENTS.md` (dành cho AI agent). Anh không cần đọc file đó, nhưng agent sẽ đọc để giúp anh đúng hướng.
> Cập nhật: 01/10/2026.

---

## Phần 1. Đồ án này là gì? (đọc 2 phút)

Favorita là chuỗi siêu thị thật ở Ecuador. Mình có dữ liệu **mỗi ngày, mỗi cửa hàng bán được bao nhiêu của mỗi món hàng**, từ 2013 đến 15/08/2017.

**Nhiệm vụ:** đứng ở một ngày nào đó, đoán xem **16 ngày tới** mỗi cửa hàng sẽ bán được bao nhiêu mỗi món.

Ví dụ: hôm nay là 12/06/2017. Hỏi: cửa hàng số 1 sẽ bán bao nhiêu hộp sữa X vào ngày 13/06, 14/06, …, 28/06?

**Thầy chấm gì nhiều nhất?** Không phải mô hình phức tạp, mà là:

> "Em **nhìn thấy gì trong dữ liệu**, và **vì thế** em quyết định làm feature/mô hình như thế nào?"

Nên mọi thứ anh làm phải trả lời được câu "**vì sao?**" bằng một biểu đồ hoặc một con số.

---

## Phần 2. Anh đã làm được gì

Anh đã làm tốt phần nền móng, khoảng **1/4 đồ án**:

| Notebook | Đã làm | Đánh giá |
|---|---|---|
| `00_scope_and_preregistration` | Viết ra câu hỏi nghiên cứu, cách chia dữ liệu, baseline, các luật chống gian lận dữ liệu | Gần xong, còn vài chỗ phải sửa |
| `01_data_checking` | Kiểm tra dữ liệu có lỗi không: trùng, thiếu, sai ngày… 11/11 bài kiểm tra đều đạt | Làm rất cẩn thận 👍 |
| `02_eda_raw_tables` | Vẽ biểu đồ cho từng bảng; 6 biểu đồ quan hệ | Có hình nhưng **chưa viết nhận xét**, vài hình bị hiểu nhầm |

**Còn lại 3/4:** phân tích dữ liệu để ra quyết định → làm baseline → làm feature → huấn luyện mô hình → so sánh → viết báo cáo → bảo vệ.

---

## Phần 3. 12 khái niệm phải hiểu (rất quan trọng khi bảo vệ)

Đọc chậm từng cái. Nếu chưa hiểu, hỏi agent: *"Giải thích lại khái niệm X bằng ví dụ khác cho tôi."*

**1. Chuỗi thời gian (time series).** Dữ liệu sắp theo ngày. Hôm sau thường giống hôm trước. Vì vậy **không được xáo trộn** dữ liệu như các bài ML thông thường.

**2. Ngày cắt (cutoff, ký hiệu T).** Ngày "hôm nay" khi dự báo. Chỉ được dùng thông tin **đến ngày T**. Giống đi thi: chỉ được dùng kiến thức trước giờ thi, không được xem đáp án.

**3. Horizon (h).** Dự báo cách ngày cắt bao nhiêu ngày. Đồ án này: h = 1, 2, …, 16. Đoán ngày mai (h=1) dễ hơn đoán 16 ngày sau (h=16).

**4. Leakage (rò rỉ dữ liệu).** Vô tình cho mô hình nhìn thấy tương lai lúc học. Mô hình sẽ có điểm rất đẹp nhưng **dùng thật thì hỏng**. Đây là lỗi thầy bắt nhiều nhất.
> Ví dụ lỗi: dự báo ngày 25/06 (h=13) mà dùng "doanh số 7 ngày trước" = ngày 18/06. Nhưng ngày 18/06 là **sau** ngày cắt 12/06, lúc đó mình chưa biết! → Leakage.

**5. Baseline.** Cách đoán "ngây thơ", không cần ML. Ví dụ: "tuần sau bán giống tuần này". Mô hình ML **phải thắng baseline** thì mới có ý nghĩa. Mình có 4 baseline:
- B0: đoán 0 hết.
- B1: đoán bằng số bán ngày cuối cùng đã biết.
- B2: đoán bằng số bán **cùng thứ** tuần gần nhất đã biết (thứ Hai đoán theo thứ Hai).
- B3: trung bình 4 tuần gần nhất, cùng thứ.

**6. Fold, validation, holdout.** Cách chia dữ liệu theo thời gian để chấm điểm:
- **3 fold**: 3 lần "giả vờ" đứng ở 3 ngày cắt khác nhau (12/06, 28/06, 14/07/2017), mỗi lần đoán 16 ngày tiếp theo rồi so với thật. Dùng để chọn mô hình, chỉnh tham số.
- **Holdout** (31/07 → 15/08/2017): bài thi cuối. **Chỉ được chạy một lần duy nhất** khi mọi thứ đã chốt. Nhìn holdout rồi sửa mô hình = gian lận.
- File `test.csv` của Kaggle **không có đáp án** nên không dùng để chấm được.

**7. Feature (đặc trưng).** Thông tin mình đưa cho mô hình để nó đoán. Ví dụ: trung bình bán 7 ngày qua, hôm đó có khuyến mãi không, hôm đó là thứ mấy, có phải ngày lễ không.

**8. Target và log1p.** Target = thứ cần đoán (`unit_sales`). Số bán lệch rất mạnh: đa số món bán 0–10, vài món bán hàng nghìn. Lấy `log1p(y) = log(1 + y)` để "nén" số lớn lại, mô hình học dễ hơn.

**9. RMSLE / NWRMSLE.** Cách chấm điểm của cuộc thi: đo sai số **sau khi lấy log**. Nghĩa là đoán 2 khi thật là 1 bị phạt gần bằng đoán 200 khi thật là 100 (sai gấp đôi như nhau). Hàng tươi sống (perishable) bị phạt nặng hơn 1.25 lần. **Điểm càng thấp càng tốt.**

**10. Dữ liệu thưa và "số 0 ẩn".** File train **không có dòng nào bán = 0**. Ngày nào không bán thì **dòng đó biến mất**. Nên phải tự thêm số 0 vào, nhưng cẩn thận:
- Cửa hàng đóng cửa (25/12, 01/01) → **không** điền 0, vì không bán do đóng cửa chứ không phải do không ai mua.
- Món hàng chưa từng xuất hiện ở cửa hàng đó → không điền 0 cho giai đoạn trước khi nó xuất hiện.

**11. Cold-start.** Cặp cửa hàng–món hàng **chưa có lịch sử** (món mới). 21% cặp trong test là như vậy. Phải có cách xử lý riêng và báo cáo riêng.

**12. Ablation.** Bỏ từng nhóm feature ra xem điểm tệ đi bao nhiêu → biết nhóm nào quan trọng. Giống bỏ từng nguyên liệu khỏi món ăn để biết nguyên liệu nào làm nên vị.

---

## Phần 4. Máy yếu mà dữ liệu lớn: làm sao?

File `train.csv` nặng **4.7 GB, 125 triệu dòng**. Máy anh chỉ còn trống **2–3 GB RAM**. Đọc hết vào pandas thì sẽ cần 12–13 GB → **treo máy**.

**5 quy tắc sống còn:**

1. **Không bao giờ** chạy `pd.read_csv("train.csv")` không có tham số. Luôn đọc theo phần nhỏ (`chunksize`) hoặc chỉ đọc vài cột.
2. **Chuyển sang Parquet một lần**, sau đó chỉ đọc Parquet. Parquet nén tốt, đọc nhanh hơn nhiều, đọc được từng cột.
3. **Không dùng toàn bộ 54 cửa hàng × 4,000 món.** Chọn một phần đại diện (gọi là **scope**): khoảng 10 cửa hàng × 1,000 món, lấy dữ liệu từ 06/2016. Chọn "phân tầng", tức là có đủ loại cửa hàng, đủ ngành hàng, đủ món bán chạy lẫn bán chậm. **Không** chỉ lấy món bán chạy nhất.
4. Chạy thử trên **1–2 cửa hàng** trước, xem tốn bao nhiêu RAM, rồi mới mở rộng.
5. Kiểm tra RAM bằng:
   ```python
   import psutil
   print(psutil.virtual_memory().available / 1024**3, "GB còn trống")
   ```

> Trong notebook 00, phần "Scope Decision" đang ghi **"FULL DATA"**. Phải sửa lại theo quy tắc 3, nếu không thầy sẽ hỏi "máy 3 GB RAM làm full data kiểu gì?".

---

## Phần 5. Lộ trình từ giờ đến ngày bảo vệ

Giả định bảo vệ khoảng cuối tháng 12/2026. Nếu biết ngày chính xác thì báo agent chỉnh lại lịch. Mỗi tuần **chỉ sang tuần sau khi đã tick hết ô của tuần này**.

### Tuần 1 (05–11/10): Sửa những gì đã làm

- [x] Notebook 00: sửa "FULL DATA" thành scope (Phần 4); điền các dòng còn "...".
- [x] Notebook 01: sửa **lỗi đếm cửa hàng ở cell 16** (nhờ agent, xem Phần 7).
- [x] Tạo notebook `01a_convert_to_parquet`: chuyển CSV sang Parquet. Hiện notebook 01 đang đọc file Parquet mà không có notebook nào tạo ra nó.
- [x] Mở từng notebook → **Restart & Run All** → chạy hết không lỗi.
- [x] Cài git và lưu lại (`git init`, `git commit`). Hỏi agent cách làm nếu chưa biết.

### Tuần 2–4 (12/10–01/11): Phân tích dữ liệu để ra quyết định ⭐ quan trọng nhất

Làm notebook mới `03_eda_for_decisions`. Có 12 câu hỏi (Phần 6). Mỗi câu: vẽ biểu đồ → viết nhận xét → ghi quyết định.

- [x] Tuần 2: làm dữ liệu dạng "lưới ngày" có điền số 0 đúng cách; trả lời câu 1 và 12.
- [x] Tuần 3: câu 2 đến 8.
- [x] Tuần 4: câu 9 đến 11; điền **bảng quyết định** (Phần 6); lưu git với tên `prereg-v1` ("đã chốt kế hoạch").

### Tuần 5 (02–08/11): Cách chấm điểm và baseline

- [ ] Viết hàm tính điểm NWRMSLE và kiểm tra nó đúng.
- [ ] Chạy B0, B1, B2, B3 trên 3 fold. Ghi điểm vào bảng.

### Tuần 6 (09–15/11): Làm feature

- [ ] Viết code tạo feature. Kiểm tra **không feature nào nhìn thấy tương lai**.

### Tuần 7–8 (16–29/11): Huấn luyện mô hình

- [ ] Tuần 7: chạy 3 mô hình Ridge, HistGradientBoosting, LightGBM. So với baseline.
- [ ] Tuần 8: chỉnh tham số LightGBM (có giới hạn số lần thử, ví dụ 30 lần).

### Tuần 9 (30/11–06/12): Phân tích kết quả

- [ ] Ablation: bỏ từng nhóm feature, xem điểm thay đổi.
- [ ] Xem mô hình đoán sai nhiều ở đâu: món bán chậm? món mới? ngày có khuyến mãi? dự báo xa (h lớn)?

### Tuần 10 (07–13/12): Bài thi cuối

- [ ] Chạy holdout **một lần**. Ghi kết quả, không sửa gì nữa.

### Tuần 11–12 (14–27/12): Viết báo cáo, làm slide, tập nói

- [ ] Báo cáo theo dàn ý ở Phần 8.
- [ ] Tập trả lời các câu hỏi ở Phần 9, nói to thành lời ít nhất 2 lần.

---

## Phần 6. Làm EDA sao cho thầy thích

### Công thức 3 dòng: dùng sau MỌI biểu đồ

Ngay dưới mỗi biểu đồ, viết một ô markdown như sau:

> **Quan sát:** (thấy gì, **có con số**)
> **Giải thích:** (vì sao có thể như vậy — chỉ là giả thuyết)
> **Quyết định:** (vì thế mình sẽ làm gì với feature/mô hình)

**Ví dụ viết tốt** (số trong ví dụ chỉ để minh họa, phải thay bằng số thật từ biểu đồ của anh):

> **Quan sát:** Doanh số có đỉnh lặp lại mỗi 7 ngày; cuối tuần cao hơn thứ Ba khoảng 30%.
> **Giải thích:** Người dân đi chợ nhiều vào cuối tuần.
> **Quyết định:** Dùng feature "thứ trong tuần" và "trung bình cùng thứ trong 4 tuần gần nhất"; baseline B2/B3 cũng dựa trên điều này.

**Ví dụ viết chưa tốt:**

> Biểu đồ cho thấy doanh số thay đổi theo ngày.

(Không có số, không có quyết định → thầy sẽ hỏi "rồi sao?")

**Bí quyết:** viết sẵn **trước khi vẽ**: "Nếu thấy A thì em làm X, nếu không thấy thì em làm Y." Như vậy chứng minh được mình không vẽ xong rồi mới bịa lý do.

### 12 câu hỏi cần trả lời

| # | Câu hỏi đơn giản | Nếu thấy… thì quyết định… |
|---|---|---|
| 1 | Doanh số tăng vì mỗi món bán nhiều hơn, hay vì có thêm cửa hàng/món? | Ảnh hưởng tới việc chọn lấy dữ liệu từ năm nào |
| 2 | Bao nhiêu món bán đều mỗi ngày, bao nhiêu món lâu lâu mới bán? | Chọn cách chấm điểm, cách huấn luyện; chia nhóm để phân tích lỗi |
| 3 | Số bán phân bố thế nào? Có lệch nhiều không? | Dùng `log1p` |
| 4 | Đoán ngày mai thì nên nhìn hôm qua hay tuần trước? Đoán 2 tuần sau thì sao? | Chọn feature lịch sử; cách tổ chức mô hình theo h |
| 5 | Thứ trong tuần ảnh hưởng thế nào? Ngành hàng khác nhau có khác không? | Feature thứ; chọn mô hình cây |
| 6 | Ngày 15 và cuối tháng (ngày lương ở Ecuador) có bán nhiều hơn không? | Có thì thêm feature "gần ngày lương"; không thì ghi "đã kiểm tra, không thấy" |
| 7 | Khuyến mãi đi kèm bán nhiều hơn bao nhiêu? Ngành nào nhiều nhất? | Feature khuyến mãi |
| 8 | Ngày lễ có bán khác không (sau khi đã trừ yếu tố mùa và thứ)? | Feature ngày lễ |
| 9 | Giá dầu và số giao dịch có giúp đoán không? | Giữ hay bỏ |
| 10 | Vài món/cửa hàng có chiếm phần lớn doanh số không? | Cách chọn scope; feature ngành hàng, loại cửa hàng |
| 11 | Mỗi tháng có bao nhiêu món mới xuất hiện? | Cách xử lý món mới (cold-start) |
| 12 | Ngày cửa hàng không có dòng nào là đóng cửa hay thiếu dữ liệu? | Quy tắc điền số 0 |

Agent biết cách tính chi tiết cho từng câu. Chỉ cần nói: *"Làm câu EDA số 6 (E6) với tôi, từng bước một."*

### Bảng quyết định (rất có giá trị khi bảo vệ)

Cuối notebook 03, tổng hợp lại thành một bảng:

| # | Thấy gì (có số) | Quyết định |
|---|---|---|
| 1 | … | … |
| 2 | … | … |

Bảng này đưa vào báo cáo và slide. Khi thầy hỏi "vì sao có feature này?", chỉ vào bảng.

### Biểu đồ cũ trong notebook 02 cần sửa

| Biểu đồ | Vấn đề | Sửa |
|---|---|---|
| Tổng sales theo thời gian | Tăng có thể chỉ vì mở thêm cửa hàng | Vẽ thêm "doanh số trung bình mỗi dòng" |
| "Hiệu ứng khuyến mãi" | Chỉ đếm số dòng có khuyến mãi, chưa đo hiệu ứng | Làm lại theo câu 7 |
| Boxplot theo thứ | Trộn năm 2013 (bán ít) với 2017 (bán nhiều) | So sánh trong từng tuần (câu 5) |
| Ngày lễ vs ngày thường | Lễ dồn vào tháng 12 và các năm sau, vốn đã bán nhiều → **hiểu nhầm** là lễ làm tăng doanh số | So với các ngày thường cùng thứ gần đó (câu 8) |
| Histogram của `item_nbr`, `store_nbr` | Đây là **mã số**, vẽ phân bố không có ý nghĩa | Bỏ khỏi báo cáo |

---

## Phần 7. Những lỗi cần biết

1. ~~**Lỗi đếm cửa hàng (notebook 01, cell 16):** in ra ngày 01/08/2017 chỉ có 39 cửa hàng, ngày 11/08 chỉ có 42. **Sai.** Thực tế đủ 54. Lỗi do khi gộp các phần dữ liệu, code lấy số lớn nhất thay vì đếm gộp. Đừng đưa con số 39/42 vào báo cáo.~~ (Đã fix)
2. **Thiếu file trung gian:** notebook 01 và 02 đọc `train.parquet` và `train_series_profile.csv` nhưng không có notebook nào tạo ra hai file này. Người khác (và thầy) sẽ không chạy lại được.
3. **Thứ tự chạy cell bị nhảy** trong notebook 01: chạy lại từ đầu bằng Restart & Run All.
4. **Khuyến mãi bị "đếm lệch":** ngày có khuyến mãi mà không bán được thì dòng đó biến mất khỏi dữ liệu. Nên khi tính, hiệu ứng khuyến mãi trông **mạnh hơn thực tế**. Phải ghi điều này trong phần hạn chế của báo cáo.
5. Notebook Kaggle (`demand-forecasting-...ipynb`) **chỉ để tham khảo ý tưởng**. Không chép số liệu hay kết luận từ đó. Nếu dùng ý tưởng thì ghi nguồn "tham khảo lời giải trên Kaggle".

---

## Phần 8. Dàn ý báo cáo

1. **Giới thiệu:** bài toán, vì sao quan trọng, 3 câu hỏi nghiên cứu.
2. **Dữ liệu:** mô tả 7 bảng, kết quả kiểm tra chất lượng (notebook 01).
3. **Phân tích dữ liệu → quyết định:** 12 câu hỏi, mỗi câu có hình + 3 dòng nhận xét; cuối chương là bảng quyết định. **Chương dài nhất (~25–30%).**
4. **Phương pháp:** chia dữ liệu theo thời gian, baseline, feature, mô hình, cách chống leakage.
5. **Kết quả:** ML so với baseline; ablation; mô hình sai ở đâu; kết quả holdout.
6. **Thảo luận và hạn chế:** chỉ dùng một phần dữ liệu; số 0 là tự điền; khuyến mãi bị đếm lệch; không kết luận nhân quả.
7. **Kết luận.**

**3 câu hỏi nghiên cứu** (nhớ thuộc):
- RQ1: Mô hình ML có đoán tốt hơn cách "tuần sau giống tuần này" không?
- RQ2: Nhóm thông tin nào (lịch sử, khuyến mãi, ngày lễ, thông tin cửa hàng/món…) giúp nhiều nhất?
- RQ3: Mô hình tốt/kém ở nhóm nào (món bán đều, món lâu lâu mới bán, món mới, ngày khuyến mãi, dự báo xa)?

---

## Phần 9. Câu hỏi thầy có thể hỏi (kèm gợi ý)

Gợi ý chỉ là ý chính. **Anh phải tự nói lại bằng lời mình**, có dẫn biểu đồ/số của chính mình.

| Câu hỏi | Gợi ý ý trả lời |
|---|---|
| Sao không chia train/test ngẫu nhiên? | Dữ liệu theo thời gian; chia ngẫu nhiên thì mô hình học từ tương lai → điểm đẹp giả |
| Dữ liệu không có số 0, em điền 0 theo giả định gì? | Không có dòng = không bán, **chỉ khi** cửa hàng mở cửa và món đã từng xuất hiện |
| Ngày 25/12 và 01/01 xử lý sao? | Siêu thị đóng cửa → bỏ, không điền 0 |
| Sao dùng RMSLE, không dùng MAPE? | Nhiều ngày bán 0, MAPE chia cho 0; RMSLE phù hợp số lệch và là metric của cuộc thi |
| Dự báo ngày thứ 10 có dùng "lag 7" được không? | Không, ngày đó sau ngày cắt → leakage; phải dùng cùng thứ của 14 ngày trước |
| Khuyến mãi có làm tăng doanh số không? | Có **liên hệ**, nhưng không kết luận nhân quả; ước lượng còn bị lệch (Phần 7, ý 4) |
| Feature nào quan trọng nhất, có bằng chứng trước khi chạy mô hình không? | Chỉ vào bảng quyết định và kết quả ablation |
| Sao bỏ (hoặc giữ) giá dầu? | Chỉ vào biểu đồ câu 9 |
| Mô hình kém nhất ở đâu, vì sao? | Kết quả tuần 9 |
| Holdout khác validation chỗ nào? Em xem holdout mấy lần? | Holdout chỉ chạy 1 lần sau khi chốt; validation dùng để chọn mô hình |
| Sao không dùng hết dữ liệu? | RAM 3 GB; chọn mẫu phân tầng để vẫn đại diện; nêu hạn chế |
| Sao không dùng deep learning / ARIMA? | Không có GPU; 174 nghìn chuỗi, nhiều chuỗi thưa và món mới → mô hình cây trên feature phù hợp hơn |

---

## Phần 10. Làm việc với AI agent cho hiệu quả

Agent đã được dặn (trong `AGENTS.md`) là **kèm anh từng bước**, không làm thay hết.

**Nên nói như vầy:**
- "Mình đang ở Tuần 2. Hôm nay làm gì?"
- "Viết cho tôi **một** cell để làm việc X, giải thích từng dòng."
- "Output này nghĩa là gì? Tôi nên viết nhận xét thế nào?"
- "Tôi viết nhận xét thế này, sửa giúp tôi cho rõ hơn: …"
- "Hỏi tôi 3 câu để kiểm tra tôi đã hiểu phần leakage chưa."
- "Cell này có tốn RAM không? Chạy được trên máy 3 GB không?"

**Không nên:**
- "Làm hết notebook 03 cho tôi." → Anh sẽ không giải thích được khi thầy hỏi.
- Chạy code mà không đọc output.
- Copy kết luận của agent mà không tự kiểm tra lại với biểu đồ.

**Quy tắc vàng:** sau mỗi buổi làm, tự viết 3 câu: *Hôm nay mình làm gì? Thấy gì? Quyết định gì?* Ghi vào `reports/experiment_log.md`. Đến lúc viết báo cáo, gần như đã có sẵn nội dung.

---

## Phần 11. Khi bị kẹt

| Tình huống | Làm gì |
|---|---|
| Máy treo / hết RAM | Restart kernel; giảm số cửa hàng/món; đọc ít cột hơn; hỏi agent cách đọc theo chunk |
| Không hiểu một khái niệm | Hỏi agent giải thích bằng ví dụ đời thường; đọc lại Phần 3 |
| Không biết viết nhận xét biểu đồ | Dùng công thức 3 dòng ở Phần 6; nhìn xem cái gì cao/thấp/lặp lại/bất thường |
| Mô hình thua baseline | **Không sao**, đó cũng là kết quả. Ghi lại, phân tích vì sao. Kiểm tra có lỗi điền 0 hay leakage không |
| Điểm mô hình đẹp bất thường | **Nghi leakage ngay.** Kiểm tra feature có dùng ngày sau ngày cắt không |
| Trễ lịch | Ưu tiên: EDA có kết luận → baseline → LightGBM → ablation. Bỏ bớt tuning nếu cần |
