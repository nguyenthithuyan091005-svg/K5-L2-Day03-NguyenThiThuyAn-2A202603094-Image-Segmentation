# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202603094
- Ngày / CVAT local: 12/11/2026; CVAT localhost
- Công cụ đã dùng: Polygon và công cụ chỉnh mask trong CVAT; nhãn gợi ý Mask2Former Cityscapes/COCO; scorer của bài để tự đánh giá ba tier sau khi nhận reference.

## 1. Bài đã nộp

Tôi đã QC, Save và export đủ các task. Các ZIP dưới đây có đủ ảnh và đã qua kiểm tra cấu trúc. Cột điểm là điểm tối đa của task, không phải điểm tự chấm.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip| 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip| 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip| 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip| 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip| 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip| 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip| 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip| 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Trong ảnh 000000181542.jpg của task medium_instance, tôi chọn người phụ nữ mặc áo dài sáng màu đang đi bộ trên vạch sang đường, gần giữa ảnh và hơi lệch sang trái. Tôi dùng công cụ Polygon để tạo một object riêng, hiển thị là ID 1 trong danh sách Objects.
- Class và quy tắc tôi dùng để chọn biên: Tôi gán class person. Tôi chọn đường viền theo phần cơ thể và trang phục nhìn thấy, gồm tóc, đầu, vai, cánh tay, bàn tay, áo dài, quần và bàn chân. Tôi giữ toàn bộ người này trong cùng một instance; không tách quần áo hay các bộ phận cơ thể thành object riêng. Tại những vị trí sát người đi xe máy và xe máy phía sau, tôi bám theo biên của người phụ nữ để tránh gộp các vật khác vào mask. Mặt đường, vạch sang đường và bóng đổ không thuộc mask person.
- Tôi tự vẽ object đầu tiên bằng Polygon, không dùng gợi ý tự động cho object này. Sau đó tôi dùng nhãn gợi ý Mask2Former cho các ảnh và kiểm tra lại trong CVAT.
- Quyết định gán nhãn: Tôi chọn một người cụ thể làm một instance, dù trong ảnh còn nhiều người khác. Những người còn lại sẽ được gán thành các object person riêng.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `cp6_coverage`, ảnh `7daa6479-67988f3f.jpg`, vùng vỉa hè cạnh mặt đường.
- Lỗi thuộc loại: Sai lớp và sai ranh giới giữa `road` và `sidewalk`.
- Bằng chứng tôi nhìn thấy: Một phần vỉa hè có màu khá giống mặt đường nên bị gán thành `road`, nhưng bó vỉa và vị trí của vùng này cho thấy đây là phần dành cho người đi bộ.
- Quy tắc và hành động sửa: Tôi dựa vào bó vỉa và chức năng sử dụng để xác định ranh, rồi đổi vùng đó từ `road` sang `sidewalk`.
- Sau sửa: Tôi đã Save trong CVAT và export lại ZIP của task.

Chạy scorer với tiers_gt.zip: Tổng ba tier đạt 71,9/82, chưa phải điểm cuối trên 100. Đơn giản vậy thôi rồi chỉnh sửa lại file và push nó lên repo với 2 file đã tải từ cvat

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp6_coverage`, ảnh `7daa6479-67988f3f.jpg`, vùng sát mép đường| Gán `road` vì màu giống mặt đường, hoặc gán `sidewalk`| Bó vỉa và chức năng của vùng quan trọng hơn sự giống nhau về màu | Tôi chọn `sidewalk` và sửa lại vùng đã gán nhầm `road`. |
| `easy_semantic`, ảnh `7ee6d192-89e2408b.jpg`, dải phân cách bên đường cao tốc | Giữ trong mask `road` vì nằm sát đường, hoặc loại khỏi mask `road` | Dải phân cách là cấu trúc riêng; mask `road` chỉ phủ phần mặt đường nhìn thấy | Tôi xóa phần dải phân cách bị gán nhầm vào `road`.  |
| `cp1_holes`, ảnh `000000144300.jpg`, kính chắn gió và chi tiết bên trong xe mô tô | Giữ trong cùng mask `motorcycle`, hoặc khoét thành lỗ nền| Quy tắc trạm yêu cầu không khoét kính và chi tiết thuộc vật một cách tùy tiện| Tôi dựa vào phần vật nhìn thấy để giữ các chi tiết thuộc xe trong cùng instance; nếu một khe thực sự cho thấy nền phía sau, tôi sẽ hỏi coach trước khi quyết định biên.|
