Diễn giải nghiệp vụ:
holiday_events(date [str], type[str], locale[str], locale_name[str], description[str], transferred[bool])
- date: ngày diễn ra sự kiện
- type: loại sự kiện
- locale: vùng miền diễn ra sự kiện
- locale_name: tên vùng miền diễn ra sự kiện
- description: mô tả sự kiện
- transferred: sự kiện có được chuyển sang ngày khác không - nghĩa là ngày nghỉ nhưng có bị chuyển dời sang ngày khác đê nghỉ bù không

oil(date [str], dcoilwtico[float64]):
- date: ngày giá dầu được ghi nhận
- dcoilwtico: giá dầu thô WTI

items(item_nbr[int64], family[str], class[int64], perishable[int64]):
- item_nbr: số thứ tự mặt hàng
- family: họ mặt hàng
- class: nhóm mặt hàng
- perishable: hàng dễ hỏng, trọng số là 1.25, nếu không là 1.0

stores(store_nbr[int64], city[str], state[str], type[str], cluster[int64]):
- store_nbr: số thứ tự cửa hàng
- city: thành phố
- state: bang
- type: loại cửa hàng
- cluster: cụm cửa hàng có cấu trúc giống nhau

sample_submission(id[int64], unit_sales[float64]):
- id: id của mẫu
- unit_sales: số lượng bán

transactions(date[str], store_nbr[int64], transactions[int64])
- date: ngày giao dịch
- store_nbr: số thứ tự cửa hàng
- transactions: số lượng giao dịch

train(id[int64], date[str], store_nbr[int64], item_nbr[int64], unit_sales[float64], onpromotion[bool])
- id: id của mẫu
- date: ngày diễn ra giao dịch
- store_nbr: số thứ tự cửa hàng
- item_nbr: số thứ tự mặt hàng
- unit_sales: số lượng bán
- onpromotion: có khuyến mãi hay không
