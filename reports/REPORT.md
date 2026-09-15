# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Hoàng Công Tùng
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | [Điền số phút, ví dụ: 20] phút |
| Thời gian gán `clip_01` | [Điền số phút, ví dụ: 60] phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | [Điền ước chừng, ví dụ: 10] |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất sau cột/cây: Tôi vẫn giữ nguyên ID và chỉ vẽ phần hở ra.
2. Xe lọt ra ngoài mép màn hình: Tôi bấm phím O (Outside) ngay frame đầu tiên xe biến mất.
3. [Thêm một khó khăn của riêng bạn, ví dụ: xe đi xa quá khó nhìn...]

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Bắt được lỗi ID nhảy lung tung hoặc xe bị đổi ID.
- Lượt 2: Phát hiện hộp vẽ sớm trước khi xe xuất hiện, hoặc quên bấm Outside làm hộp treo.
- Lượt 3: Hộp bị lỏng lẻo ở đoạn giữa khi xe chạy nhanh.

Kiểm chéo với: [Tự kiểm tra]. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 0. Số lỗi bạn ấy tìm được trong bản của bạn: 0.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Luật quy định ngưỡng xuất hiện của xe ở phía xa (phải to cỡ bao nhiêu mới bắt đầu vẽ).

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | 19eea5aa2868e88889d03afdf4ee294448dbbcf681c267c50f49ce3f71dbec9a |
| Thời điểm khóa | 2026-09-15 05:09:43 UTC |
| Số row / frame / track trước khi mở reference | 584 rows / 190 frames / 8 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.814 | 0.806 | 0.824 | 0.876 | 0.961 | 0.921 | 0.866 | 28 | 17 | 0 |
| Sau rework | 0.981 | 0.975 | 0.989 | 0.979 | 0.997 | 0.995 | 0.978 | 3 | 0 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **CÓ**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TREO | 73-78 | 5 | Bấm Outside (O) để ngắt hộp do vẽ quá sớm |
| BBOX TREO | 149-151| 4 | Bấm Outside đúng frame xe rời khung |
| BBOX TRÔI | 54,87,168| 4,5,8| Thêm keyframe để kéo chỉnh lại mép hộp cho ôm khít xe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml & botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] |
| device | 0 (GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.981 | 0.975 | 0.989 | 0.979 | 0.997 | 0.995 | 0.978 | 3 | 0 | 0 |
| ByteTrack control vs gold | 0.699 | 0.649 | 0.754 | 0.843 | 0.874 | 0.750 | 0.822 | 84 | 58 | 3 |
| BoT-SORT + ReID vs gold | 0.792 | 0.736 | 0.855 | 0.891 | 0.909 | 0.811 | 0.880 | 83 | 26 | 1 |
| ReID vs bạn | 0.771 | 0.710 | 0.841 | 0.878 | 0.907 | 0.806 | 0.862 | 83 | 29 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Của tôi MOTA (0.995) và IDF1 (0.997) đều rất cao và xấp xỉ nhau vì tôi không bị lỗi IDSW (đổi ID) nào. Nếu MOTA cao mà IDF1 thấp, điều đó chứng tỏ hộp vẽ vẫn khít (bắt được vật thể) nhưng mã ID của xe bị nhảy liên tục. MOTA không phạt nặng lỗi ID vì nó chỉ đếm số lần nhảy ID (IDSW) mỗi lần 1 điểm phạt, trong khi IDF1 phạt dựa trên toàn bộ thời gian sống của track bị sai ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID vượt trội hơn ByteTrack: IDF1 tăng từ 0.874 lên 0.909; AssA tăng từ 0.754 lên 0.855; IDSW giảm từ 3 xuống 1. Điều này cho thấy khi kết hợp ngoại hình (appearance cue), tracker giữ được ID tốt hơn khi xe bị che lấp. (Lưu ý: Sự khác biệt này cũng có thể do implementation của hai tracker khác nhau chứ không hoàn toàn 100% do ReID).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 (ByteTrack) lên 0.736 (ReID). Đặc biệt, số lần bỏ sót xe (FN) giảm mạnh từ 58 xuống còn 26. FP (vẽ thừa) giữ nguyên mức 83-84. Lỗi còn lại (FP cao) chủ yếu là do Detector (YOLO) nhận diện nhầm các vật thể tĩnh thành xe và cứ thế gán hộp.

**4. Tìm một chỗ bạn đúng và ReID sai, và một chỗ ReID đúng mà bạn cần xem lại.**

- Tôi đúng, ReID sai: ReID có tới 83 FP (vẽ thừa hộp), nó nhận diện nhầm các vật thể tĩnh (hoặc xe không đúng chuẩn) trong khi tôi thì không vẽ (FP của tôi chỉ là 3).
- ReID đúng, tôi cần xem lại: Có một số frame xe đi vào vùng tối hoặc bị lấp một nửa, ReID nội suy quỹ đạo khít hơn mắt người.

**5. Nếu phải gán thêm 10 clip nữa, bạn sẽ sửa gì trong `GUIDELINE_MINI.md` để người gán tiếp theo không mắc lại lỗi bạn vừa mắc?**

Tôi sẽ ghi rõ quy định bắt buộc tua lại và bấm phím Outside (O) ngay lập tức khi xe ra khỏi khung hình để không bao giờ bị lỗi BBOX TREO lơ lửng nữa.
