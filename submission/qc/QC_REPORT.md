# Báo cáo QC — TEAMAIHEINEKEN

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả, hiệu chỉnh và QC nhãn bounding box 2D cho người và phương tiện trong 40 ảnh thực hành Day 07. Lô ảnh được tuyển chọn để luyện ca biên, chưa được đánh giá là đại diện cho dữ liệu vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: Tác giả, giấy phép và URL nguồn của từng ảnh được ghi trong `data/ATTRIBUTION.csv`; thông tin tương ứng được khóa trong `submission/qc/batch_manifest.json`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; nhãn cuối `submission/qc/labels_calibrated.json` có SHA-256 `9c6c8e1ec95b9ef60849b8a8992ccdbafd73f2246f376c6b1e6da2eedb227d42`.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo taxonomy và guideline v1.0; bàn giao theo `submission/guidelines/guideline_v1.1_calibrated.md` và taxonomy v1.0. V1.1 chỉ chuẩn hóa cách áp dụng sau calibration, không sửa class, thuộc tính hay ngưỡng rule gốc.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt gán, 40/40 ảnh có ít nhất hai người gán, tổng cộng 4 thành viên `203`, `209`, `235`, `242`.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 316 cặp box ghép một-một ở IoU >= 0.5; IoU trung bình 0.813; đồng thuận lớp 90.2%; mẫu số là 316 cặp box ghép trên các cặp người có ảnh chung.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 121/84 box chưa ghép theo hai phía của các cặp so sánh; 0 cặp bị loại do ignore; 59 ca bất đồng vùng `ignore_region` được đưa vào calibration.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Các ca mô phỏng, phản chiếu, xe diễu hành và phương tiện ba bánh được đối chiếu trực tiếp ảnh gốc với R-02, R-04, R-05 và R-06. Ví dụ `img_019.jpg` được kiểm lại theo R-05 để tách xe thật khỏi mascot; `img_022.jpg` được kiểm theo R-02 để phân loại phương tiện ba bánh.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 542 ca IAA ban đầu và ca `QC-R1-001` được ghi trong `submission/qc/calibration_decisions.csv`; `235` và `203` chốt, `209` và `242` kiểm lại. Mỗi dòng lưu ảnh, quyết định, rule tham chiếu, phiếu sửa và người kiểm.
- Tác động v1.1 lên nhãn và ảnh: Giữ nguyên rule và taxonomy v1.0; chuẩn hóa class, bbox, thuộc tính và cách dùng `ignore_region`. Phạm vi calibration cuối gồm 35 ảnh; bản cuối có 40 ảnh và 518 annotation.
- Tham chiếu bằng chứng khi Rework: Round 1 tại `submission/qc/reviews/round-01/` phát hiện các ca cần kiểm lại. Nhãn được sửa tại `submission/qc/labels_calibrated_0cfc2261eb989e7d.json`, build lại và duyệt toàn bộ ở round 2 tại `submission/qc/reviews/round-02/`.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: `209`, `242`; ngày 2026-11-17.
- Đã duyệt / cả lô: 40/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: 0/40, tương đương 0.0%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/518, tương đương 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 ở round 2.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: 34/34 ảnh ca khó và 6/6 ảnh còn lại đã duyệt, đều không còn lỗi; chi tiết tại `submission/qc/reviews/round-02/qc_review.csv` và `qc_report_auto.md`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được box ngoài ảnh, box thấp hơn ngưỡng, class/thuộc tính ngoài taxonomy hoặc `can_review` còn treo. Người đọc ảnh cần quyết định các ca ngữ nghĩa như xe thật có mascot ở `img_019.jpg`, phản chiếu, tượng và sidecar ở `img_022.jpg`.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Round 2 đã duyệt toàn bộ 40 ảnh, không còn lỗi ghi nhận, checksum nhãn khớp manifest và không còn `can_review`; bàn giao cho coach/nhóm chấm trong phạm vi bài thực hành đặc tả và QC bbox 2D Day 07.
- Nếu còn việc: Không còn sửa nhãn theo round 2. Trước khi nộp, các thành viên còn thiếu phải bổ sung guideline/ghi chú cá nhân và nhóm chạy lại `python scripts/check_submission.py`.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: `QC-R1-001` truy về `img_022.jpg`, R-02, quyết định trong `calibration_decisions.csv`, nhãn `three_wheeler` trong `labels_calibrated.json` và overlay round 2; kết quả đạt. Ca `img_019.jpg` truy về R-05 và box `car` trong bản nhãn cuối; kết quả đạt.
