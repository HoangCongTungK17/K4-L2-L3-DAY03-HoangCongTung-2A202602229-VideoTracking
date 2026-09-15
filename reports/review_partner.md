# Peer review — Day 3

> Bài làm cá nhân — không có bạn cùng nhóm. File này ghi lại quá trình tự kiểm chéo.

| Trường | Giá trị |
| --- | --- |
| Author | Hoàng Công Tùng |
| Reviewer | Hoàng Công Tùng (tự kiểm) |
| Pair ID | solo |
| CVAT version | 2.x |
| Thời điểm review | 2026-09-15 |

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 73 | 73 | 5 | BBOX TREO | Hộp vẫn hiện sau khi xe rời khung. Rule: bấm Outside ngay frame đầu tiên xe biến mất | Bấm Outside (O) tại frame 73 để ngắt track | fixed |
| 2 | 149 | 149 | 4 | BBOX TREO | Hộp treo từ frame 149-151 sau khi xe ra khỏi mép phải. Rule: track kết thúc = Outside | Bấm Outside đúng frame xe rời khung | fixed |
| 3 | 54 | 54 | 4 | BBOX TRÔI | Interpolation bị lệch khi xe tăng tốc đột ngột. Rule: keyframe dày khi xe thay đổi tốc độ | Thêm keyframe tại frame 54 để kéo chỉnh mép hộp | fixed |
| 4 | 87 | 87 | 5 | BBOX TRÔI | Hộp bị lệch khỏi thân xe khi xe rẽ cua | Thêm keyframe và chỉnh lại mép hộp | fixed |
| 5 | 168 | 168 | 8 | BBOX TRÔI | Mép hộp không bám sát xe ở đoạn cuối clip | Thêm keyframe để hộp ôm khít hơn | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, ID 1-8, đều là xe bốn bánh |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0 sau rework |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Frame 80-95 ID 5&6 chồng nhau, giữ đúng ID sau khi tách |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING → fixed | Frame 73 ID 5, frame 149 ID 4 đã sửa |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Kiểm tra các frame xe sát rìa ảnh |
| Frame giữa hai keyframe không bị interpolation drift | FINDING → fixed | Frame 54, 87, 168 đã thêm keyframe |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py: 0 lỗi định dạng |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Xem bảng finding ở trên |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Tua toàn bộ clip, IDSW = 0, 8 track khớp với thực tế video |
| 2 — endpoint/scope | ĐÃ SỬA | Phát hiện và sửa BBOX TREO tại frame 73 ID 5 và frame 149 ID 4 |
| 3 — geometry/interpolation | ĐÃ SỬA | Thêm keyframe tại frame 54, 87, 168 để chống interpolation drift |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: **BBOX TREO tại frame 73 ID 5** — xe đã rời khung nhưng hộp vẫn còn hiện, vi phạm rule "bấm Outside ngay frame đầu tiên xe biến mất". Đã fix.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: Track 3 đứng im frame 1-15 — công cụ cảnh báo nhưng xem lại video xác nhận đây là xe đang chờ đèn đỏ thật sự, không phải lỗi.
3. Một rule cần Lab Coach làm rõ: Khi xe bị che **đúng bằng** 25 frame (bằng ngưỡng, không hơn không kém) thì giữ ID cũ hay tạo ID mới?
