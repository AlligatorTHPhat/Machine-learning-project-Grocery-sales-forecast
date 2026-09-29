**Lộ trình từ dữ liệu thô đến báo cáo môn Machine Learning có thực
nghiệm đáng tin cậy**

Tài liệu hướng dẫn thực hiện đề tài dự báo bán lẻ trên dữ liệu thực tế,
với thời lượng chuẩn 10 tuần. Mục tiêu gồm hoàn thành đồ án, giải thích
được cơ chế của mô hình và duy trì khả năng nâng cấp thành một nghiên
cứu hẹp.

  -----------------------------------------------------------------------
  **Kết luận định hướng:** Chọn bài toán dự báo unit_sales theo từng cặp
  cửa hàng - mặt hàng - ngày; dùng seasonal naive làm đối thủ bắt buộc,
  LightGBM hoặc HistGradientBoosting làm mô hình chính, và đánh giá bằng
  rolling-origin validation. Không chia train/test ngẫu nhiên, không dùng
  dữ liệu tương lai để tạo rolling features, không lấy leaderboard làm
  bằng chứng khoa học.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 1 Bài học rút ra từ ba đề tài trước

Ba tài liệu cho thấy năng lực nổi bật nằm ở việc biến nghiệp vụ thành
đặc trưng và pipeline. Điểm cần nâng cấp cho Favorita là kỷ luật thực
nghiệm: baseline đúng, điều kiện so sánh đồng nhất, kiểm soát leakage và
tuyên bố vừa đúng với bằng chứng.

  -----------------------------------------------------------------------
  **Tài liệu**            **Điểm nên kế thừa**    **Điểm phải sửa trong
                                                  Favorita**
  ----------------------- ----------------------- -----------------------
  Báo cáo ML 9 điểm       Feature engineering có  Kết quả gần hoàn hảo
                          lý do nghiệp vụ; so     trên PaySim chịu ảnh
                          sánh nhiều mô hình;     hưởng quy luật mô
                          hiểu precision/recall   phỏng; cần ablation,
                          thay vì accuracy.       validation theo thời
                                                  gian và baseline mạnh.

  Toàn văn NCKH giải ba   Đặt hệ thống hai lớp;   Không so AUC giữa
                          kết nối mô hình với cơ  nghiên cứu/dataset khác
                          chế phản ứng; công khai nhau như bằng chứng
                          hạn chế dữ liệu.        vượt trội; mọi so sánh
                                                  phải cùng dữ liệu, cùng
                                                  split, cùng metric.

  Báo cáo Data Mining 8,3 Business-first,         Không biến tri thức
  cộng điểm lên 10        CRISP-DM, dữ liệu thật, nghiệp vụ thành "ground
                          mô hình nhẹ, giải thích truth"; cần phân biệt
                          được và có đầu ra vận   tín hiệu, giả thuyết và
                          hành.                   nhãn thật. Với
                                                  forecasting, quyết định
                                                  phải dựa trên backtest.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Chuẩn trưởng thành mới:** Không cần thuật toán lạ. Đóng góp của đồ án
  là một quy trình dự báo có thể tái lập, chứng minh được từng cải tiến
  và không rò rỉ tương lai.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 2 Đề tài đăng ký và phạm vi

## 2.1 Tên đề tài đề xuất

**Tên tiếng Việt khuyến nghị:** Dự báo doanh số bán lẻ theo cửa hàng và
mặt hàng bằng học máy trên bộ dữ liệu Corporación Favorita

**Tên tiếng Anh khuyến nghị:** Machine Learning Based Retail Sales
Forecasting at the Store Item Level Using the Corporación Favorita
Dataset

  -----------------------------------------------------------------------
  **Lý do chọn cách đặt tên:** Biến đích unit_sales phản ánh số lượng đã
  bán, vì vậy "dự báo doanh số bán lẻ" là cách gọi chính xác và an toàn
  nhất đối với bằng chứng có trong dữ liệu. "Demand forecasting" có thể
  dùng trong phần bối cảnh nghiệp vụ, nhưng cần nói rõ doanh số quan sát
  được chỉ là đại diện gần đúng của nhu cầu; dữ liệu không cho biết đầy
  đủ lượng cầu bị mất khi hết hàng.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

**Phương án thiên về nghiệp vụ:** Dự báo nhu cầu sản phẩm theo cửa hàng
bằng học máy trên dữ liệu bán lẻ Corporación Favorita / Machine Learning
Based Product Demand Forecasting at the Store Level Using the
Corporación Favorita Retail Dataset. Chỉ dùng phương án này nếu phần
phạm vi định nghĩa unit_sales là biến đại diện cho nhu cầu quan sát
được.

## 2.2 Phát biểu bài toán

-   Đơn vị dự báo: một store_nbr - item_nbr - date.

-   Biến mục tiêu: unit_sales của ngày tương lai; giá trị âm do hoàn trả
    phải được xử lý theo quy tắc được công bố rõ.

-   Chân trời chính: 16 ngày để bám cấu trúc cuộc thi gốc. Nếu khối
    lượng quá lớn, vẫn giữ chân trời 16 ngày nhưng thu hẹp cửa hàng/mặt
    hàng.

-   Ngữ cảnh đầu vào: lịch sử bán hàng, khuyến mãi, metadata cửa
    hàng/mặt hàng, ngày lễ, giá dầu và các biến lịch hợp lệ tại thời
    điểm dự báo.

-   Người dùng giả định: quản lý cần dự kiến lượng hàng để bổ sung tồn
    kho và giảm thiếu hàng/dư hàng. Đồ án chỉ dự báo, không tuyên bố đã
    tối ưu tồn kho.

## 2.3 Ba câu hỏi nghiên cứu

  ----------------------------------------------------------------------------
  **Mã**                  **Câu hỏi**                  **Bằng chứng cần có**
  ----------------------- ---------------------------- -----------------------
  RQ1                     Mô hình ML có vượt           Điểm trung bình và độ
                          seasonal-naive không?        ổn định qua ít nhất 3
                                                       cửa sổ backtest.

  RQ2                     Khuyến mãi, lịch và lịch sử  Ablation: bỏ từng nhóm
                          bán hàng đóng góp bao nhiêu? feature và đo mức giảm
                                                       hiệu năng.

  RQ3                     Hiệu năng khác nhau thế nào  Error analysis theo
                          giữa hàng bán nhanh/chậm,    segment, không chỉ một
                          có/không khuyến mãi và nhóm  con số tổng.
                          perishable/non-perishable?   
  ----------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Không đăng ký quá tay:** Không ghi "tối ưu chuỗi cung ứng", "triển
  khai thời gian thực" hay "dự báo toàn bộ hệ thống" khi sản phẩm mới
  dừng ở notebook. Sản phẩm của học phần là pipeline dự báo và bằng chứng
  thực nghiệm.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 3 Thiết kế thực nghiệm bắt buộc

## 3.1 Cách chia dữ liệu

Dữ liệu thời gian không được xáo trộn. Dùng ba cửa sổ rolling-origin,
mỗi cửa sổ có 16 ngày validation. Ngày cắt cụ thể chỉ được chọn sau khi
kiểm tra min/max date và độ đầy đủ của dữ liệu.

  -----------------------------------------------------------------------
  **Fold**          **Train**         **Validation**    **Mục đích**
  ----------------- ----------------- ----------------- -----------------
  F1                Tất cả ngày trước 16 ngày V1        Kiểm tra một giai
                    V1                                  đoạn tương đối
                                                        bình thường.

  F2                Mở rộng đến trước 16 ngày V2        Kiểm tra độ ổn
                    V2                                  định khi thời
                                                        gian dịch chuyển.

  F3                Mở rộng đến trước 16 ngày V3 gần    Mô phỏng sát nhất
                    V3                cuối train        thời điểm dự báo
                                                        cuối.
  -----------------------------------------------------------------------

-   Fit encoder, scaler, median, category mapping và mọi thống kê chỉ
    trên phần train của từng fold.

-   Mọi lag tại ngày t chỉ được lấy từ t-1 trở về trước. rolling_mean_7
    tại t phải dùng sales.shift(1).rolling(7), không chứa sales ngày t.

-   Giữ một holdout cuối cùng chưa nhìn tới cho lần báo cáo chốt; không
    tinh chỉnh trên holdout.

-   Nếu dự báo recursive nhiều ngày, mô tả rõ ngày thứ 2 trở đi dùng giá
    trị thật hay dự đoán trước đó. Không được vô thức dùng actual future
    sales.

## 3.2 Metric

-   Metric chính: RMSLE hoặc NWRMSLE theo quy tắc cuộc thi. Với NWRMSLE,
    cần tự xác minh trọng số perishable từ mô tả chính thức và viết unit
    test cho công thức.

-   Metric phụ: MAE để đọc sai số theo đơn vị bán; WAPE cho góc nhìn
    nghiệp vụ ở cấp tổng hợp. Không dùng MAPE làm chính vì sales bằng 0
    làm metric bất ổn.

-   Báo cáo mean ± standard deviation qua các fold, kèm kết quả từng
    fold. Một con số ở một lần chia chưa đủ kết luận.

-   Luôn so cùng rows, cùng horizon và cùng preprocessing. Không dùng
    điểm từ một paper có thiết kế khác để tuyên bố kết quả hiện tại vượt
    trội.

## 3.3 Baseline tối thiểu

  -----------------------------------------------------------------------
  **Mã**                  **Baseline**            **Ý nghĩa**
  ----------------------- ----------------------- -----------------------
  B0                      Zero forecast           Mốc kiểm tra dữ liệu
                                                  bán thưa; nếu model
                                                  thua B0 ở nhóm
                                                  slow-moving thì phải
                                                  giải thích.

  B1                      Last value t-1          Persistence; đơn giản
                                                  nhưng có thể mạnh ở
                                                  chuỗi ổn định.

  B2                      Seasonal naive t-7      Dùng doanh số cùng thứ
                                                  trong tuần trước. Đây
                                                  là baseline bắt buộc.

  B3                      Mean các lag 7, 14, 21, Mốc mùa vụ tuần bền hơn
                          28                      một lag đơn.
  -----------------------------------------------------------------------

# 4 Kiến trúc mô hình vừa sức

Lộ trình khuyến nghị là tabular forecasting: biến mỗi điểm
store-item-date thành một hàng đặc trưng. Cách này hợp môn ML, chạy được
trên Kaggle và buộc người học hiểu dữ liệu. Không bắt đầu bằng LSTM,
Transformer, Prophet cho hàng chục nghìn chuỗi.

  --------------------------------------------------------------------------------
  **Tầng**          **Mô hình**            **Vai trò**           **Điều kiện
                                                                 dừng**
  ----------------- ---------------------- --------------------- -----------------
  0                 B0-B3                  Thiết lập mức khó     Chỉ qua khi
                                           thật của bài toán.    metric được
                                                                 unit-test và dự
                                                                 đoán khớp đúng
                                                                 key.

  1                 Ridge/Elastic Net hoặc Mốc ML đơn giản, kiểm Phải chạy toàn bộ
                    HistGradientBoosting   tra pipeline.         3 fold không lỗi.

  2                 Random Forest hoặc     So sánh bagging;      Dừng nếu RAM/thời
                    Extra Trees trên mẫu   không ép chạy toàn bộ gian không hợp
                    có kiểm soát           125M dòng.            lý; ghi nhận chi
                                                                 phí.

  3                 LightGBM regression    Mô hình chính cho     Chỉ tuning sau
                                           tabular lớn, tương    khi B3 và feature
                                           tác phi tuyến và      pipeline đã khóa.
                                           categorical/encoded   
                                           features.             

  4 tùy chọn        Hai-stage demand model Phân loại bán/không   Chỉ làm khi error
                                           bán rồi hồi quy lượng analysis chứng
                                           bán dương.            minh
                                                                 zero-inflation là
                                                                 nút thắt.
  --------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Giới hạn có chủ đích:** Nếu môn học yêu cầu tối thiểu 3 thuật toán,
  dùng Ridge hoặc HistGradientBoosting, Random Forest trên cùng tập mẫu,
  và LightGBM. Deep learning là nhánh bonus, không phải con đường mặc
  định.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 5 Feature engineering theo giả thuyết

  -----------------------------------------------------------------------
  **Nhóm**                **Feature gợi ý**       **Giả thuyết phải tự
                                                  trả lời**
  ----------------------- ----------------------- -----------------------
  Lịch                    day_of_week, week,      Doanh số có chu kỳ tuần
                          month, payday proxy,    hay hiệu ứng đầu/cuối
                          weekend                 tháng không?

  Lịch sử                 lag 1, 7, 14, 28;       Lag nào còn tín hiệu
                          rolling mean/std 7, 14, sau khi kiểm soát store
                          28; số ngày bán gần đây và item?

  Khuyến mãi              onpromotion hiện tại;   Tác động promotion khác
                          promo rate quá khứ;     nhau giữa nhóm hàng
                          tương tác promotion ×   nào?
                          family                  

  Mặt hàng                family, class,          Có leakage nếu target
                          perishable; mean        encoding dùng cả
                          encoding quá khứ        validation không?

  Cửa hàng                city, state, type,      Cluster cửa hàng có
                          cluster; store          giải thích sai số hay
                          historical level        chỉ là ID thay thế?

  Sự kiện                 holiday type, locale,   Ngày transferred nên
                          transferred, days       hiểu như ngày lễ thật
                          to/from holiday         hay ngày được chuyển?

  Kinh tế                 oil price đã            Giá dầu có cải thiện
                          forward-fill theo ngày  ngoài biến lịch hay chỉ
                          quá khứ                 tạo tương quan giả?

  Tương tác               store × family; item ×  Tương tác nào có lý do
                          promo; dow × family     nghiệp vụ và đủ dữ
                                                  liệu?
  -----------------------------------------------------------------------

-   Không forward-fill sales của ngày thiếu như thể đó là doanh số thật
    nếu chưa hiểu ý nghĩa thiếu dữ liệu.

-   Không dùng transactions của ngày cần dự báo nếu tại thời điểm dự báo
    chưa biết số giao dịch tương lai. Chỉ dùng lag/rolling của
    transactions hoặc loại bỏ.

-   Không tạo "average sales per item" trên toàn bộ dataset. Mọi
    aggregate target phải được tính theo quá khứ của từng fold.

-   Không one-hot hàng chục nghìn item_nbr nếu gây nổ bộ nhớ; cân nhắc
    categorical native, frequency encoding hoặc historical aggregates
    leakage-safe.

# 6 Lộ trình 10 tuần

  --------------------------------------------------------------------------------------------
  **Tuần**          **Trọng tâm**     **Đầu ra bắt buộc**                    **Cổng kiến
                                                                             thức**
  ----------------- ----------------- -------------------------------------- -----------------
  Tuần 1            Đọc đề và audit   Data dictionary tự viết; sơ đồ quan hệ Giải thích được
                    dữ liệu           7 bảng; báo cáo min/max date, rows,    vì sao train
                                      missing, duplicates, memory.           không thể load
                                                                             ngây thơ và
                                                                             trường nào biết
                                                                             trước ở thời điểm
                                                                             forecast.

  Tuần 2            Định nghĩa bài    Problem statement một trang; script    Tính tay được
                    toán và metric    metric có 3 unit tests; danh sách      RMSLE cho ví dụ
                                      feature allowed/forbidden.             nhỏ; phân biệt
                                                                             target, feature,
                                                                             identifier.

  Tuần 3            EDA theo thời     6-8 biểu đồ: tổng sales, zero/negative Không kết luận
                    gian              rate, promo effect, weekday,           nhân quả từ biểu
                                      store/family, holiday; 3 phát hiện có  đồ; chỉ nêu quan
                                      bằng chứng.                            sát và giả
                                                                             thuyết.

  Tuần 4            Baseline và       B0-B3 trên ba fold; bảng mean ± std;   Seasonal naive
                    validation        lưu predictions theo key.              chạy đúng; kiểm
                                                                             tra ngẫu nhiên 20
                                                                             dòng để chứng
                                                                             minh không lệch
                                                                             key.

  Tuần 5            Pipeline feature  Module tạo calendar, lag, rolling;     Thay sales
                    v1                unit test chống leakage; dataset       validation bằng
                                      matrix có schema cố định.              giá trị cực lớn
                                                                             mà feature
                                                                             validation không
                                                                             đổi đối với
                                                                             feature không
                                                                             được phép nhìn
                                                                             tương lai.

  Tuần 6            Mô hình ML v1     Ridge/HGB và RF mẫu; cùng              Giải thích vì sao
                                      folds/metrics; log thời gian và RAM.   một mô hình
                                                                             thắng/thua dựa
                                                                             trên bias, phi
                                                                             tuyến, scale hoặc
                                                                             sparsity.

  Tuần 7            LightGBM và       Một cấu hình mặc định + 8-15 trials có Không tuning trên
                    tuning nhỏ        ngân sách; early stopping; feature     holdout; không
                                      importance.                            dùng leaderboard
                                                                             để chọn tham số.

  Tuần 8            Ablation và error Bảng                                   Chỉ giữ feature
                    analysis          -history/-promo/-calendar/-metadata;   group có lợi ổn
                                      lỗi theo store, family, promo,         định hoặc có giải
                                      perishable và sales volume.            thích hợp lý.

  Tuần 9            Hoàn thiện và tái Pipeline one-command hoặc notebook     Một người khác
                    lập               đánh số; seed, requirements, config;   chạy từ raw data
                                      rerun sạch.                            tới bảng kết quả
                                                                             mà không sửa tay
                                                                             đường dẫn.

  Tuần 10           Báo cáo và bảo vệ Báo cáo, slide, demo; 20 câu hỏi phản  Mọi con số trong
                                      biện; limitation và future work cụ     kết luận truy
                                      thể.                                   được về file kết
                                                                             quả; không
                                                                             overclaim.
  --------------------------------------------------------------------------------------------

# 7 Hướng dẫn thao tác theo từng giai đoạn

## Giai đoạn 0 Khóa phạm vi

1.  Kiểm tra tài nguyên Kaggle: RAM, CPU/GPU, thời gian phiên. Chọn một
    trong ba scope: toàn dữ liệu tối ưu bộ nhớ; 10-20 cửa hàng; hoặc top
    N item theo doanh số. Scope nhỏ vẫn phải giữ tính thời gian và nhiều
    chuỗi.

2.  Viết pre-registration ngắn: RQ, folds, metric chính, baselines, mô
    hình chính, tiêu chí dừng. Mọi thay đổi sau đó ghi vào experiment
    log.

3.  Tạo cấu trúc repo: data/raw không commit; src/data.py;
    src/features.py; src/metrics.py; src/models.py; notebooks/01-05;
    reports/figures; configs; README.

## Giai đoạn 1 Hiểu bảng dữ liệu

4.  Đọc từng file: train, test, items, stores, transactions, oil,
    holidays_events. Tự vẽ khóa join và cardinality trước khi merge.

5.  Kiểm tra duplicate trên date-store-item; tỷ lệ negative sales;
    missing oil; holiday transferred; item/store xuất hiện mới; độ phủ
    onpromotion.

6.  Ước lượng RAM bằng sample 100 nghìn dòng rồi nhân tỷ lệ. Chỉ định
    dtype khi đọc: integer nhỏ, float32 khi phù hợp, category cho chuỗi
    lặp.

## Giai đoạn 2 Baseline trước mô hình

7.  Tạo validation dates trước, rồi cắt train. Viết B0-B3 bằng
    groupby/shift; tuyệt đối không merge dự đoán theo vị trí dòng.

8.  Đánh giá ở mức tổng và segment. Vẽ actual-vs-predicted cho 10 chuỗi
    được lấy mẫu có seed; thêm histogram residual theo log scale.

9.  Nếu B3 rất mạnh, coi đó là phát hiện quan trọng. ML chỉ có giá trị
    khi vượt B3 ổn định, không phải chỉ ở một fold.

## Giai đoạn 3 Feature pipeline

10. Viết một hàm nhận cutoff_date và chỉ đọc lịch sử trước cutoff. Đây
    là ranh giới chống leakage.

11. Sinh feature theo batch/date hoặc dùng Polars/pandas tối ưu bộ nhớ.
    Lưu feature matrix theo Parquet để tránh chạy lại toàn bộ.

12. Mỗi nhóm feature có một giả thuyết và một thí nghiệm ablation dự
    kiến. Không tạo hàng chục feature chỉ vì có thể.

## Giai đoạn 4 Mô hình và tuning

13. Chạy mô hình tuyến tính/cây đơn giản trước để bắt lỗi. Sau đó mới
    chạy LightGBM với objective regression phù hợp và early stopping.

14. Chỉ tuning vài tham số tác động lớn: learning_rate,
    num_leaves/max_depth, min_data_in_leaf, feature_fraction,
    bagging_fraction. Giữ ngân sách trials cố định.

15. Lưu config, seed, thời gian fit, peak RAM, metric từng fold và
    artifact model. Chọn model theo trung bình fold và độ ổn định, không
    theo fold đẹp nhất.

## Giai đoạn 5 Phân tích và viết

16. Trả lời RQ1 bằng bảng model vs baseline; RQ2 bằng ablation; RQ3 bằng
    segment errors.

17. Chọn 3 case study: dự báo tốt, forecast thiếu khi promotion/holiday,
    forecast dư ở slow-moving. Mỗi case có biểu đồ và giải thích có điều
    kiện.

18. Viết hạn chế thật: dữ liệu Ecuador giai đoạn cũ; không có tồn
    kho/lost sales; observational promotion; khó suy ra nhu cầu khi hết
    hàng; không kiểm chứng chi phí tồn kho.

# 8 Cách dùng AI mà vẫn học thật

AI chỉ nên được sử dụng như gia sư, công cụ phản biện và hỗ trợ gỡ lỗi;
các quyết định nghiên cứu vẫn phải được giải thích bằng bằng chứng. Mỗi
prompt cần kèm mục tiêu, schema hoặc đoạn code nhỏ, điều đã thử, kết quả
và câu hỏi cụ thể.

  -----------------------------------------------------------------------
  **Mức**                 **Được phép hỏi AI**    **Không nên hỏi**
  ----------------------- ----------------------- -----------------------
  1 Hiểu                  Giải thích một khái     "Làm toàn bộ project
                          niệm bằng ví dụ 5 dòng; Favorita cho tôi."
                          hỏi kiểm tra hiểu sai.  

  2 Thiết kế              Yêu cầu nêu 3 lựa chọn  Xin AI chọn thuật toán
                          và trade-off; xin       mà không nêu dữ liệu,
                          checklist leakage.      metric, compute.

  3 Code                  Xin pseudocode hoặc     Copy notebook winning
                          review một hàm; yêu cầu solution rồi đổi tên
                          viết unit test cho lỗi  biến.
                          đã mô tả.               

  4 Phản biện             Đóng vai giảng viên hỏi Xin viết kết luận trước
                          10 câu; chỉ ra claim    khi có bảng kết quả.
                          chưa có bằng chứng.     
  -----------------------------------------------------------------------

## 8.1 Prompt mẫu có kiểm soát

-   "Tôi đang dự báo store-item-day với horizon 16 ngày. Đây là schema
    và cutoff. Hãy hỏi tôi 5 câu để tự phát hiện leakage; chưa đưa lời
    giải."

-   "Đây là hàm lag feature của tôi. Chỉ đánh dấu dòng nào có thể dùng
    dữ liệu ở t hoặc tương lai, giải thích bằng một ví dụ 4 ngày; không
    viết lại toàn bộ hàm."

-   "B3 seasonal-naive thắng Random Forest ở 3/3 fold. Hãy đưa 4 giả
    thuyết có thể kiểm chứng và thí nghiệm phân biệt chúng; không kết
    luận thay tôi."

-   "Đây là bảng ablation. Hãy đóng vai phản biện, hỏi những bằng chứng
    còn thiếu. Không viết phần thảo luận hoàn chỉnh."

-   "Tôi giải thích RMSLE như sau: \<đoạn của tôi\>. Chấm đúng/sai từng
    ý và cho một bài tính tay mới để tôi tự làm."

  -----------------------------------------------------------------------
  **Luật 30 phút:** Trước khi hỏi AI về lỗi, cần ghi rõ kết quả mong đợi,
  kết quả thực tế và hai giả thuyết đã thử. Sau khi nhận phản hồi, cần tự
  diễn giải lại và viết một test nhỏ.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 9 Bảng thí nghiệm tối thiểu

  ----------------------------------------------------------------------------
  **ID**            **Cấu hình**           **Mục tiêu**      **Kết quả phải
                                                             lưu**
  ----------------- ---------------------- ----------------- -----------------
  E00               B0 zero                Sanity check      3 fold × metrics
                                           sparse demand     × segments

  E01               B1 t-1                 Persistence       Như trên

  E02               B2 t-7                 Weekly            Như trên
                                           seasonality       

  E03               B3 mean lag tuần       Baseline mạnh     Như trên

  E10               ML đơn giản + calendar Pipeline smoke    Metric, runtime,
                                           test              RAM

  E11               ML + history           Giá trị           Delta so E10
                                           lag/rolling       

  E12               ML + promo             Giá trị promotion Delta theo promo
                                                             segment

  E13               ML +                   Giá trị context   Ablation từng
                    metadata/holiday/oil                     nhóm

  E20               LightGBM mặc định      Mốc mô hình chính 3 folds

  E21-E35           Tuning có ngân sách    Tìm vùng tham số  Trial log đầy đủ

  E40               Best model trên        Ước lượng cuối    Chỉ chạy khi khóa
                    holdout                                  thiết kế
  ----------------------------------------------------------------------------

# 10 Tiêu chí hoàn thành

  -----------------------------------------------------------------------
  **Mức**                             **Yêu cầu**
  ----------------------------------- -----------------------------------
  Đạt học phần                        B0-B3; ít nhất 3 mô hình ML theo
                                      yêu cầu môn; 3 time folds; metric
                                      đúng; report tái lập; giải thích
                                      giới hạn.

  Tốt                                 LightGBM vượt baseline ổn định;
                                      ablation; error analysis theo
                                      segment; unit tests leakage; log
                                      tài nguyên.

  Có mầm nghiên cứu                   Một câu hỏi hẹp có bằng chứng mới:
                                      robustness qua chế độ
                                      promotion/holiday, cold-start,
                                      hierarchical reconciliation hoặc
                                      uncertainty; so sánh công bằng và
                                      có statistical test/CI phù hợp.
  -----------------------------------------------------------------------

## 10.1 Checklist trước khi nộp

-   \[ \] Không có random train_test_split.

-   \[ \] Mọi rolling/aggregate target đều shift và fit theo train fold.

-   \[ \] Baseline seasonal được chạy trên đúng horizon và đúng key.

-   \[ \] Metric có unit test và khớp mô tả cuộc thi.

-   \[ \] Có mean ± std qua ít nhất 3 folds.

-   \[ \] Tuning không nhìn holdout cuối.

-   \[ \] Bảng kết quả cùng dataset, split, metric.

-   \[ \] Kết luận trả lời đúng RQ, không tuyên bố tối ưu tồn kho.

-   \[ \] README ghi cách chạy, seed, thư viện và cấu trúc output.

-   \[ \] Tự giải thích được ít nhất 20 dòng quan trọng nhất của feature
    pipeline.

# 11 Phân công thực hiện và hỗ trợ kỹ thuật

  -----------------------------------------------------------------------
  **Phần trực tiếp thực   **Phần hỗ trợ kỹ        **Ranh giới**
  hiện**                  thuật**                 
  ----------------------- ----------------------- -----------------------
  Đọc data dictionary;    Thiết lập cổng kỹ       Không bàn giao notebook
  xây dựng baseline; giải thuật; rà soát leakage; hoàn chỉnh để sao chép.
  thích metric; trình bày kiểm tra tính công bằng Mỗi PR hoặc notebook
  EDA; thực hiện ít nhất  của thí nghiệm; phản    cần có ba mục: nội dung
  một mô hình và một      biện kết luận; hỗ trợ   đã hiểu, nội dung chưa
  ablation.               dựa trên log lỗi.       hiểu và bằng chứng hiện
                                                  có.

  -----------------------------------------------------------------------

Nhịp review đề xuất: hai buổi mỗi tuần, mỗi buổi 30-45 phút. Buổi 1
duyệt câu hỏi và thiết kế trước khi chạy; buổi 2 duyệt bằng chứng sau
khi chạy. Không review bằng ảnh chụp điểm số đơn lẻ: phải có code
version, config và bảng kết quả.

# 12 Câu hỏi bảo vệ phải tự trả lời

-   Vì sao không được chia ngẫu nhiên dữ liệu Favorita?

-   Seasonal naive khác last-value thế nào và vì sao có thể rất mạnh?

-   RMSLE phạt over-forecast và under-forecast ra sao? Vì sao cần xử lý
    dự đoán âm?

-   Feature transactions ngày tương lai có hợp lệ không?

-   Một rolling mean bị leakage như thế nào?

-   Vì sao kết quả trên một fold chưa đủ?

-   LightGBM có lợi thế gì so với Random Forest ở dữ liệu này?

-   Promotion correlation có chứng minh promotion gây tăng sales không?

-   Nếu item chưa từng xuất hiện ở train nhưng có ở validation thì xử lý
    thế nào?

-   Nếu model tổng thể tốt nhưng rất tệ với perishable thì chọn model
    nào?

-   Vì sao không thể đồng nhất observed sales với true demand khi có
    stockout?

-   Ablation chứng minh điều gì và không chứng minh điều gì?

-   Tại sao không so trực tiếp điểm của nhóm với một paper dùng split
    khác?

-   Scope thu hẹp còn đại diện cho câu hỏi nghiên cứu không?

-   Nếu B3 thắng tất cả ML, đồ án có thất bại không? Kết luận khoa học
    đúng là gì?

# 13 Nguồn khởi đầu và quy tắc đọc

Ưu tiên nguồn chính thức trước notebook cộng đồng. Notebook tốt chỉ dùng
để học kỹ thuật triển khai sau khi nhóm đã tự hoàn thành baseline và
validation.

-   Kaggle competition overview và data:
    https://www.kaggle.com/competitions/favorita-grocery-sales-forecasting
    và
    https://www.kaggle.com/competitions/favorita-grocery-sales-forecasting/data

-   Scikit-learn TimeSeriesSplit và cross-validation:
    https://scikit-learn.org/stable/modules/cross_validation.html

-   LightGBM documentation: https://lightgbm.readthedocs.io/

-   Paper về giải pháp WaveNet hạng 2, chỉ đọc sau baseline:
    https://arxiv.org/abs/1803.04037

  -----------------------------------------------------------------------
  **Nhiệm vụ mở đầu:** Trong 48 giờ đầu, hoàn thành một trang trả lời 8
  câu: target là gì, grain là gì, horizon là gì, bảng nào join bằng khóa
  nào, trường nào biết trước, metric là gì, baseline mạnh nhất dự kiến là
  gì, và ba rủi ro leakage. Chưa xây dựng mô hình trước khi qua bước này.
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

# 14 Quyết định cuối cùng

Hướng phù hợp nhất là một đồ án forecasting tabular có kỷ luật thực
nghiệm, không phải cuộc đua mô hình phức tạp. Khi hoàn thành baseline
đúng, three-fold backtest, pipeline feature chống leakage, LightGBM,
ablation và error analysis, đề tài sẽ vượt khỏi dạng bài "so sánh vài
thuật toán" thông thường. Nhánh nghiên cứu chỉ mở sau khi phần lõi có
thể tái lập.
