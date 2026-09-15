# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Nguyễn Đình Nhật Trường |
| Reviewer | Bạn cùng nhóm (Pair Partner) |
| Pair ID | D03-PAIR-01 |
| CVAT version | 2.14.0 (CVAT.ai) |
| Thời điểm review | 2026-09-15 15:45 (UTC+7) |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 84 | 85 | 5 | Bbox trôi | Giữa 2 keyframe xe hơi giảm tốc và chuyển làn, bbox bị lùi sau đuôi xe ~10px | Thêm keyframe tại frame 85, kéo bbox ôm khít thân xe | fixed |
| 2 | 108 | 109 | 6 | Bbox lệch | Xe rẽ góc, bbox interpolation không khít góc trước bên phụ | Thêm keyframe tại frame 109, chỉnh lại góc quay bbox | fixed |
| 3 | 14 | 15 | 2 | Bbox đứng yên | Xe dừng chờ ở làn rẽ, bbox không thay đổi vị trí trong 15 frame đầu | Giữ nguyên vì xe thật dừng chờ đèn/giao thông, không phải quên bấm Outside | not-a-defect |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đã có 8 track xe bốn bánh hợp lệ |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Timeline ID liên tục, không trùng lặp |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 4 và 5 giữ đúng ID qua đoạn che khuất |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Bấm Outside kịp thời tại rìa khung hình |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox ôm sát phần nhìn thấy theo rule |
| Frame giữa hai keyframe không bị interpolation drift | FIXED | Đã chèn thêm keyframe tại frame 85 và 109 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py báo 0 lỗi định dạng |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đầy đủ 3 finding kèm closure |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Toàn bộ 8 track duy trì 1 ID duy nhất |
| 2 — endpoint/scope | PASS | Kiểm tra frame vào/ra của cả 8 track |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã sửa hiện tượng trôi bbox ở frame 85 và 109 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Trôi bbox khi xe đổi hướng/vận tốc giữa 2 keyframe cách nhau xa (Áp dụng rule: tua chậm frame giữa và đặt keyframe dày hơn quanh điểm chuyển động phi tuyến tính).
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Finding 3 (Track 2 frame 1-15 đứng yên do xe dừng chờ đèn đỏ thực tế, không phải lỗi quên Outside).
3. Một rule cần Lab Coach làm rõ (nếu có): Quy định cụ thể về khoảng cách/kích thước tối thiểu để bắt đầu gán xe ở hậu cảnh xa.
