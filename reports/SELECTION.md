# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có
ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét
ảnh gần trùng hoặc trường hợp model không dự đoán được box: ĐIỀN

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet: 
Ba frame thuộc lô 12 ảnh model chọn và bằng chứng:
- frame_0182.jpg: rank 1, điểm 0.9591, có 28 box model dò tìm và 18 box mơ hồ. Contact sheet cho thấy nhiều xe có thân tối và cụm đèn sát nhau.
- frame_0369.jpg: rank 2, điểm 0.9324, có 43 box model dò tìm và 16 box mơ hồ. Contact sheet cho thấy đây là cảnh đông xe, có nhiều xe gần nhau và dễ bị bỏ sót hoặc gộp box.
- frame_0099.jpg: rank 8, điểm 0.9063, U=0.9460, có 29 box model dò tìm và 14 box mơ hồ. Ảnh có xe bị cắt ở mép dưới và các cụm đèn xa khó xác định.


Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do: frame_0372.jpg. Frame này xếp hạng 6, điểm 0.9101, ở 148.8 s, có 42 box model dò tìm và 15 box mơ hồ. Tuy nhiên, nó chỉ cách frame_0369.jpg ở 147.6 s một khoảng ngắn nên bị loại bởi luật giãn cách thời gian.

Điều phép chọn này chưa chứng minh về chất lượng mô hình: Điểm cao chỉ cho biết model hiện tại đang phân vân, không chứng minh frame đó chứa thông tin hữu ích hoặc nhãn sau khi sửa sẽ giúp model tổng quát tốt hơn
