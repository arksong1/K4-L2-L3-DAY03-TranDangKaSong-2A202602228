# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `06 / Trần Đăng Ka Song`
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

Bổ sung của nhóm (nếu có): Không gán xe đang được chở trên xe tải khác.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Vẫn có thể nhận diện và nội suy chính xác quỹ đạo |
| Xe bị che lâu hơn ngưỡng trên | Tạo track mới (ID mới) | Tránh sai sót do nội suy quá dài hoặc nhầm lẫn với xe khác |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Đã ra ngoài phạm vi camera thì coi như mất dấu hoàn toàn |
| Hai xe cắt nhau / chồng lên nhau | Gán đúng ID cho từng xe, bbox chỉ ôm phần nhìn thấy của xe bị che | Tránh chồng chéo ID và dính BBOX |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: có thể nhìn rõ hình khối tối thiểu (ví dụ > 15x15 px) |
| Xe đang đỗ, không di chuyển | Vẫn gán ID xuyên suốt nhưng chỉ cần 2 keyframe ở đầu/cuối nếu không bị xe khác che |
| Keyframe đặt dày ở đâu | Điểm xe bắt đầu rẽ, chuyển hướng, đổi tốc độ đột ngột hoặc chuẩn bị bị che khuất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: clip_01 / frame 87 / ID 5
- Tình huống: Xe bị che khuất một phần trong thời gian ngắn (đứt ID trên model).
- Quyết định: Vẫn giữ nguyên ID (không tạo ID mới) và kéo keyframe nội suy qua đoạn này.
- Lý do: Xe đi theo một đường thẳng, mất dấu rất nhanh nên hoàn toàn đoán được vị trí.

### Ca 2
- Clip / frame / ID: clip_01 / frame 79-100 / ID 6
- Tình huống: Xe tiến ra sát mép và từ từ rời khỏi khung hình.
- Quyết định: Phải bấm 'outside' ngay frame đầu tiên xe khuất hẳn 100% khỏi ảnh.
- Lý do: Nhóm quên bấm outside sinh ra lỗi BBOX treo kéo dài 22 frame ở file log.

### Ca 3
- Clip / frame / ID: clip_01 / frame 139 / ID 1
- Tình huống: Khung hình chụp rõ xe nhưng góc nhìn nghiêng khiến hình chiếu thay đổi.
- Quyết định: Cần thống nhất có khoanh cả gương, ăng-ten hay mép bánh xe không.
- Lý do: BBOX gán và BBOX của model chênh nhau khá lớn (IoU tụt còn 0.50), cần quy chuẩn chặt chẽ hơn.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Thiếu quy định kiểm tra BBOX treo: Bắt buộc phải có bước soi lại (lượt 2) ngay khoảnh khắc xe rời rìa màn hình, nhằm bấm phím 'Outside' để tắt track kịp thời.
- Mơ hồ về mép xe (cạnh BBOX): Cần định nghĩa rõ BBOX có tính phần bóng đổ xuống mặt đường hoặc gương chiếu hậu hay không, tránh tình trạng IoU thấp (BBOX lệch) so với mô hình và người kiểm chéo.
