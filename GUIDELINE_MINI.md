# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Hoàng Gia Linh`
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

Bổ sung của nhóm (nếu có): `Chỉ gán xe khi phần nhìn thấy được (visibility) đạt từ 0.2 trở lên. Không gán xe đồ chơi, xe mô hình hoặc xe 3 bánh.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Tránh nhảy ID (ID switch) khi xe tạm thời bị vật cản (cây, cột, xe khác) che khuất ngắn hạn |
| Xe bị che lâu hơn ngưỡng trên | Tạo track ID mới khi xe xuất hiện lại | Sau 25 frames (> 2 giây), không thể đảm bảo độ tin cậy về nhận dạng identity |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Xe đã ra khỏi góc nhìn FOV không đảm bảo là cùng một xe khi vào lại |
| Hai xe cắt nhau / chồng lên nhau | Xe đằng trước giữ nguyên ID; xe đằng sau giữ ID cũ nếu lộ lại dưới 25 frame | Tránh tráo đổi ID (ID swap) giữa 2 xe khi giao nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** (visible area) |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh và phần nhìn thấy (visibility) đạt từ **0.2 (20%) trở lên**. |
| Xe đang đỗ, không di chuyển | Đặt keyframe cố định đầu/cuối đoạn đỗ, kiểm tra định kỳ 10-15 frame để tránh bbox trôi lệch IoU |
| Keyframe đặt dày ở đâu | Đặt dày tại các frame xe bắt đầu đổi hướng, tăng/giảm tốc, hoặc khi vừa chui ra khỏi vật che |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1: Xe ID 6 bị che khuất khi mới xuất hiện
- Clip / frame / ID: `clip_01 / frame 79-100 / ID 6`
- Tình huống: Xe ID 6 vừa chớm xuất hiện nhưng bị xe khác che khuất phần lớn thân xe.
- Quyết định: Chưa tạo track khi bị che quá 80%; bắt đầu khởi tạo ID 6 từ frame xe lộ ra đạt ngưỡng visibility >= 0.2 và giữ nguyên ID khi hiện lại.
- Lý do: Tránh gán nhãn sai/tạo track ảo quá sớm khi chưa xác định chắc chắn được đối tượng.

### Ca 2: Xe đứng yên không di chuyển
- Clip / frame / ID: `clip_01 / frame 0-189 / ID 2`
- Tình huống: Xe đỗ cố định trong nhiều frame liên tục.
- Quyết định: Khóa vị trí bbox bằng keyframe tại điểm bắt đầu và kết thúc đứng yên, rà soát lại định kỳ mỗi 15 frame.
- Lý do: Đảm bảo bbox đứng im tuyệt đối, không bị trôi hay biến dạng do nội suy (interpolation) của phần mềm.

### Ca 3: Xe đi ra ngoài khung hình (Outside)
- Clip / frame / ID: `clip_01 / frame 44-45 / ID 3`
- Tình huống: Xe di chuyển ra khỏi rìa ảnh và biến mất dần.
- Quyết định: Đặt thuộc tính `Outside` chính xác tại frame đầu tiên xe hoàn toàn rời khỏi ranh giới ảnh.
- Lý do: Ngắt track kịp thời, tránh lỗi bbox treo/thừa (kéo dài vô nghĩa ra vùng ngoài ảnh) làm giảm điểm MOTA/DetA.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Luật về bbox xe vừa xuất hiện**: Chỉ gắn bbox khi xuất hiện trên 20% xe và đủ bằng chứng xác định rõ được là xe 4 bánh
- **Mật độ Keyframe khi chuyển động phức tạp**: Khoảng cách keyframe không nên vượt quá 10 frame đối với các đoạn xe rẽ hoặc đổi tốc độ để đảm bảo `MOTP` và `LocA` đạt điểm cao (IoU > 0.7).
