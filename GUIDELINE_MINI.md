# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Nguyễn Đình Nhật Trường / 2A202602321  
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

Bổ sung của nhóm:
- Xe đang đỗ/dừng ven đường hoặc chờ đèn đỏ: **GÁN**.
- Xe ba bánh chở hàng, xe máy kéo nhỏ: **KHÔNG GÁN**.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Duy trì tính liên tục của identity đối với các che khuất ngắn hạn |
| Xe bị che lâu hơn ngưỡng trên | Mở **track mới** với ID mới | Tránh gán sai identity khi khoảng gián đoạn quá dài không thể chắc chắn quỹ đạo |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Đã ra khỏi khung hình thì coi như kết thúc một quỹ đạo quan sát |
| Hai xe cắt nhau / chồng lên nhau | Mỗi xe giữ đúng ID của mình, không hoán đổi ID | Giữ vững danh tính thực sự của từng phương tiện khi giao cắt |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox chỉ ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **chiều rộng tối thiểu 18px và nhìn rõ bánh xe/khung xe** |
| Xe đang đỗ, không di chuyển | Gán và giữ nguyên ID suốt thời gian xe nằm trong khung hình |
| Keyframe đặt dày ở đâu | Đặt dày quanh các khúc cua, chuyển làn, thay đổi tốc độ đột ngột, và quanh frame bắt đầu/kết thúc bị che khuất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 1–15 / ID 2
- Tình huống: Xe dừng chờ tại làn rẽ, bbox không di chuyển qua nhiều frame liên tục.
- Quyết định: Vẫn gán track và giữ nguyên ID 2.
- Lý do: Xe thật đang đỗ/dừng trên đường giao thông, không phải vật thể nền hay lỗi quên bấm Outside.

### Ca 2
- Clip / frame / ID: `clip_01` / frame 55–65 / ID 4
- Tình huống: Xe di chuyển qua bóng cây và bị cột đèn/xe khác che một phần thân xe.
- Quyết định: Giữ nguyên ID 4, thu hẹp bbox chỉ ôm phần thân xe nhìn thấy được, đặt keyframe tại frame 55, 60, 65.
- Lý do: Thời gian che khuất chỉ khoảng 10 frame (< 25 frame) và quỹ đạo chuyển động rõ ràng.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 106–120 / ID 5
- Tình huống: Xe ở hậu cảnh xa bắt đầu rẽ vào từ góc trên bên phải, hình ảnh mờ.
- Quyết định: Bắt đầu gán từ frame 106 khi nhận diện được khối xe 4 bánh, không chờ đến khi xe lại gần.
- Lý do: Đảm bảo độ bao phủ (coverage), giảm False Negative (FN) khi đối chiếu với ground truth.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Xe dừng đỗ**: Làm rõ rằng xe đỗ ven đường vẫn phải gán `vehicle`, trừ khi đỗ sâu trong sân/gara nhà dân bị che khuất > 50%.
- **Thời điểm bấm Outside**: Quy định bấm `O` ngay frame đầu tiên mà thân xe nhìn thấy giảm dưới 10% diện tích hoặc tâm xe vượt quá viền khung hình để tránh lỗi bbox treo.
