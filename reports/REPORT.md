# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Hoàng Gia Linh` (Làm cá nhân)
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 50 phút |
| Số track đã vẽ trong `clip_01` | 8 track (tổng 632 bbox) |
| Số keyframe trung bình mỗi track | ~10 keyframe / track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất một phần khi mới xuất hiện (ID 6 - frame 79-100)**: Chỉ bắt đầu khởi tạo track khi phần nhìn thấy của xe đạt ngưỡng visibility >= 0.2, giữ nguyên ID cũ khi xe hiện lại hẳn.
2. **Xe dừng đỗ cố định (ID 2 - frame 0-189)**: Rà soát lại mỗi 15 frame để tránh bbox bị co giãn hay trôi lệch vị trí do CVAT tự động nội suy.
3. **Xác định ranh giới ngắt track khi xe ra khỏi khung (ID 3 - frame 44-45)**: Bấm thuộc tính `Outside` chính xác tại frame xe khuất hẳn khỏi rìa ảnh để tránh CVAT kéo dài bbox thừa ngoài khung hình.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1**: Kiểm tra tính liên tục của ID các xe trong toàn bộ video, đảm bảo không có xe nào bị nhảy ID hay hoán đổi ID (IDSW = 0).
- **Lượt 2**: Rà soát frame xuất hiện và ngắt track (`Outside`) của từng xe, phát hiện 2 đoạn kéo dài bbox thừa ở ID 5 và ID 6 khi xe đã ra khỏi rìa.
- **Lượt 3**: Kiểm tra độ khít của bbox ở các frame trung gian giữa các keyframe.

Kiểm chéo với: `Làm cá nhân.
Số lỗi bạn tìm được trong bản của bạn ấy: `Không. Số lỗi bạn ấy tìm được trong bản của bạn: `Không.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `964671e6ee6ae30f8ddf12dec0f4fbc786935b1c9e0ed891bb3737d319b1d1f0` |
| Thời điểm khóa | `2026-09-15T03:03:11.833130+00:00` |
| Số row / frame / track trước khi mở reference | `632 row / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.800 | 0.785 | 0.817 | 0.884 | 0.948 | 0.890 | 0.874 | 61 | 2 | 0 |
| Sau rework | 0.800 | 0.785 | 0.817 | 0.884 | 0.948 | 0.890 | 0.874 | 61 | 2 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:
Do đã đạt mức ngay ở lần đầu tiên nên không chạy bản rework
| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa (tạo sớm) | 79-100 | 6 | Chỉnh frame bắt đầu track lùi lại đúng mốc xe đạt visibility >= 0.2 |


## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.800 | 0.785 | 0.817 | 0.884 | 0.948 | 0.890 | 0.874 | 61 | 2 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.769 | 0.711 | 0.834 | 0.917 | 0.868 | 0.736 | 0.912 | 86 | 80 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Bản gán nhãn tay của tôi có **IDF1 (0.948) cao hơn MOTA (0.890)**. 
- Nhãn tay đạt độ chính xác gán định danh tuyệt đối với **0 lỗi ID switch (IDSW = 0)**. 
- Nếu một bản gán có MOTA cao nhưng IDF1 thấp, điều đó cho biết đối tượng được phát hiện đúng vị trí (ít FP/FN) nhưng bị **nhảy ID hoặc tách track nhiều lần**. 
- MOTA không phạt nặng lỗi ID vì công thức `MOTA = 1 - (FP + FN + IDSW) / GT` chỉ phạt mỗi lần chuyển đổi ID (IDSW) với trọng số bằng 1 lần bỏ sót hay bắt thừa. Trong khi đó, IDF1 đo lường tỉ lệ gán đúng ID xuyên suốt toàn bộ quãng đời đối tượng (F1-score of ID mapping), do đó phạt rất nặng tính thiếu nhất quán về identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- BoT-SORT + ReID cho kết quả vượt trội so với ByteTrack control: **IDF1 tăng từ 0.875 lên 0.900**, **AssA tăng từ 0.776 lên 0.820**, và số lượng bỏ sót (FN) giảm mạnh từ 54 xuống 26. Số lần nhảy ID giữ nguyên ở mức 2 IDSW.
- *Ví dụ chuỗi frame*: Tại các frame xe bị vật cản che khuất ngắn hạn (occlusion), mô hình BoT-SORT + ReID nhờ kết hợp đặc trưng ngoại hình (appearance embeddings) giúp duy trì lại track cũ tốt hơn hẳn so với chỉ dựa vào IoU/Kalman của ByteTrack.
- *Lưu ý*: Đây là so sánh mức hệ thống (system comparison), không thể cô lập duy nhất hiệu ứng nhân quả của ReID vì ByteTrack và BoT-SORT sử dụng hai thuật toán ghép nối và Kalman filter khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- DetA tăng từ **0.649 (ByteTrack) lên 0.711 (ReID)**, chủ yếu nhờ FN giảm sâu từ 54 xuống 26 (detector bớt bỏ sót xe mờ/nhỏ).
- Số lượng FP của cả hai mô hình vẫn ở mức cao (88 ở ByteTrack và 91 ở ReID) do detector YOLO26n phát hiện nhầm các vật thể cố định/nhiễu ngoài đường thành xe.
- Phần lớn lỗi còn lại xuất phát từ **Detector** (False Positives nhầm nhiễu và chưa phát hiện khít ranh giới) kết hợp với hiện tượng tách track (track gold 4, 5, 7 bị mô hình cắt thành nhiều ID lẻ).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame 104-107, T7 và T27**: Mô hình ReID nhận diện phần rải phân cách ven đường ở góc trái và quầy hàng trên vỉa hè (vật tĩnh) thành xe

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Frame 104**: Mô hình ReID bắt được một xe nhỏ ở rìa xa bên phải khung hình. Trong lần rà soát ban đầu của nhãn tay, xe này chưa được gán do bị mờ góc viền. Sau khi đối chiếu, xe này xuất hiện và đạt ngưỡng visibility <= 0.2 nên chưa bổ sung gán nhẵn.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`**: Quy định ranh giới ngắt `Outside` nghiêm ngặt ngay tại frame xe khuất hẳn khỏi ảnh, và thiết lập cố định ngưỡng xe mờ/nhỏ từ visibility >= 0.2 để đồng bộ giữa tất cả các clip.
- **Thay đổi quy trình làm việc**: Dành riêng lượt tua thứ 2 tập trung rà soát thuộc tính `Outside` và mốc xuất hiện đầu tiên của từng xe. Đặt keyframe định kỳ 10-15 frame cho các xe đứng yên để tối ưu hóa điểm `MOTP` và `LocA`.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` 
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [] `reports/review_partner.md`(không làm vì bài làm cá nhân và so sánh với gold của Lab Coach cung cấp, không review partner)
- [x] `reports/REPORT.md` 
