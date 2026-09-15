# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Hình thức / tên: **cá nhân** — Trần Đình Thọ
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung cá nhân: không gán xe máy, người hoặc vật thể trong ảnh quảng cáo dù detector/model có thể nhận nhầm là phương tiện.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật cá nhân | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (xấp xỉ 2 giây ở 12.5 fps) | Có đủ liên tục không gian/thời gian để nhận ra cùng xe; tránh tạo ID mới chỉ vì occlusion ngắn. |
| Xe bị che lâu hơn ngưỡng trên | kiểm tra lại toàn bộ đoạn; nếu không còn đủ bằng chứng nhận dạng thì kết thúc track cũ và chỉ tạo ID mới khi xe xuất hiện lại rõ ràng | Không đoán ID qua một đoạn mất dấu dài. |
| Xe rời khung hình rồi quay lại | dùng **track mới** | Lần xuất hiện lại không được nối ID cũ nếu xe đã rời khỏi ảnh. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo vị trí, hướng chuyển động và đặc điểm hình dáng trước/sau vùng chồng; không đổi ID chỉ vì hai bbox giao nhau | So sánh frame trước và sau occlusion để tránh tráo ID giữa hai xe. |

## 3. Luật bbox

| Tình huống | Luật cá nhân |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên có thể xác định chắc chắn là xe bốn bánh; không gán khi chỉ còn vài pixel hoặc dễ nhầm với vật thể khác |
| Xe đang đỗ, không di chuyển | vẫn giữ bbox và ID ở các frame xe còn hiện diện; không xóa track chỉ vì xe đứng yên |
| Keyframe đặt dày ở đâu | đặt dày khi xe đổi hướng, tăng/giảm tốc, bị che, chồng với xe khác, xuất hiện/rời khung hoặc bbox nội suy bắt đầu lệch; ưu tiên kiểm tra frame 9, 96 và 164 theo diagnostics |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 149-151 / ID 4`
- Tình huống: Xe tham chiếu đã rời khung nhưng annotation còn bbox treo thêm 3 frame.
- Quyết định: Kết thúc track bằng `outside` ở frame cuối xe còn nhìn thấy.
- Lý do: Bbox sau khi xe ra khỏi ảnh là false positive và làm sai biên track.

### Ca 2
- Clip / frame / ID: `clip_01 / 9 / ID 2`
- Tình huống: Bbox nội suy chỉ đạt IoU `0.509`, sát ngưỡng đánh giá và có dấu hiệu trôi.
- Quyết định: Kiểm tra lại frame quanh điểm này và thêm keyframe nếu bbox không còn ôm sát phần xe nhìn thấy.
- Lý do: Đoạn xe đổi vị trí giữa hai keyframe cần được hiệu chỉnh hình học, không chỉ giữ một bbox kéo dài.

### Ca 3
- Clip / frame / ID: `clip_01 / 164 / ID 8`
- Tình huống: Bbox annotation chỉ đạt IoU `0.525`; track ID 8 cũng chỉ phủ `23/33` frame của track tham chiếu.
- Quyết định: Xem lại cả keyframe và đoạn bắt đầu/kết thúc của track 8; không tự đổi ID nếu chưa có bằng chứng xe bị tráo.
- Lý do: Đây có thể là lỗi hình học hoặc thiếu đoạn, chưa đủ evidence để kết luận lỗi identity.

## 5. Sửa gì sau khi chấm với gold và sau khi tự kiểm

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Cần ghi rõ frame cuối cùng xe còn nhìn thấy trước khi bấm `outside`; không để bbox tiếp tục tồn tại sau khi xe rời khung.
- Cần kiểm tra độ phủ của từng track sau khi gán: các đoạn như track 8 (23/33 frame) và track 6 (43/56 frame) phải được tua lại trước khi khóa pre-gold.
