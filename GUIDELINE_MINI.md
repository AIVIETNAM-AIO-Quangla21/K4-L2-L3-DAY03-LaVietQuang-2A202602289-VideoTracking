# Mini annotation guideline — Ngày 3 (tracking)

> File này được điền trong lúc gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> nghĩ “cái này tính sao nhỉ?” thì đó là một câu phải ghi vào đây. Đây là tài liệu để
> người gán nhãn tiếp theo làm giống bạn. Nếu hai người trong nhóm gán khác nhau, gần
> như luôn là vì file này chưa nói rõ — không phải vì ai kém hơn ai.

Nhóm / tên: Nhóm học viên Video Tracking
Clip: clip_01, clip_02

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **vehicle** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: không gán xe máy, mô tô hay các vật thể trong gương, biển quảng cáo hoặc phản xạ. Nếu một vật thể có dạng ô tô nhưng bị che hoàn toàn hoặc quá mờ để xác định, không đặt track mới nếu không đủ bằng chứng.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới 25 frame (tương đương khoảng 2 giây ở 12.5 fps) | Duy trì tính liên tục của track, tránh ID switch giả tạo khi xe vẫn cùng một đối tượng |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới nếu mất dấu quá 25 frame hoặc không còn đủ bằng chứng nhận dạng | Một track kéo dài quá lâu mà không có bounding box chứng minh là cùng xe sẽ phá vỡ identity |
| Xe rời khung hình rồi quay lại | Mặc định là **track mới** | Khi xe đã ra khỏi vùng quan sát, ta không có đủ căn cứ để coi là cùng một đối tượng |
| Hai xe cắt nhau / chồng lên nhau | Giữ hai bbox riêng, không “gộp” vào nhau; ưu tiên bám theo đường đi và kích thước xe | Cắt chéo là trường hợp dễ suy nhầm nhất, và nhãn phải phản ánh vật thể thực mà không “đoán” theo ý chủ quan |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm phần **nhìn thấy được** của xe đó, không kéo dài ra ngoài để lấp đầy |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng tối thiểu là khi có đủ hình dạng và kích thước để phân biệt với background |
| Xe đang đỗ, không di chuyển | Vẫn đánh dấu là `vehicle` và giữ track liên tục trong thời gian xe còn xuất hiện |
| Keyframe đặt dày ở đâu | Đặt keyframe dày ở các vị trí xe đổi hướng, bị che, đổi tốc độ, hoặc khi box cần chỉnh khít để không drift |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung. Dưới đây là dạng ca mơ hồ theo quy tắc lab và cách quyết định chuẩn hóa.

### Ca 1
- Clip / frame / ID: clip_01 / frame gần xuất hiện xe đầu tiên / ID đang được gán ở thời điểm xe vừa hiện
- Tình huống: Xe mới xuất hiện nhưng còn nhỏ, mờ và có thể chưa rõ là xe bốn bánh hay vật thể khác
- Quyết định: Bắt đầu track từ frame đầu tiên xác định được rõ đây là `vehicle`
- Lý do: không bắt sớm quá mức khi chưa chắc chắn; không chờ quá muộn vì lúc đó xe đã lớn hơn khiến bbox nền và ID bị lệch

### Ca 2
- Clip / frame / ID: clip_01 / khi xe bị che cắt ngang hoặc bị chắn bởi xe khác / ID cũ
- Tình huống: Xe bị che tạm thời, sau đó lộ lại trong cùng luồng di chuyển
- Quyết định: Giữ nguyên ID nếu mất dấu dưới 25 frame và đường đi khớp với track cũ
- Lý do: đây là trường hợp identity phải gắn theo “hành vi chuyển động” và “kích thước”, không vứt ID chỉ vì bị che ngắn |

### Ca 3
- Clip / frame / ID: clip_01 hoặc clip_02 / khi xe rời khung rồi quay lại sau một khoảng dài / ID mới
- Tình huống: xe đi ra khỏi vùng quan sát và xuất hiện lại ở vị trí khác
- Quyết định: Mở track mới nếu xe đã rời khỏi khung và không thể xác nhận là cùng đối tượng
- Lý do: nếu không có bằng chứng từ motion, bề mặt hoặc vị trí tương hợp, việc giữ ID cũ sẽ tạo ra ID switch nhầm trong evaluation

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ thì phải viết lại cho rõ. Sau khi đánh giá và kiểm chéo, nhóm đã thống nhất các điểm quan trọng sau:

- Xe bị che ngắn dưới 25 frame thì giữ nguyên ID; nếu che quá lâu hoặc mất dấu nghiêm trọng thì mở track mới.
- Bbox phải sát phần nhìn thấy được, không đoán phần bị che hoặc vùng ngoài khung.
- Xuất hiện lần đầu, nếu vật thể mờ nhưng đủ dấu hiệu là xe bốn bánh thì bắt đầu track ngay từ frame đầu tiên xác định được.
- Khi hai xe chồng tầm nhìn, ưu tiên “đưa ra quyết định dựa trên sự liên tục chuyển động” thay vì tưởng tượng cùng một xe.

---

## 6. Checklist thao tác để tránh lỗi thường gặp

- Export đúng định dạng MOT 1.1; không export YOLO hoặc file thiếu track_id.
- Dùng Track mode thay vì Shape mode để giữ dữ liệu thời gian.
- Không sửa trực tiếp file MOT để “đổi điểm”; sửa trong CVAT rồi export lại.
- Tua ngược và kiểm tra keyframe giữa các frame để tránh drift và bbox lệch.
- Khi xe rời khỏi vùng quan sát, bấm outside để không để bbox tiếp tục tồn tại ở chỗ trống.
- Sau khi lưu, reload task và kiểm tra số item / bbox còn tồn tại trước khi coi là an toàn.

## 7. Quy định áp dụng cho cả clip

- Quy tắc trên phải dùng thống nhất trên `clip_01` và `clip_02`.
- Nếu một trường hợp mơ hồ, ghi rõ frame, ID và lý do quyết định vào phần “ca mơ hồ”.
- Mục tiêu là tránh ID switch và bbox drift, vì đây là lỗi có thể gây hạ mạnh `IDF1` dù MOTA vẫn nhìn ổn trên bề mặt.
