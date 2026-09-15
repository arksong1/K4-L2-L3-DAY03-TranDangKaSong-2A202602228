# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Trần Đăng Ka Song / 6`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `90` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `78` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Frame dài ngồi gán nhãn mất thời gian và chưa có cách xử lý
2. Xe bị che khó track và giải pháp zoom to ảnh
3. Mới đầu làm chưa quen và mất đôi chút thời gian làm quen

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Phát hiện và nối lại các đoạn đứt ID (ID switch) hoặc 1 xe bị nhảy sang ID mới sau khi bị che khuất.
- Lượt 2: Bắt được thời điểm chính xác xe đi vào và rời khỏi khung hình. Tuy nhiên, vẫn còn sót lỗi chưa bấm 'outside' kịp thời khiến sinh ra lỗi BBOX treo ở một số ID (4, 5, 6, 8) lúc xe đi sát mép cam.
- Lượt 3: Tua qua các frame giữa và phát hiện BBOX bị lệch (trôi) khỏi mép xe do xe chuyển động cong hoặc thay đổi tốc độ. Đã thêm các keyframe để nắn lại quỹ đạo (nhưng vẫn còn sót vài frame bị trôi chưa khít hẳn ở ID 4, 5).

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `85d1fb527280b07ac9c72176140d5679e112c0381e7b7c6acd165b8eeb72e7df` |
| Thời điểm khóa | `2026-09-15T10:00:33.583674+00:00` |
| Số row / frame / track trước khi mở reference | `615 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.825 | 0.805 | 0.847 | 0.891 | 0.955 | 0.906 | 0.881 | 48 | 6 | 0 |
| Sau rework | 0.842 | 0.819 | 0.868 | 0.891 | 0.967 | 0.911 | 0.881 | 44 | 12 | 1 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TREO | 79-100 | 6 | Bấm outside đúng frame xe rời khung |
| BBOX TREO | 149-151, 51-53 | 4 | Bấm outside đúng frame xe rời khung |
| BBOX TREO | 76-78 | 5 | Bấm outside đúng frame xe rời khung |
| BBOX TREO | 169-171 | 8 | Bấm outside đúng frame xe rời khung |
| BBOX TRÔI | 81, 83, 85, 88, 96 | 5 | Thêm keyframe quanh đây để khớp khít vật thể |
| BBOX TRÔI | 54 | 4 | Thêm keyframe quanh đây để khớp khít vật thể |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `/content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.7` / `960` / `[2, 5, 7]` |
| device | `0` |

                    HOTA    DetA    AssA    LocA    IDF1    MOTA    MOTP      FP      FN    IDSW
------------------------------------------------------------------------------------------------
ban_vs_gold        0.825   0.805   0.847   0.891   0.955   0.906   0.881      48       6       0
bytetrack_vs_gold   0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2
reid_vs_gold       0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2
reid_vs_ban        0.761   0.708   0.819   0.894   0.884   0.767   0.884      82      59       2

Cổng annotation: ĐẠT {'IDF1': 0.955, 'MOTA': 0.906, 'MOTP': 0.881}

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của em là 0.906, thấp hơn IDF1 là 0.955. Trong trường hợp MOTA cao mà IDF1 thấp, điều này có nghĩa là bộ tracking làm rất tốt trong việc phát hiện vật thể (ít FP và FN), nhưng lại kém trong việc duy trì ID của đối tượng (xảy ra nhiều ID Switch hoặc tách track). MOTA không phạt nặng lỗi ID vì công thức tính MOTA = 1 - (FP + FN + IDSW) / GT. Trong đó, FP và FN được cộng dồn theo từng frame, trong khi IDSW chỉ bị tính là 1 lỗi duy nhất tại thời điểm chuyển đổi. Do đó, một lỗi gán sai ID kéo dài nhiều frame cũng chỉ bị phạt 1 lỗi IDSW, khiến MOTA chủ yếu phản ánh chất lượng detection thay vì khả năng liên kết (association).

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So sánh ByteTrack và BoT-SORT + ReID:
- IDF1 tăng từ 0.875 lên 0.900, AssA tăng từ 0.776 lên 0.820, IDSW giữ nguyên là 2.
- Việc AssA và IDF1 tăng đáng kể cho thấy treatment BoT-SORT + ReID có khả năng duy trì track dài và ổn định hơn, kết nối ID tốt hơn so với ByteTrack.
- Ví dụ cụ thể: track gold 4 (95 frame) bị chia thành 2 ID [15, 14] ở frame 59 trong ByteTrack, nhưng trong mô hình BoT-SORT+ReID thì track 4 không bị ID switch. Điều này cho thấy ReID có thể giúp bắt lại ID tốt hơn sau khi bị che khuất. Tuy nhiên, lưu ý rằng không thể kết luận điều này hoàn toàn là tác động nhân quả (causal effect) của riêng module ReID, do bản thân hai tracker implementation (ByteTrack và BoT-SORT) đã có kiến trúc khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So sánh ByteTrack và BoT-SORT + ReID:
- DetA tăng từ 0.649 lên 0.711. FP tăng nhẹ từ 88 lên 91. FN giảm mạnh từ 54 xuống 26.
- Mặc dù sử dụng chung detector (yolo26n.pt), tracker BoT-SORT giúp duy trì track tốt hơn, từ đó giảm đáng kể số FN (có thể do cơ chế track management giữ track sống lâu hơn).
- Lỗi còn lại chủ yếu nằm ở detector: Điểm AssA (0.820) khá cao, trong khi DetA (0.711) thấp hơn. Lượng FP vẫn khá lớn (91 FP), cho thấy mô hình hay vẽ ra các BBOX thừa không có thật, chứng tỏ detector đang nhận diện nhầm các vật thể không phải xe hoặc ngoài phạm vi.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Em gán đúng nhưng ReID sai ở track số 5 (frame 87). Cụ thể mô hình ReID đã tách track 5 thành 2 ID [18, 17] ở frame 87 (ID switch). Trong khi đó, ở nhãn của em, em đã có thể theo dõi xuyên suốt xe này bằng 1 ID duy nhất nhờ khả năng nội suy (interpolate) của người gán khi xe bị che khuất một phần, còn mô hình mất dấu và sinh ra ID mới.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID khiến em phải xem lại annotation ở frame 139 (track 1) và các frame 107, 112, 118 (track 6). Báo cáo 'reid_vs_ban' chỉ ra rằng ở các frame này IoU giữa bbox của em và bbox của model tụt xuống rất thấp (khoảng 0.50 - 0.56). Điều này có thể do detector của model khoanh sát mép vật thể hơn, hoặc do em đã khoanh bbox bị lệch/quá rộng so với thực tế, cần phải soi lại ảnh gốc để thống nhất lại luật khoanh bbox cho chính xác. Hơn nữa, lỗi BBOX thừa ở ID 7, 27, 38 là minh chứng ReID sai, bắt đối tượng ảo (FP).

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Sửa GUIDELINE_MINI.md: Bổ sung quy định rõ ràng về việc xử lý frame đầu/cuối của đối tượng, đặc biệt là phải đánh dấu outside chính xác ngay khi xe rời khỏi khung hình để tránh lỗi BBOX TREO (như đã gặp ở ID 4, 5, 6, 8 dư từ 3-22 frame). Thống nhất lại định nghĩa 'cạnh xe' để khoanh bbox khít hơn (tránh lỗi BBOX LỆCH ở mục 4).
- Thay đổi quy trình làm việc: Thay vì chú ý quá nhiều vào giữa track, em sẽ kiểm tra kỹ lượt thứ 2 (check frame đầu/cuối của từng ID) để chặn lỗi BBOX treo. Tăng cường sử dụng tính năng interpolate (tạo keyframe và để CVAT tự nội suy) tại những thời điểm xe di chuyển mượt mà, và chỉ chỉnh sửa tay (thêm keyframe) ở những đoạn quỹ đạo cong hoặc kích thước xe thay đổi đột ngột.

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
