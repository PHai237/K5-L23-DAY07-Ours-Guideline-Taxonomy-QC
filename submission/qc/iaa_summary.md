# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 853 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 853 | 0.896 | 90.2% | 66.6% | 87.1% | 330/430 | 0 | CHƯA ĐẠT |
| 047|054 | 13 | 91 | 0.888 | 83.5% | 75.8% | 83.5% | 38/57 | 0 | CHƯA ĐẠT |
| 047|325 | 12 | 96 | 0.867 | 88.5% | 69.8% | 81.2% | 78/24 | 0 | CHƯA ĐẠT |
| 047|332 | 12 | 101 | 0.827 | 96.0% | 42.6% | 91.1% | 30/43 | 0 | CHƯA ĐẠT |
| 047|353 | 11 | 85 | 0.863 | 94.1% | 68.2% | 78.8% | 25/52 | 0 | CHƯA ĐẠT |
| 054|325 | 13 | 98 | 0.989 | 83.7% | 80.6% | 96.9% | 60/9 | 0 | CHƯA ĐẠT |
| 054|332 | 11 | 67 | 0.894 | 88.1% | 47.8% | 83.6% | 18/14 | 0 | CHƯA ĐẠT |
| 054|353 | 11 | 60 | 0.968 | 90.0% | 81.7% | 88.3% | 11/35 | 0 | CHƯA ĐẠT |
| 325|332 | 11 | 82 | 0.853 | 92.7% | 53.7% | 81.7% | 25/51 | 0 | CHƯA ĐẠT |
| 325|353 | 12 | 75 | 0.985 | 92.0% | 76.0% | 92.0% | 19/72 | 0 | CHƯA ĐẠT |
| 332|353 | 14 | 98 | 0.867 | 92.9% | 71.4% | 91.8% | 26/73 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1426 ({'bo_sot_thua': 753, 'iou_thap': 202, 'sai_thuoc_tinh': 351, 'sai_lop': 84, 'co_can_review': 3, 'bat_dong_ignore': 33}), trên 38 ảnh cần calibration.
