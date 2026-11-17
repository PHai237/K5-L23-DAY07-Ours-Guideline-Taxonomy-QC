# Báo cáo QC — Ours

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Bài thực hành đặc tả, calibration và QC nhãn phát hiện vật thể 2D trên 40 ảnh; dùng cho phạm vi bài lab, chưa đánh giá độ đại diện cho dữ liệu vận hành.
- Nguồn và phép sử dụng dữ liệu: Theo `data/ATTRIBUTION.csv`; nguồn, tác giả, giấy phép và URL từng ảnh được giữ trong `submission/qc/batch_manifest.json`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `submission/qc/labels_calibrated.json` có SHA-256 `946b1ff2a82a2e324e88e24a33b35a08b27d7ecd211af83d52bb6dce9203b02b`.
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo guideline v1.0 và taxonomy v1.0; bàn giao theo guideline v1.1 editorial, không đổi taxonomy hoặc ý nghĩa lớp.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt trên 40 ảnh, mỗi ảnh có 3 người gán.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 853 cặp box ghép; IoU trung bình 0.896; cùng lớp 90.2% trên 853 cặp, chưa đạt ngưỡng 95% trước calibration.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Chưa ghép 332/446 box theo hai phía của các cặp so sánh; 0 box bỏ qua do ignore; 33 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: M001 tại `img_009.jpg`: consensus còn box `van` bao minibus dù R-02 quy định bus/minibus ngoài phạm vi; nhóm đã xóa box này trước vòng QC 3.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 1.426/1.426 ca IAA đã được nhóm chốt trong `calibration_decisions.csv`, cộng ca chung M001 theo R-02; người chốt 047, reviewer 054 kiểm lại.
- Tác động v1.1 lên nhãn và ảnh: 38/40 ảnh thuộc phạm vi calibration; nhãn cuối có 40 ảnh, 639 annotation, không còn `can_review=true`; taxonomy v1.0 được giữ nguyên.
- Tham chiếu bằng chứng khi Rework: Vòng 2 bị thay thế sau khi phát hiện M001; box minibus ở `img_009.jpg` được xóa, nhãn được build lại. Vòng 8 là vòng bàn giao cuối gắn với guideline và manifest hiện tại.

## 4. Kết quả duyệt

- Vòng QC: 8
- Người review và ngày: 054 (Vũ Duy Thế), 2026-11-17.
- Đã duyệt / cả lô: 40/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/639 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót sau khi nhóm duyệt toàn bộ 40 ảnh.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: 38/38 ảnh khó và 2/2 ảnh còn lại đã được duyệt; không còn lỗi ở vòng 8. Ví dụ ca sửa: M001, `img_009.jpg`, R-02.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script chặn schema, box ngoài ảnh, checksum sai và cờ `can_review`; người đọc ảnh phát hiện lỗi chung như minibus bị nhiều người cùng gán thành `van`.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Release cho bài lab vì 40/40 ảnh đã được reviewer 054 duyệt, 0/40 ảnh lỗi, 0/639 annotation lỗi và không còn cờ `can_review`; người nhận là coach, phạm vi là bài thực hành Day 07.
- Nếu còn việc: Điền commit chốt bài trong `TEAM.md` sau khi commit/push và mở thử repo bàn giao bằng một bản clone sạch.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: M001 → `img_009.jpg` → R-02 trong guideline v1.1 → dòng M001 trong `calibration_decisions.csv` → box minibus đã không còn trong `labels_calibrated.json` → vòng 8 ghi `khong` bởi reviewer 054.
