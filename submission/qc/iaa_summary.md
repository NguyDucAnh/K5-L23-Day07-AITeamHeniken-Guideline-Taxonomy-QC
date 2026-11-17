# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 316 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 316 | 0.813 | 90.2% | 73.7% | 88.6% | 121/84 | 0 | CHƯA ĐẠT |
| 203|209 | 6 | 23 | 0.807 | 82.6% | 60.9% | 95.7% | 4/8 | 0 | CHƯA ĐẠT |
| 203|235 | 9 | 88 | 0.762 | 88.6% | 89.8% | 93.2% | 42/31 | 0 | CHƯA ĐẠT |
| 203|242 | 5 | 49 | 0.747 | 79.6% | 83.7% | 87.8% | 16/14 | 0 | CHƯA ĐẠT |
| 209|235 | 5 | 32 | 0.848 | 93.8% | 59.4% | 71.9% | 7/20 | 0 | CHƯA ĐẠT |
| 209|242 | 9 | 90 | 0.865 | 98.9% | 55.6% | 88.9% | 24/4 | 0 | ĐẠT |
| 235|242 | 6 | 34 | 0.873 | 88.2% | 88.2% | 88.2% | 28/7 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 542 ({'iou_thap': 155, 'sai_lop': 31, 'sai_thuoc_tinh': 112, 'bo_sot_thua': 183, 'bat_dong_ignore': 59, 'co_can_review': 2}), trên 34 ảnh cần calibration.
