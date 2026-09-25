# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Phạm Anh Huy

Công cụ gán nhãn đã dùng: CVAT (AnyLabeling, CVAT, SAM hoặc sửa trực tiếp file nhãn)

Sao chép file này thành `reports/REPORT.md` rồi điền vào các chỗ ĐIỀN. Mọi con số phải truy được
từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc
`outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

Pool và test được chia theo thời gian vì các frame liên tiếp của camera cố định gần như giống nhau; cùng một xe có thể xuất hiện trong nhiều frame gần nhau. Vùng đệm giúp ngăn một xe hoặc một cảnh giao thông gần như y hệt xuất hiện ở cả train và test.
Nếu chia ngẫu nhiên, metric thường bị lệch theo hướng cao hơn thực tế do rò rỉ dữ liệu: model được kiểm tra trên những xe và bối cảnh nó gần như đã thấy khi train.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

Khởi đầu lạnh không khớp tham chiếu chủ yếu ở:
- Xe nhỏ, xa, chỉ còn cụm đèn
- Xe cỡ vừa có thân tối
- Xe bị che một phần hoặc bị cắt ở mép ảnh
- Một số xe đi gần nhau bị tách hoặc gộp box chưa đúng
Recall chi tiết là:
- Small: 0.1818
- Medium: 0.5473
- Large: 0.5610
Như vậy xe nhỏ/xa là điểm yếu rõ nhất. Tuy nhiên, xe vừa và lớn cũng chưa được phát hiện đầy đủ

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

score = W_U·U + W_A·A + W_D·D
- U: độ bất định của các box khó nhất.
- A: số lượng/tỷ lệ box có confidence mơ hồ.
- D: mức đa dạng theo thời gian so với ảnh đã gán.
- Các trọng số quyết định mức đóng góp của từng yếu tố.
MIN_GAP_S ngăn chọn hai frame quá gần nhau, giảm ảnh gần trùng và công gán nhãn lặp lại.
Ba frame có thể dùng để phân tích:
- frame_0182.jpg: rank 1, score 0.9591, U=0.9182, A=1.0, có 18 box mơ hồ — nhiều thông tin nhưng tốn công rà.
- frame_0369.jpg: rank 2, score 0.9324, có 43 box model dò tìm và 16 box mơ hồ — cảnh đông xe, chi phí cao.
- frame_0099.jpg: rank 8, score 0.9063, U=0.9460, có 29 box model dò tìm — ít tốn công hơn và có ca xe bị cắt ở mép ảnh.

Điểm bất định không chứng minh ảnh chắc chắn cải thiện model. Ảnh có thể chứa nhiễu, gần trùng hoặc được gán nhãn không nhất quán.


## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
  `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

| vòng | model | ảnh train | box train | AP50 | Δ AP50 | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 275 | 0.575 | -0.197 | 0.980 | 0.122 | 0.216 | 0.000 | 0.088 | 0.561 |

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

Ca thay đổi sau fine-tune: trong frame_0050, cột vòng 1 có nhiều box vàng hơn cold start, đặc biệt ở xe nhỏ/xa và xe cỡ vừa. Khả năng có thể kiểm tra là confidence bị giảm hoặc model quá khớp với lô train nhỏ; chưa thể kết luận nguyên nhân chỉ từ ảnh.
Ba loại bằng chứng cần tách riêng:
- BLIND_SCAN.md: quan sát độc lập ở frame_0099, cảnh báo xe bị cắt tại góc phải dưới và cụm đèn trắng bên trái dễ bị bỏ sót.
- REVIEW_LOG.csv: lỗi pre-label đã sửa, gồm box chồng ở frame_0107, box chồng ở frame_0227 và xe tải bị thiếu ở frame_0270.
- metrics_round1.json và compare_round1.jpg: kết quả của model sau train. Việc sửa pre-label không tự động chứng minh model sau train đã sửa được cùng lỗi.
Ca khó theo guideline: xe bị cắt ở góc phải dưới frame_0099. Chỉ vẽ phần xe nằm trong ảnh, không kéo box ra ngoài khung và không ôm vệt sáng trên mặt đường. Nếu thân xe tối nhưng còn suy ra được đường biên, box phải ôm phần thân nhìn thấy, không chỉ khoanh hai đèn.


## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Vòng 1 kém hơn cold start về AP50, recall và F1. Precision cao hơn không bù được việc bỏ sót xe tăng mạnh. Nên tạm dừng train thêm để QC nhãn và kiểm tra pipeline trước.
Hai ca có thể cân nhắc cho vòng sau:
- frame_0020.jpg: score 0.8451, U=0.8902, 24 box model dò tìm và 12 box mơ hồ. Chi phí thấp hơn các cảnh quá đông xe và nguy cơ gần trùng với lô vòng 1 thấp.
- frame_0372.jpg: score 0.9101, U=0.9202, 42 box model dò tìm và 15 box mơ hồ. Ảnh nhiều xe, chi phí rà cao và gần frame_0369, nên chỉ chọn nếu thực sự có ca che khuất mới.
Các điểm trên đến từ selection_round1.csv, tức model cold start. Trước khi quyết định vòng sau cần chấm lại pool bằng model vòng 1.
Giới hạn:
- Test chỉ có 20 ảnh và 403 box được tính nên kết quả nhạy với từng cảnh.
- Có 14 box quá nhỏ bị bỏ qua, do đó metric không phản ánh đầy đủ năng lực phát hiện xe cực nhỏ.
- Nhãn tham chiếu do model tạo, chưa được người rà từng box, nên AP50 chỉ đo mức khớp với bộ tham chiếu này.
Nếu AP50 giảm, nên kiểm tra trước:
1. Ghép đúng ảnh–nhãn và class id.
2. Box có thiếu, trùng, vượt biên hoặc không nhất quán với guideline không.
3. Lô train có đúng 12 ảnh và 275 box không.
4. Test và cấu hình đánh giá có giữ nguyên không.
5. Confidence có bị dồn xuống dưới ngưỡng 0.25 không.
6. Checkpoint hoặc cấu hình train có gây quá khớp hay quên kiến thức pretrained không.