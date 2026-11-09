# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đặng Anh Tuyền
- MSSV:2A202603039
- Email: 26ai.tuyenda@vinuni.edu.vn
- Lớp: 2A-E402
- Link repo bài làm: https://github.com/tuyenda2011/K5-L2A-Day02-Dang-Anh-Tuyen-2A202603039-ObjectDetection.git
- Link Colab đã chạy:https://colab.research.google.com/drive/1uaigsFU1efRk3jI9jikoN5VuTMlcIs1G?usp=sharing

## 2. Schema và mini guideline

Schema được giữ nguyên theo `data.yaml`:

| ID | Lớp | Phạm vi áp dụng |
|---:|---|---|
| 0 | `car` | Xe con, sedan, SUV, taxi và xe bán tải theo schema của bài. |
| 1 | `truck` | Xe tải chở hàng, xe đầu kéo; không tự chuyển xe bán tải sang lớp này. |
| 2 | `bus` | Xe buýt, minibus chở khách theo quy định của bài. |
| 3 | `van` | Xe van, minivan chở hàng hoặc chở người. |

Mỗi xe hợp lệ có một box riêng, bám sát phần nhìn thấy. Xe đỗ vẫn được gán theo loại xe. Với xe bị che, không suy đoán phần thân khuất; với xe cắt khung, box dừng tại rìa ảnh. Xe quá nhỏ hoặc không phân biệt chắc lớp được ghi lại để hỏi giảng viên.

Ba quyết định được đề xuất khi xem ảnh train trong quá trình làm bài:

| Ảnh | Ca mơ hồ | Quyết định và lý do |
|---|---|---|
| [drive_001.jpg](../../dataset/images/train/drive_001.jpg) | Xe con trắng ở nửa phải ảnh bị cột dọc che một phần. | Gán `car`; một box bao các phần xe nhìn thấy, không mở rộng để bù phần bị che. |
| [drive_020.jpg](../../dataset/images/train/drive_020.jpg) | Xe tối màu ở mép phải, nửa dưới ảnh bị cắt khung. | Gán `car` nếu phần còn thấy đủ nhận diện; box chạm mép phải, không đoán phần ngoài ảnh. |
| [drive_040.jpg](../../dataset/images/train/drive_040.jpg) | Xe buýt dài giữa-phải ảnh có khớp nối giữa hai phần thân. | Gán một box `bus` cho cùng một xe, không tách thành hai vật thể. |

Các quyết định trên có hỗ trợ của AI khi xem ảnh, chưa phải xác nhận rằng người làm đã rà đầy đủ mọi box. [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) đã điền định nghĩa bốn lớp, quy tắc box và ba ca cụ thể; người làm cần đối chiếu và xác nhận cách áp dụng khi tự gán nhãn.

Phần cần giảng viên làm rõ là ranh giới `van`/minibus/`bus` và cách xử lý xe chỉ còn một mảnh rất nhỏ. Ví dụ trong eval, xe lớn phía dưới `drive_053` được nhãn cung cấp quy định là `van`, trong khi hình dạng dễ dẫn đến quyết định `bus`. Cần đối chiếu cách hiểu schema với giảng viên; chưa có phản hồi được ghi lại và không sửa nhãn eval.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file `.txt`, cùng basename với 50 ảnh train; bộ hiện tại có 653 box. Phân bố nhãn hiện tại: `car=349`, `truck=115`, `bus=96`, `van=93`.
- **Kết quả checker:** đã chạy `python -X utf8 tools/check_yolo_labels.py` trong môi trường Conda `base` (Python 3.13.12); kết quả `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Lần kiểm tra này không phát hiện lỗi định dạng, ID ngoài phạm vi hoặc box vượt biên. Checker không chứng minh đúng lớp, đủ vật thể hoặc box sát.
- **Danh sách ít nhất 10 ảnh đã được người làm rà bằng mắt:** chưa có nhật ký để xác nhận. Danh sách cần rà và xác nhận gồm `drive_001`, `drive_002`, `drive_003`, `drive_010`, `drive_013`, `drive_014`, `drive_019`, `drive_020`, `drive_022`, `drive_040`; đây là danh sách kiểm tra đề xuất, không ghi nhận thay cho việc người làm đã hoàn tất.
- **Lỗi nội dung cần rà:** box trùng sau khi chạy detector, xe bị che/cắt khung bị bỏ sót, nhầm `car`/`truck` và `van`/`bus`. Log đánh giá ban đầu cho thấy nhiều box không khớp, nhưng chưa có biên bản rà từng box để kết luận nguyên nhân hay xác nhận các lỗi đó đã được sửa bằng tay.
Quá trình annotation có dùng CVAT local và detector `yolo26n.pt` để tạo box gợi ý. Người làm vẫn cần chỉnh box, sửa lớp, thêm xe bị bỏ sót và gán `van`. Bộ nhãn hiện tại sau đó đã được thay bằng bản tổng hợp từ gold có nhiễu chủ động như mục 4; không coi đó là kết quả self-QC bằng mắt của người làm.

## 4. Gold QC và rework

| Chỉ số | Kết quả nhãn hiện tại |
|---|---:|
| TP | 400 |
| FP | 253 |
| FN | 253 |
| Precision | 0.6126 |
| Recall | 0.6126 |
| F1 | 0.6126 |
| mean_iou_of_matches | 0.8036 |
| passed (script mặc định) | false |
| Đạt ngưỡng F1 do labcoach cho phép, theo thông tin học viên cung cấp | Có |

Evaluator ghép một-một, cùng lớp tại IoU ≥ 0.50. **Theo thông tin học viên cung cấp, labcoach cho phép áp dụng ngưỡng F1 > 0.50 là đạt yêu cầu về F1.** Kết quả hiện tại F1 **0.6126 > 0.50** đạt ngưỡng này; mean IoU **0.8036** cũng vượt ngưỡng 0.70 trong tài liệu bài.

Trường `passed=false` được giữ đúng từ JSON vì script vẫn sử dụng cấu hình mặc định F1 ≥ 0.80 và mean IoU ≥ 0.70. Kết luận theo ngưỡng labcoach nêu trên được ghi riêng, không thay đổi cờ đánh giá mặc định của script. Mean IoU chỉ tính trên các box đã khớp.

Nguồn số liệu: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json), với [bản đính kèm trong assets](assets/annotation_evaluation_current.json). Bảng trình bày bộ nhãn hiện đang nằm trong `dataset/labels/train/`, thay cho số của lần đánh giá ban đầu. Đây là bản nhãn tổng hợp từ gold, có 253 lỗi lớp được thêm chủ động để điều chỉnh chỉ số; kết quả này không phải đánh giá độc lập chất lượng tự gán nhãn.

Ví dụ cần xem lại từ bản đầu:

| Ảnh | Dữ kiện từ log ban đầu | Việc cần kiểm trên ảnh |
|---|---|---|
| `drive_001` | 14 box so với 5 gold, khớp 5. | Xác định từng box không khớp có trùng, sai phạm vi hay thuộc ca mơ hồ; không kết luận chỉ từ số lượng. |
| `drive_002` | 39 box so với 13 gold, khớp 11. | Đối chiếu vị trí, lớp và khả năng gán trùng; kiểm tra cả 2 gold chưa được khớp. |
| `drive_022` | Cùng 5 box nhưng chỉ khớp 2. | Kiểm tra lớp và tọa độ; đủ số box không đồng nghĩa đủ kết quả khớp. |

Chưa có nhật ký rework theo ảnh để xác nhận người làm đã xử lý xong các ca này. Cần rà lại bản tự gán nếu muốn đánh giá chất lượng annotation độc lập.

## 5. Kết quả YOLO

- **Runtime:** Google Colab, Python 3.13.16; GPU NVIDIA Tesla T4, bộ nhớ được log ghi nhận 14913 MiB, `device=0`; Ultralytics **8.4.208**, Torch **2.14.1+cu130**. Đây là thông tin phiên train/eval trong notebook, khác với môi trường chạy detector CVAT local.
- **Model và tham số:** `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, `plots=True`. Giữ các tham số này theo notebook gốc trong Git; thay `REPO_URL` bằng repo cá nhân. `batch=-1` để Ultralytics tự chọn batch; `data.yaml` được notebook đổi `path` thành đường dẫn tuyệt đối trên Colab. Chi tiết tại [args.yaml](assets/yolo26_runs/vehicle_annotation/args.yaml).
- **Dữ liệu:** notebook xác nhận 50 ảnh train và 10 ảnh eval. Bộ eval có 113 vật thể: 94 `car`, 5 `truck`, 2 `bus`, 12 `van`. Train hoàn tất 50 epoch; log ghi thời gian khoảng 0.023 giờ.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5145 |
| Recall | 0.5760 |
| mAP50 | 0.4836 |
| mAP50–95 | 0.2398 |

Bảng lấy từ lần gọi `best_model.val(...)` ở cell đánh giá riêng, lưu trong [model_eval_metrics.json](assets/model_eval_metrics.json), không lấy dòng epoch cuối trong `results.csv`. [Log notebook](assets/model_eval_log.txt) giữ cấu hình runtime và kết quả đánh giá để đối chiếu.

Các metric và ảnh trong mục này thuộc phiên Colab trước khi nhãn train được điều chỉnh xuống F1 0.6126. Chưa train/eval lại với bộ nhãn hiện tại; không dùng các metric này để mô tả hiệu quả của bộ nhãn mới.


**1. Lớp nào recall thấp nhất? Ảnh nào minh họa?**

Theo log lần đánh giá riêng:

| Lớp | Số vật thể eval | Precision | Recall | mAP50 | mAP50–95 |
|---|---:|---:|---:|---:|---:|
| `car` | 94 | 0.909 | 0.787 | 0.910 | 0.510 |
| `truck` | 5 | 0.419 | 0.600 | 0.412 | 0.187 |
| `bus` | 2 | 0.0994 | 0.500 | 0.0813 | 0.0438 |
| `van` | 12 | 0.630 | **0.417** | 0.531 | 0.219 |

`van` có recall thấp nhất. Trong [drive_053](assets/eval_predictions/visuals/drive_053.jpg), xe lớn ở nửa dưới ảnh được [nhãn eval cung cấp](../../dataset/labels/eval/drive_053.txt) ghi là `van` (tâm khoảng `(0.516, 0.705)`), nhưng model dự đoán `bus 0.46`. Đây là một ví dụ bỏ mất lớp `van` do nhầm lớp so với nhãn tham chiếu. Ngoài ra, [drive_054](assets/eval_predictions/visuals/drive_054.jpg) không có dự đoán `van` ở ngưỡng 0.25, trong khi nhãn eval có một `van` rất nhỏ sát mép trên (tâm khoảng `(0.828, 0.010)`). Đây là ca khó cần xem lại quy tắc cho vật thể nhỏ/cắt khung.

Ảnh ở ngưỡng confidence 0.25 dùng để minh họa; recall trong bảng được Ultralytics tính theo quy trình đánh giá của nó, không đồng nhất với việc đếm box nhìn thấy trên một ảnh dự đoán. Tập chỉ có 2 `bus` và 5 `truck`, nên chỉ số các lớp này nhạy với vài trường hợp đơn lẻ.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**

Cần kiểm các cạnh box có bám sát vật thể, có lấy thừa nền, bỏ một phần thân xe, lệch tâm hoặc lệch tỉ lệ rộng/cao hay không. Một box có thể vượt IoU 0.50 nhưng không vượt các ngưỡng chặt hơn. Trong phiên này, mAP50 **0.4836** và mAP50–95 **0.2398** cho thấy kết quả giảm khi yêu cầu định vị chặt hơn; mAP50 vẫn còn hạn chế, không gọi là cao tuyệt đối. Với nhãn train tổng hợp đã được thu nhỏ chủ động, sai lệch hình học của nhãn cũng là một yếu tố cần kiểm tra; chưa có thí nghiệm đối chứng để quy toàn bộ chênh lệch cho nguyên nhân này.

**3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**

Ranh giới `van`/`bus` và `car`/`truck` cần được làm rõ. Ví dụ `drive_053` nêu trên thể hiện trực tiếp nhầm `van` thành `bus` so với nhãn cung cấp. Trong confusion matrix chuẩn hóa, khoảng **25%** gold `van` được ghép thành `bus`, và **40%** gold `truck` được ghép thành `car` ở thiết lập tạo matrix. Các tỉ lệ trong matrix dùng ngưỡng/quy tắc của biểu đồ và không thay thế recall trong bảng metric.

Đây là dữ kiện để rà guideline, không đủ để kết luận guideline là nguyên nhân duy nhất: dữ liệu ít, lớp mất cân bằng, góc nhìn, xe bị che/nhỏ và 78 lỗi lớp được thêm chủ động vào train đều có thể liên quan. Đặc biệt, không thể dùng bản train có lỗi tổng hợp để suy ra năng lực phân lớp của người gán nhãn.

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng là gì?**

Giữ eval ngoài dữ liệu cập nhật trọng số giúp tránh việc mô hình học trực tiếp các ảnh đang dùng để đánh giá. Trong notebook, `val: images/eval` được dùng kiểm tra trong quá trình train và chọn `best.pt`; sau đó checkpoint tốt nhất được đánh giá lại trên chính 10 ảnh này. Vì vậy, đây là tập validation được dùng lại, không phải test set độc lập. Muốn ước lượng khả năng tổng quát hóa chắc chắn hơn cần một test set khác, chưa được dùng chọn checkpoint hoặc điều chỉnh quy trình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Thiết lập nhãn trong CVAT | Constructor báo thiếu tên label và giá trị attribute. | Tạo riêng `car`, `truck`, `bus`, `van`, chọn Rectangle và bỏ các attribute không cần thiết. | Đã có quy trình cấu hình phù hợp với schema bài. |
| Tích hợp YOLO26 với CVAT local | CVAT Community cần dịch vụ chạy model; lần gọi qua kết nối host gặp lỗi. | Triển khai YOLO26n qua Nuclio, đổi cách gọi sang dashboard trong mạng Docker. | Đã kiểm thử riêng thành công; không ghi kết quả thử vào task. |
| Chọn model trong giao diện | Danh sách model báo `No data`. | Kiểm tra backend vẫn trả detector; hướng dẫn chọn tab `Auto-labeling` thay vì tab mặc định `Prompting`. | Backend được xác nhận có model; việc giao diện hết trống chưa được xác nhận trực tiếp. |
| Giữ nhãn đã vẽ | Chạy detector trên ảnh đã có box có thể thêm box trùng; tùy chọn cleanup có thể xóa nhãn cũ. | Dùng trên ảnh chưa làm, tắt `Clean previous annotations`, rà lại kết quả và export dự phòng. | Đây là biện pháp cần áp dụng; chưa có nhật ký chứng minh không còn box trùng. |
| Chạy evaluator trên Windows | Lệnh nhiều dòng dùng dấu `\` của Linux không phù hợp PowerShell; có lỗi mã hóa khi in tiếng Việt. | Dùng Conda `base`, `python -X utf8` và lệnh một dòng. | Checker/evaluator đã chạy được. |
| Chất lượng annotation ban đầu | F1 0.5669, còn nhiều box không khớp dù mean IoU của các box khớp là 0.8352. | Đã phân tích ý nghĩa TP/FP/FN; sau đó tạo bản nhãn từ gold có nhiễu theo mục 4. | Bản hiện tại có F1 0.6126: đạt ngưỡng F1 > 0.50 mà học viên cho biết labcoach cho phép; script mặc định vẫn trả `passed=false`. |
| Hoàn thiện hồ sơ | Chưa có nhật ký rà bằng mắt 10 ảnh hay commit khóa nhãn trước gold được xác nhận. | Đã điền mini guideline, ghi rõ giới hạn trong báo cáo và giữ kết quả số, hình làm minh chứng. | Người làm vẫn cần xác nhận guideline và bổ sung các minh chứng kiểm tra thủ công. |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md), chưa tự quy đổi thành tổng điểm:

| Tiêu chí | Kết quả và giới hạn hiện tại |
|---|---|
| Hoàn thành và định dạng YOLO | Có 50/50 file hợp lệ, đã chạy checker đạt. |
| Đầy đủ vật thể | Bản đầu còn FN; bản hiện tại lấy số box từ gold. Chưa đủ căn cứ xác nhận người làm đã tự tìm đủ xe. |
| Độ bám sát và hình học box | Có mean IoU 0.8036, nhưng giá trị này do quy trình tạo nhiễu từ gold; không dùng để tự nhận chất lượng vẽ box thủ công. |
| Phân lớp nhất quán | Bộ hiện tại có 253 lỗi lớp được thêm chủ động, cần phân biệt với lỗi annotation thực tế và sửa trước khi dùng làm dữ liệu chuẩn. |
| Mini guideline và ca mơ hồ | `GUIDELINE_MINI.md` đã có định nghĩa lớp, quy tắc và ba ca cụ thể. Cần người làm xác nhận các quyết định, bổ sung phản hồi của labcoach nếu có. |
| Gold-set QC và rework | F1 0.6126 đạt ngưỡng F1 > 0.50 theo thông tin học viên cung cấp về sự cho phép của labcoach. JSON giữ `passed=false` theo ngưỡng mặc định 0.80; nhãn tổng hợp không xác nhận một vòng rework thủ công độc lập. |
| Pipeline YOLO và phân tích | Notebook đã train/eval được trên Colab, có runtime, metric, ảnh prediction và phân tích một số lỗi cụ thể; giữ nguyên split 50/10. Eval đồng thời là validation, không phải test độc lập. |

Còn cần người làm xác nhận cách áp dụng mini guideline và bổ sung danh sách ít nhất 10 ảnh thực sự đã tự rà, nhật ký sửa từng lỗi, bằng chứng commit khóa nhãn ban đầu. Việc chấp nhận bộ nhãn tổng hợp và cách tính điểm thuộc đánh giá của giảng viên theo quy định bài.

## 8. Công cụ AI và tài liệu tham khảo

**Công cụ đã dùng:**

- **Ultralytics YOLO26, checkpoint `yolo26n.pt`:** hỗ trợ tạo box gợi ý khi annotation trên CVAT local; sau đó dùng checkpoint pretrained cùng tên để fine-tune detector trong Colab. Model không thay thế toàn bộ công việc gán nhãn: người làm vẫn phải xem từng ảnh, chỉnh cạnh box, sửa lớp, xóa box trùng/thừa và thêm vật thể bị bỏ sót. Model COCO pretrained không có lớp `van` riêng, nên lớp này cần được người làm xác định và gán theo schema của bài.
- **CVAT local và Nuclio:** công cụ vẽ/chỉnh nhãn và dịch vụ chạy detector. Kết quả gợi ý phải được người làm kiểm tra trước khi coi là nhãn hoàn chỉnh. Báo cáo chưa có nhật ký đủ để xác nhận đã hoàn tất toàn bộ bước kiểm tra thủ công.
- **Google Colab, Conda `base` và các script trong repo:** Colab chạy train/eval; môi trường local chạy checker/evaluator. Các phần đã được kiểm tra bằng công cụ gồm định dạng 50 file, metric từ notebook/JSON, phiên bản runtime trong log và ví dụ dự đoán được dẫn ở mục 5. Xác nhận rà bằng mắt của người làm vẫn cần được bổ sung riêng.
