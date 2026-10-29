# Đánh giá mô hình — 20261029_225329

Đã kiểm tra khóa trùng, số dòng, ngày dự báo, trọng số và điểm tính lại.

- V1_last16: safe 0.614003; baseline 0.638588; cải thiện 3.85%.
- V2_seasonal: safe 0.586527; baseline 0.607533; cải thiện 3.46%.

Mô hình safe tốt hơn baseline trên mọi fold.
- V1_last16, nhóm 1: mô hình kém baseline; cần xem kỹ nhóm này.
- V2_seasonal, nhóm 1: mô hình kém baseline; cần xem kỹ nhóm này.

Chưa đánh giá đủ nhóm cặp mới và mặt hàng mới ở: V1_last16, V2_seasonal.

Chưa có điểm test. Hai fold đã dùng để so sánh mô hình; không phải kiểm tra độc lập.
Chưa thử đầy đủ cách xử lý ngày không có dòng. Khuyến mãi tương lai vẫn cần kiểm chứng.

Có thể viết 07 để tổng hợp kết quả và các giới hạn trên.
