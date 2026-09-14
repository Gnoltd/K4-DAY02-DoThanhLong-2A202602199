# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Đỗ Thành Long<br>
**MSSV:** 2A202602199<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_022`, hộp tại (577.30,258.15)-(626.91,330.86)
- Dấu hiệu nhìn thấy: cabin lái tách rời khỏi khoang hàng dạng hộp kín phía sau, không có hàng cửa sổ hành khách liên tục dọc thân như xe buýt
- Quy tắc áp dụng: "thân xe khách dài, nhiều cửa sổ/hàng ghế → `bus`; thân hộp kín tách biệt khoang lái → không đạt tiêu chí `bus`"
- Quyết định: gán `van` (bộ tham chiếu của Lab Coach lại gán `bus` cho cùng hộp)
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? So sánh với xe buýt rõ ràng khác trong cùng ảnh, nếu vẫn không chắc thì đặt `review_state=needs_review` và hỏi Lab Coach thay vì tự quyết theo cảm tính.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038`, hộp tại (421.93,136.20)-(522.40,213.34)
- Dấu hiệu nhìn thấy: thùng chở hàng phủ bạt lộ rõ phía sau cabin, sàn hàng tách biệt khỏi khoang lái
- Quy tắc áp dụng: "thùng/ben/sàn chở hàng hoặc thiết bị công vụ rõ ràng → `truck`, không tính là `car`"
- Quyết định: gán `truck`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Kiểm tra thêm chi tiết khung gầm/bánh xe để loại trừ khả năng là xe con cải tạo; nếu không chắc thì đánh dấu `needs_review`.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, hộp `van` tại (497.81,0.00)-(511.95,24.49)
- Dấu hiệu nhìn thấy khi phóng 100%: hộp rất nhỏ (14×24 px), sát mép trên ảnh, chi tiết thân xe gần như không phân biệt được do khoảng cách xa và độ phân giải thấp
- Giá trị `visibility`: `unclear`
- Giá trị `boundary`: đang ghi `inside` nhưng thực tế `ytl=0.00` trùng mép trên ảnh nên cần sửa thành `truncated`
- Trạng thái `review_state`: `needs_review`
- Lý do: vật thể quá nhỏ/mờ để khẳng định chắc chắn là `van`, đồng thời tọa độ hộp cho thấy vật thể bị mép ảnh cắt nên `boundary` đang ghi sai; đây là hộp duy nhất trong 85 hộp còn ở trạng thái `needs_review` khi khóa bài.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [ ] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 85 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
