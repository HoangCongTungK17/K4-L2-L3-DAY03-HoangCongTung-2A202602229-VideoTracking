# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Hoàng Công Tùng
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 20 phút |
| Thời gian gán `clip_01` | 60 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 10 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất sau cột/cây: Tôi vẫn giữ nguyên ID và chỉ vẽ phần hở ra.
2. Xe lọt ra ngoài mép màn hình: Tôi bấm phím O (Outside) ngay frame đầu tiên xe biến mất.
3. Nhiều xe đi gần nhau và trùng nhau tại giao lộ: Tôi theo dõi hướng di chuyển và tốc độ của từng xe để không bị gán nhầm ID khi chúng chồng chéo lên nhau.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (nhìn ID): Phát hiện track 3 bị cảnh báo bbox đứng im từ frame 1-15 — xác nhận xe thật đang chờ đèn đỏ, không phải lỗi quên Outside.
- Lượt 2 (frame đầu/cuối): Phát hiện một số track kết thúc chưa khớp với lúc xe rời khỏi hình, bổ sung keyframe Outside cho đúng.
- Lượt 3 (frame giữa): Phát hiện một số hộp bị lệch khi xe tăng tốc đột ngột, thêm keyframe để hộp bám sát xe hơn.

Kiểm chéo với: bản thân tự kiểm tra (không có bạn cùng nhóm). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: N/A. Số lỗi bạn ấy tìm được trong bản của bạn: N/A.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Do tự kiểm nên không có ca bất đồng. Tuy nhiên, `GUIDELINE_MINI.md` còn thiếu quy định về ngưỡng kích thước tối thiểu của xe ở xa (bao nhiêu pixel thì mới bắt đầu vẽ hộp) và cách xử lý xe dừng chờ đèn đỏ nhiều frame liên tiếp.

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

BoT-SORT + ReID vượt trội hơn ByteTrack: IDF1 tăng từ 0.874 lên 0.909; AssA tăng từ 0.754 lên 0.855; IDSW giảm từ 3 xuống 1. Ví dụ frame sequence khoảng frame 80-100 khi có xe bị khuất sau xe tải lớn: ByteTrack mất dấu và gán ID mới khi xe xuất hiện lại (gây IDSW), trong khi BoT-SORT + ReID nhận ra ngoại hình xe và giữ nguyên ID cũ. Lưu ý: sự khác biệt này không cô lập hoàn toàn causal effect của ReID vì hai tracker có implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 (ByteTrack) lên 0.736 (ReID). Số lần bỏ sót xe (FN) giảm mạnh từ 58 xuống còn 26. FP (vẽ thừa) giữ nguyên mức 83-84. Lỗi còn lại chủ yếu là do **Detector**: YOLO nhận diện nhầm các vật thể không phải xe (biển báo, bóng đổ) và cứ thế gán hộp — đây là lỗi detector, không phải lỗi association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Khoảng frame 120-140, ID xe máy nhỏ ở góc trái: ReID tạo ra 83 FP bằng cách gán hộp vào bóng đổ và mặt đường có vệt sơn, trong khi tôi không vẽ vì đó không phải xe thật. FP của tôi chỉ là 3 chứng tỏ tôi phân biệt tốt hơn model về việc vật thể nào thực sự là phương tiện.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Khoảng frame 60-70, xe ô tô ID 6 đi vào vùng ngược sáng (backlight): ReID vẫn dự đoán hộp bám sát thân xe dựa trên quỹ đạo nội suy, còn hộp của tôi bị lệch ~15px do mắt khó nhìn rõ mép xe. FN=0 của tôi sau rework xác nhận tôi đã sửa lại đúng, nhưng đây là tình huống cần cẩn thận hơn khi gán nhãn trong vùng sáng ngược.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` hai quy tắc còn thiếu: (1) Ngưỡng kích thước tối thiểu — xe phải chiếm ít nhất 20×20 pixel thì mới bắt đầu vẽ hộp, tránh vẽ nhầm xe quá nhỏ ở xa; (2) Quy tắc xe dừng chờ đèn — bbox vẫn phải được vẽ và giữ nguyên ID trong suốt thời gian xe đứng yên, không được bấm Outside. Về quy trình cá nhân: tôi sẽ tua lại clip 3 lượt ngay từ đầu thay vì chỉ tua khi gần xong, để phát hiện lỗi sớm hơn và tốn ít thời gian sửa hơn.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
