# Mini guideline — bài làm cá nhân

Ghi lại cách hiểu bốn lớp và những tình huống dễ nhầm để áp dụng nhất quán khi gán nhãn trên CVAT.

## Định nghĩa lớp

| ID | Lớp | Cách hiểu và điểm cần phân biệt |
| ---: | --- | --- |
| 0 | `car` | Xe con, sedan, SUV, taxi và xe bán tải. Theo schema của bài, xe bán tải vẫn là `car`, dù có thùng chở đồ phía sau. |
| 1 | `truck` | Xe tải chở hàng, xe ben hoặc xe đầu kéo. Phân biệt với xe bán tải thuộc `car`; không chọn lớp chỉ vì thấy xe có kích thước lớn. |
| 2 | `bus` | Xe buýt và minibus chở khách. Xe buýt có khớp nối vẫn được tính là một xe. Nếu khó phân biệt minibus với van thì ghi lại để hỏi labcoach. |
| 3 | `van` | Xe van, minivan chở hàng hoặc chở người, thường có khoang xe dạng hộp liền với cabin. Không tự đổi sang `bus` chỉ vì xe chở nhiều người. |

Lớp được chọn theo loại xe. Xe đang đỗ hay đang chạy, quay đầu hay bị che một phần vẫn dùng cùng một cách phân loại.

## Quy tắc chung

- **Box bám sát xe:** đặt các cạnh sát phần xe nhìn thấy, không lấy thêm khoảng đường trống, bóng đổ hoặc xe bên cạnh. Mỗi xe có một box riêng.
- **Xe bị che:** vẫn gán nếu còn nhận ra loại xe. Box bao phần nhìn thấy, không kéo rộng để đoán phần bị che. Nếu cùng một xe lộ ra ở hai bên vật che thì dùng một box bao các phần đó.
- **Xe bị cắt khung:** vẽ tới đúng mép ảnh, chỉ lấy phần xe còn nằm trong khung.
- **Xe nhỏ hoặc ảnh nhòe:** vẫn gán khi xác định được lớp. Nếu chỉ thấy một mảnh nhỏ, không đủ phân biệt loại xe thì ghi tên ảnh và vị trí để hỏi labcoach, không đoán cho đủ số lượng.
- **Xe đỗ và xe chồng nhau:** xe đỗ vẫn cần nhãn. Hai xe chồng lên nhau thì tách hai box nếu nhận diện được từng xe.
- **Ngoài bốn lớp:** bỏ qua người, xe máy, xe đạp, biển báo, hình xe trên quảng cáo và xe phản chiếu trong kính/gương.
- **Khi dùng YOLO hỗ trợ:** kiểm tra lại từng box gợi ý, sửa lớp hoặc vị trí nếu cần và bổ sung xe bị bỏ sót. Lưu ý model pretrained không có lớp `van` riêng. Chạy lại trên ảnh đã làm có thể sinh box trùng.

## Ba tình huống cần xử lý nhất quán

| Ảnh | Tình huống | Cách xử lý | Lý do |
| --- | --- | --- | --- |
| [drive_001.jpg](dataset/images/train/drive_001.jpg) | Xe con trắng ở nửa phải ảnh, phía dưới thanh ngang, bị cột dọc che một phần. | Chọn `car`, vẽ một box quanh phần xe còn thấy; không mở rộng sang phần bị cột che để đoán kích thước đầy đủ. | Phần xe còn thấy đủ nhận ra xe con. Bị che không phải lý do bỏ nhãn hoặc đổi lớp. |
| [drive_020.jpg](dataset/images/train/drive_020.jpg) | Xe tối màu ở mép phải, nửa dưới ảnh bị cắt mất một phần. | Chọn `car`, đặt box sát phần thân và đuôi còn trong ảnh, cạnh phải chạm mép ảnh. | Chỉ gán phần nhìn thấy; không suy ra phần xe nằm ngoài khung. |
| [drive_040.jpg](dataset/images/train/drive_040.jpg) | Xe buýt dài màu vàng, trắng và xanh ở giữa-phải ảnh có khớp nối. | Chọn `bus`, dùng một box cho toàn bộ xe nhìn thấy, gồm cả đoạn khớp nối. | Hai phần thân nối liền thuộc cùng một xe, không phải hai xe riêng. |

## Điểm cần hỏi lại khi gặp

Ranh giới giữa minibus và van dễ nhầm, nhất là khi xe ở xa hoặc chỉ thấy phía sau. Với ca chưa chắc, cần ghi tên ảnh và vị trí xe để hỏi labcoach. Chưa có kết luận riêng của labcoach về các ca này được ghi lại trong guideline.

Khi self-QC, dùng các quy tắc trên để kiểm lại nhãn thực tế: đủ xe, đúng lớp, box sát và không trùng. Việc điền guideline không đồng nghĩa đã rà xong toàn bộ bộ nhãn.
