# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Đỗ Thành Long<br>
**MSSV:** 2A202602199<br>
**Hình thức:** cá nhân <br>
**Mã cặp:** `SOLO` 

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp:f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33
- Bốn mã ảnh: `drive_008`,`drive_022`,`drive_033`,`drive_038`
- Số vật thể thực tế: 85
- Mã SHA-256 của gói YOLO của bạn: 54e821914ea3ca16b246dd4adeabce4e7af8e2ac49da3139ec016e52c6690578
- Mã SHA-256 của gói CVAT gốc của bạn:22bf27851a4b6c5a4f89a45baeb63db7253c565b1f6564400755c5a5cce840ff
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành (Lab Coach) cấp (vì làm cá nhân)
- Mã SHA-256 của gói đối chiếu: 273826d2052246c91a508462eec141447ec61901c4749766d24271fc65769bc4
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: mã lần phát `day2-reference-4img-v1`, nhận lúc 2026-09-14 11:58

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu: Khi chưa đối chiếu những dữ liệu dùng để train hay val đều là dữ liệu được import từ chính người tạo không phải file đối chiếu nên kết quả hoàn toàn độc lập chưa sử dụng dữ liệu tham chiếu ở bước 5a.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_038`, hộp tại (421.93,136.20)-(522.40,213.34) | `truck` | thùng chở hàng phủ bạt, sàn hàng tách biệt khỏi khoang lái | "thùng/ben/sàn chở hàng rõ ràng → `truck`, không tính là `car`" |
| `drive_022`, hộp tại (577.30,258.15)-(626.91,330.86) | `van` (bộ tham chiếu gán `bus`) | cabin lái tách rời khoang hàng dạng hộp kín, không có hàng cửa sổ hành khách dọc thân | "thân hộp kín tách biệt khoang lái → không đạt tiêu chí `bus` (thân khách dài, nhiều hàng ghế)" |
| `drive_022`, hộp tại (297.20,314.30)-(360.86,361.43) | `van` | thân hộp nhỏ, kính chắn gió đứng, không có thân xe buýt dài hay khoang hàng tách biệt kiểu xe tải | "thân hộp nhỏ kín dùng chở người/hàng → `van`" |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Hộp `van` tại `drive_008` (497.81,0.00)-(511.95,24.49): **lớp** của vật thể là `van` (loại phương tiện), còn **thuộc tính** đi kèm là `visibility=unclear`, `boundary=inside` và `review_state=needs_review` — đây là các mô tả về điều kiện quan sát và trạng thái xử lý hộp, hoàn toàn không phải loại xe, và YOLO không lưu được các thuộc tính này.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| `drive_008`, hộp `van` (497.81,0.00)-(511.95,24.49), `boundary=inside` | hình học/thuộc tính | lọc mọi hộp `review_state=needs_review` trong `my_native_export_audit.json`, phóng ảnh 100% tại tọa độ hộp | `ytl=0.00` trùng đúng mép trên ảnh nên phải đổi `boundary` → `truncated`; giữ `review_state=needs_review` vì hộp quá nhỏ/mờ để chốt lớp `van` |
| `drive_033`, hộp `bus` (0.50,435.68)-(192.20,640.00), `boundary=inside` | hình học | so khớp `ybr=640.00` với chiều cao ảnh 640px khi rà lại các hộp chạm biên | đổi `boundary` → `truncated` vì mép dưới ảnh cắt qua thân xe |

- Số hộp `needs_review` trước và sau khi kiểm: tại thời điểm khóa bài (gói xuất đã nộp) còn **1/85** hộp `needs_review` chưa xử lý dứt điểm (`drive_008`, hộp `van` nêu trên).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: hộp `van` nói trên chỉ rộng 14×24 px và `visibility=unclear`, không đủ căn cứ khẳng định lớp dù đã phóng 100%; theo quy tắc "không đoán khi vật thể quá nhỏ/mờ", tôi giữ `needs_review` và sẽ hỏi Lab Coach thay vì tự đoán lớp.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.262383 0.573133 0.122641 0.071016` (ảnh `drive_022`, dòng 1)
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp `car` (class_id `0`); `xyxy` ≈ (128.7, 344.1, 207.2, 389.5) trên ảnh 640×640
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Kiểm tra định dạng chỉ xác nhận có đúng 5 trường số và các giá trị nằm trong [0,1] — nó không biết vật thể thật sự là gì. Người gán vẫn có thể chọn nhầm lớp (như ca `van`/`bus` ở mục 2), vẽ hộp lệch khỏi phần nhìn thấy, hoặc gán cả vật thể ngoài phạm vi (người, xe máy) mà dòng số vẫn hoàn toàn hợp lệ về mặt toán học.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: với ngưỡng `conf=0.25` sau khi huấn luyện dừng sớm ở epoch 1/8 (`mAP50≈0.006`), mô hình **không vẽ được bất kỳ hộp dự đoán nào** trên ảnh `drive_008`, dù ảnh này có tới 29 vật thể thật.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Không phải do quy tắc gán nhãn sai, mà do khối lượng dữ liệu huấn luyện quá nhỏ (3 ảnh, 56 hộp) và số epoch quá ít khiến mô hình chưa học được gì; cần nhiều ảnh và epoch hơn trước khi dùng kết quả để đánh giá chất lượng nhãn.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu hạ ngưỡng `conf` xuống rất thấp (ví dụ 0.01) mà mô hình vẫn không sinh được hộp nào đúng vị trí/lớp, vấn đề có thể nằm ở chính dữ liệu/nhãn (ảnh lỗi, nhãn sai hệ tọa độ) chứ không chỉ do thiếu dữ liệu huấn luyện.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Bốn ảnh (3 train/1 val) là tập cực nhỏ, không đại diện điều kiện thực tế (ánh sáng, góc quay, mật độ xe), và mô hình chỉ huấn luyện vài epoch với phần lớn lớp bị đóng băng (`freeze=10`) nên số liệu mAP chỉ có giá trị kiểm tra đường ống dữ liệu, không phải năng lực mô hình dùng sản xuất.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 46 (trên 85 hộp của tôi và 50 hộp tham chiếu)
- IoU trung bình và trung vị: trung bình 0.859; trung vị 0.876
- Mức đồng thuận lớp: 71.7% (33/46 hộp ghép trùng lớp)
- Số hộp phía bạn không ghép được: 39
- Số hộp phía đối chiếu không ghép được: 4
- Một điểm khác biệt cụ thể: ở `drive_022`, hộp xe buýt khớp nối lớn (118.80,351.50)-(387.30,572.10) tôi gán `bus`, ghép với hộp tham chiếu ở IoU rất cao 0.948 nhưng tham chiếu gán `van`; ngược lại một hộp lân cận (577.30,258.15)-(626.91,330.86) — xe có cabin lái tách rời khoang hàng dạng hộp kín — tôi gán `van` còn tham chiếu gán `bus`, IoU 0.866. Hai cặp hộp gần như trùng khớp hình học nhưng lớp bị đảo ngược ở cả hai phía, cho thấy ranh giới `bus/van/truck` cho xe thân hộp cỡ trung là điểm quy tắc còn mơ hồ, không phải lỗi vẽ hộp.
- Quy tắc hoặc hành động sửa phát sinh: đề xuất bổ sung quy tắc phụ — "xe có cabin lái tách rời khỏi thùng/hộp chở hàng phía sau (dù kín) → ưu tiên `truck`/`van` theo có/không cửa sổ hành khách dọc thân, không dùng kích thước tổng thể để suy ra `bus`".
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? IoU cao chỉ xác nhận hai người vẽ hộp ở cùng vị trí hình học; ca IoU=0.948 ở trên vẫn sai lệch lớp hoàn toàn, và cả hai bên có thể cùng mắc lỗi theo cùng một cách (cùng nhìn nhầm xe thân hộp ở xa/mờ) nên đồng thuận chỉ đo khả năng tái lập quy tắc, không đo tính đúng của nhãn.

## 7. Kiểm tra kho GitHub cá nhân

- [ ] Có phiếu quy tắc với ba tình huống mơ hồ.
- [ ] Có kết quả kiểm hai gói xuất.
- [ ] Có thông tin lần huấn luyện và ảnh dự đoán.
- [ ] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [ ] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [ ] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là cặp hộp ở mục 6 (`drive_022`, IoU 0.948 nhưng lớp đảo ngược `bus`↔`van` giữa hai bên) — cho thấy rõ ràng IoU cao không đồng nghĩa với nhãn đúng và chỉ ra một lỗ hổng quy tắc thật (ranh giới `bus/van/truck` cho xe thân hộp cỡ trung), chứ không phải suy đoán chung chung.

Câu hỏi còn lại cho Lab Coach: với xe có cabin lái tách rời khỏi khoang hàng dạng hộp kín cỡ trung (như hộp `drive_022` (577.30,258.15)-(626.91,330.86)), quy tắc hiện tại nên ưu tiên `van` hay `truck` khi không thấy rõ chi tiết sàn hàng/ben, và ngưỡng kích thước nào phân biệt ca này với `bus` nhỏ?
