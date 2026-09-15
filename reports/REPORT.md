# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Nguyễn Đình Nhật Trường / 2A202602321  
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (Computer Vision Annotation Tool), chế độ Rectangle Track |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 45 phút |
| Số track đã vẽ trong `clip_01` | 8 track |
| Số keyframe trung bình mỗi track | 5 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất một phần (occlusion) khi di chuyển**: Khi xe đi qua vật cản hoặc bị xe khác che một phần thân, tuân thủ nghiêm ngặt rule: giữ nguyên `track_id` cũ (vì thời gian che dưới 25 frame), chỉ vẽ bbox ôm sát phần nhìn thấy được (không phỏng đoán phần khuất) và đặt keyframe dày hơn quanh điểm bắt đầu/kết thúc che khuất.
2. **Xe rời khỏi khung hình ở rìa ảnh (boundary exit)**: Khi xe đi dần ra khỏi mép camera, bbox phải chạm đúng mép ảnh (clipping). Tại frame đầu tiên xe hoàn toàn khuất khỏi tầm nhìn, lập tức bấm phím `O` (Outside) để kết thúc track, tránh để bbox tiếp tục tồn tại ở vùng trống.
3. **Xe mới xuất hiện ở hậu cảnh xa (kích thước nhỏ và mờ)**: Khó phân biệt xe con với vật thể nền hoặc xe hai bánh; xử lý bằng cách chỉ tạo track từ frame đầu tiên xác định rõ ràng đó là xe bốn bánh (nhận diện được hình khối/bánh xe) với chiều rộng bbox tối thiểu ~15-20px để đảm bảo tính nhất quán.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (Nhìn ID/timeline): Kiểm tra tính nhất quán và liên tục của `track_id` trên thanh timeline; phát hiện và sửa các ca vô tình tạo track mới khi xe đổi hướng, đảm bảo 1 xe duy nhất 1 ID suốt toàn bộ clip, không xảy ra ID switch.
- Lượt 2 (Frame đầu/cuối): Rà soát chính xác frame bắt đầu xuất hiện (entry) và frame rời khung (exit). Xác nhận mọi track đều được bấm Outside (`O`) đúng thời điểm xe khuất hẳn, không để sót bbox treo lơ lửng.
- Lượt 3 (Frame giữa): Tua chậm tốc độ 0.5x kiểm tra chuyển động nội suy (interpolation) giữa các keyframe. Phát hiện hiện tượng trôi bbox (interpolation drift) khi xe chuyển làn hoặc thay đổi vận tốc, bổ sung thêm keyframe trung gian để bbox luôn ôm khít thân xe.

Kiểm chéo với: Bạn cùng nhóm (Pair Partner). Chi tiết ở `reports/review_partner.md`.  
Số lỗi bạn tìm được trong bản của bạn ấy: 3 lỗi (trôi bbox khi xe rẽ và 1 track quên Outside khi ra rìa).  
Số lỗi bạn ấy tìm được trong bản của bạn: 2 lỗi (bbox ở frame xe mới vào còn hơi rộng và thiếu 1 keyframe khi xe tăng tốc).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- Ca xe đỗ tĩnh ở ven đường (Track 2 frame 1–15): Bạn reviewer băn khoăn liệu bbox đứng yên có phải là lỗi quên bấm Outside hay không. Hai bên đối chiếu và thống nhất theo quy định lab: Xe bốn bánh đang đỗ/dừng chờ giao thông vẫn là `vehicle`, phải gán và giữ nguyên track suốt thời gian nó xuất hiện trong khung hình. Đã bổ sung luật rõ ràng vào `GUIDELINE_MINI.md`: *"Mọi xe 4 bánh kể cả đang đỗ tĩnh ven đường đều phải gán track liên tục"*.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `b69c3ff49e69d338fa0bb5fe6cbebf0f34f6b411f36195459158212223c264b6` |
| Thời điểm khóa | `2026-09-15T09:23:34.548004+00:00 (UTC)` (16:23:34 GMT+7) |
| Số row / frame / track trước khi mở reference | 553 rows / 190 frames / 8 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.739 | 0.701 | 0.792 | 0.818 | 0.947 | 0.895 | 0.781 | 20 | 40 | 0 |
| Sau rework | 0.739 | 0.701 | 0.792 | 0.818 | 0.947 | 0.895 | 0.781 | 20 | 40 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** (ĐẠT cả 3 tiêu chí: IDF1 = 0.947, MOTA = 0.895, MOTP = 0.781; 0 ID switch)

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Thiếu đoạn | 46–60 | 5 | Kéo dài track 5 từ frame 45 đến frame 60 cho đến khi xe hoàn toàn khuất góc quay |
| Bbox trôi | 82–85 | 5 | Thêm 2 keyframe ở frame 83 và 85 khi xe chuyển làn để nâng IoU > 0.70 |
| Bbox trôi | 109 | 6 | Thêm keyframe tại frame 109, chỉnh lại góc quay bbox ôm khít đầu xe đang rẽ |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.12.7 / ultralytics 8.4.145 / torch 2.14.0+cpu / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml (Control) & configs/trackers/botsort-reid.yaml (Treatment) |
| conf / IoU / imgsz / classes | conf 0.25 / IoU 0.70 / imgsz 960 / classes [2, 5, 7] (car, bus, truck) |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.739 | 0.701 | 0.792 | 0.818 | 0.947 | 0.895 | 0.781 | 20 | 40 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.660 | 0.597 | 0.736 | 0.793 | 0.897 | 0.779 | 0.744 | 103 | 18 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả của em: IDF1 (0.947) cao hơn MOTA (0.895).  
Nếu xảy ra tình huống MOTA cao mà IDF1 thấp, điều đó phản ánh rằng hệ thống (hoặc người gán) phát hiện vị trí vật thể trên từng frame rất tốt (FN và FP thấp), nhưng khả năng duy trì danh tính (identity association) xuyên suốt thời gian rất kém (xảy ra nhiều lần ID switch hoặc phân mảnh track).  
MOTA không phạt nặng lỗi ID vì công thức của MOTA là $1 - \frac{FN + FP + IDSW}{GT}$. Tại mỗi lần đổi ID (ID switch), MOTA chỉ cộng thêm 1 đơn vị phạt vào $IDSW$ tại đúng 1 frame xảy ra sự cố; ở tất cả các frame tiếp theo, dù vật thể đang mang sai ID, MOTA vẫn tính đó là True Positive vì bbox vẫn khớp với vị trí vật thể. Ngược lại, IDF1 tính toán mức độ bảo toàn danh tính trên toàn bộ quỹ đạo (trajectory matching); nếu 1 xe bị đổi ID ở giữa chừng, toàn bộ nửa sau của quỹ đạo sẽ bị phân loại là IDFP và IDFN, khiến IDF1 sụt giảm rất nặng nề.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- Về chỉ số định lượng: BoT-SORT + ReID vượt trội hơn ByteTrack control ở hầu hết các chỉ số tracking cốt lõi:
  - IDF1 tăng từ 0.875 lên 0.900 (+0.025).
  - AssA tăng từ 0.776 lên 0.820 (+0.044).
  - HOTA tăng từ 0.709 lên 0.763 (+0.054).
  - Cả hai đều có 2 lần ID switch, nhưng BoT-SORT duy trì được độ dài track toàn vẹn hơn rất nhiều.
- Phân tích frame sequence cụ thể:
  - Ở ByteTrack control: Tại frame 59, track gold 4 (dài 95 frame) bị cắt xẻ thành 2 ID riêng biệt [14, 15] khi xe bị che khuất một phần; tương tự track gold 7 (dài 85 frame) cũng bị đứt đoạn thành ID [52, 71]. ByteTrack chỉ sử dụng chuyển động Kalman và IoU, nên khi xe bị che hoặc chuyển động phi tuyến làm IoU dự đoán bị tụt, nó coi xe xuất hiện lại là một track mới.
  - Ở BoT-SORT + ReID treatment: Nhờ có thêm vector đặc trưng ngoại hình (appearance embedding) hỗ trợ song song với spatial proximity, tracker đã duy trì trọn vẹn track gold 4 từ đầu đến cuối mà không bị tách đôi.
- **Lưu ý phương pháp luận**: So sánh này là so sánh cấp hệ thống (system-level comparison), **không cô lập được hiệu ứng nhân quả (causal effect) thuần túy của ReID**. Lý do là ByteTrack và BoT-SORT có sự khác biệt lớn về kiến trúc thuật toán liên kết: BoT-SORT sử dụng ma trận khoảng cách kết hợp giữa IoU và cosine similarity của visual appearance, có ngưỡng matching threshold và cơ chế track buffer khác biệt so với cơ chế matching 2 vòng của ByteTrack.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- DetA tăng từ 0.649 (ByteTrack) lên 0.711 (BoT-SORT + ReID).
- FN giảm mạnh hơn một nửa, từ 54 xuống còn 26, cho thấy BoT-SORT thu hồi được nhiều bbox ở các góc khó/rìa ảnh mà ByteTrack làm mất.
- FP tăng nhẹ từ 88 lên 91.
- Đánh giá lỗi còn lại: Lỗi còn lại hiện tại **chủ yếu là lỗi của detector, không phải của association**. Cả hai model chỉ có 2 lần ID switch (association rất tốt), nhưng đều có lượng FP rất cao (88 và 91 FP so với chỉ 20 FP của người gán). Khi soi chi tiết chẩn đoán lỗi, các FP này chủ yếu là do YOLO phát hiện nhầm các vật thể tĩnh ở lề đường/bảng hiệu (ví dụ ID 7/ID 10 kéo dài hơn 40 frame tĩnh) thành xe bốn bánh.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- Frame 16–116 (ID 7 của model): Model BoT-SORT + ReID liên tục duy trì track ID 7 suốt 43 frame tại vị trí lề đường bên trái. Trên thực tế, đây là một biển hiệu/vật thể cố định bị detector YOLO nhận diện nhầm là xe với confidence ~0.3-0.4, và ReID embedding duy trì nhận diện tĩnh này qua thời gian.
- Người gán nhãn quan sát toàn cảnh video động, xác định đây là vật thể tĩnh ngoài schema nên không gán -> Người đúng (tránh được 43 FP sai lệch), model sai do hạn chế nhận diện ngữ cảnh 2D của detector.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- Frame 106–121 (ID 27 của model): Model phát hiện một track xe (ID 27) xuất hiện ở hậu cảnh xa phía trên bên phải kéo dài 16 frame. Khi soi lại video, ở khu vực này có một chiếc xe thật đang di chuyển vào khung hình nhưng kích thước nhỏ (~18px) và hơi mờ.
- Trong bản nhãn tay ban đầu, người gán nhãn đặt ngưỡng kích thước hơi khắt khe nên chỉ bắt đầu track khi xe tới frame 115 (gây ra hiện tượng thiếu frame / FN). Việc ReID phát hiện được xe từ frame 106 đã giúp người gán nhãn nhận ra cần hạ ngưỡng nhận biết phương tiện ở cự ly xa khi gán nhãn.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Sửa đổi trong `GUIDELINE_MINI.md`:
  1. Quy định rõ ràng bằng số liệu cho kích thước xe tối thiểu ở hậu cảnh xa: *"Bắt đầu gán track khi chiều rộng xe đạt tối thiểu 18 pixel và nhận diện được bánh xe hoặc đèn xe"*.
  2. Bổ sung luật chi tiết cho xe dừng/đỗ: *"Gán track cho mọi xe 4 bánh đỗ ven đường trong phạm vi lòng đường; không gán xe đỗ sâu trong gara/nhà dân bị mép ảnh cắt xẻ quá 50%"*.
  3. Chuẩn hoá thời điểm Outside: *"Bấm phím O tại frame đầu tiên diện tích xe nhìn thấy được giảm dưới 10% hoặc tâm xe đã đi ra ngoài viền ảnh"*.
- Đổi mới trong quy trình làm việc:
  1. Kiểm tra cuốn chiếu (Rolling Inspection): Sau khi hoàn thành xong 1 track, tua lại ngay lập tức ở tốc độ 0.5x để rà soát interpolation drift trước khi sang xe khác, không để dồn việc kiểm tra đến cuối.
  2. Tận dụng sớm công cụ validator: Chạy `check_mot_labels.py` và `visualize_tracks.py` định kỳ sau mỗi 2–3 track để phát hiện sớm các lỗi bbox đứng yên hoặc trôi dạt.

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
