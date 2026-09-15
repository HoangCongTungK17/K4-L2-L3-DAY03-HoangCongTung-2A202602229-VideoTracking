# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Hoàng Công Tùng
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

Bổ sung của nhóm (nếu có): Xe cứu thương và xe cảnh sát vẫn gán nhãn bình thường nếu là xe bốn bánh. Xe bị che hoàn toàn bởi xe khác (không nhìn thấy bất kỳ phần nào) thì bấm Outside, không giữ hộp.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Cùng một xe, không đổi danh tính chỉ vì bị khuất tạm thời |
| Xe bị che lâu hơn 25 frame | Kết thúc track cũ (Outside), tạo track mới khi xe xuất hiện lại | Không thể xác định chắc chắn đây vẫn là xe cũ hay xe khác |
| Xe rời khung hình rồi quay lại | **Track mới** — ID mới hoàn toàn | Xe đã ra ngoài phạm vi quan sát, không còn liên tục |
| Hai xe cắt nhau / chồng lên nhau | Theo dõi hướng và tốc độ từng xe trước khi chồng để giữ đúng ID sau khi tách ra | Nhầm ID khi hai xe chồng là lỗi IDSW nghiêm trọng nhất |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **tối thiểu 20×20 pixel** |
| Xe đang đỗ, không di chuyển | Vẫn vẽ và giữ nguyên ID trong toàn bộ thời gian dừng. **Không** bấm Outside khi xe dừng chờ đèn đỏ |
| Keyframe đặt dày ở đâu | Đặt keyframe dày hơn (mỗi 3-5 frame) khi xe tăng/giảm tốc đột ngột, rẽ cua, hoặc bị che khuất một phần |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, frame 1–15, ID 3
- Tình huống: Xe đứng hoàn toàn yên từ frame 1 đến 15, công cụ check cảnh báo "bbox gần như đứng im".
- Quyết định: Giữ nguyên track, không bấm Outside.
- Lý do: Xem lại video xác nhận xe đang dừng chờ đèn đỏ thật sự, không phải lỗi quên bấm Outside.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 80–95, ID 5 và ID 6
- Tình huống: Hai xe đi sát nhau và chồng lên nhau tại giao lộ, khó phân biệt mép hộp.
- Quyết định: Theo dõi hướng di chuyển trước khi xe chồng (ID 5 rẽ trái, ID 6 đi thẳng) để giữ đúng ID sau khi tách ra.
- Lý do: Dựa vào hướng và tốc độ, không dựa vào vị trí tức thời khi chồng.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 149–155, ID 4
- Tình huống: Xe đi ra sát mép phải màn hình, chỉ còn thấy một phần nhỏ thân xe.
- Quyết định: Vẽ bbox ôm phần nhìn thấy và chạm đúng rìa ảnh, bấm Outside ngay frame đầu tiên xe biến hẳn.
- Lý do: Theo luật bbox — không đoán phần ngoài ảnh, kết thúc track khi xe rời khung.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Bổ sung ngưỡng kích thước tối thiểu 20×20 pixel vào mục 3 (trước đó bỏ trống).
- Bổ sung rõ quy tắc xe dừng chờ đèn đỏ: vẫn vẽ hộp, không bấm Outside (trước đó chưa nói rõ, dẫn đến cảnh báo từ check_mot_labels.py).
