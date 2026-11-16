# Guideline gán nhãn Bounding Box – v0.1 (Individual)

- **Phiên bản:** v0.1
- **Người viết:** 332
- **Loại nhãn:** Bounding Box
- **Công cụ:** CVAT
- **Định dạng xuất:** COCO 1.0
- **Phạm vi:** Người và phương tiện trong ảnh đường phố.

---

## 1. Mục tiêu

Xây dựng bộ dữ liệu gán nhãn phục vụ huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố.
Đảm bảo các đối tượng được gán nhãn đầy đủ, đúng lớp, đúng vị trí và thống nhất giữa các annotator.

## 2. Lớp và thuộc tính

### 2.1. Các lớp đối tượng (Classes)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô chở người, bao gồm sedan, hatchback, SUV. |
| `truck` | Phương tiện chủ yếu dùng để vận chuyển hàng hóa, như xe tải thùng, xe ben. |
| `van` | Phương tiện có thân xe dạng hộp, thường dùng để chở người hoặc hàng hóa. |
| `motorcycle` | Xe máy, mô tô hai bánh. |
| `person` | Người đi bộ hoặc đứng trên đường phố, không ngồi trên xe máy hoặc xe đạp. |
| `rider` | Người điều khiển hoặc ngồi trên xe máy, xe đạp, bao gồm cả người ngồi phía sau. |
| `ignore_region` | Vùng chứa đối tượng không cần hoặc không thể gán nhãn riêng theo quy định. |

**Nguyên tắc phân loại:**
- Mỗi đối tượng thuộc đúng một class.
- Người đi bộ được gán `person`.
- Người ngồi hoặc điều khiển xe máy, xe đạp được gán `rider`.
- Hành khách bên trong ô tô, xe bus, xe tải và xe van không gán nhãn.
- Không tự ý tạo thêm class khi chưa được nhóm thống nhất.

### 2.2. Các thuộc tính (Attributes)

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức độ che khuất: 0 không bị che, 1 che ít, 2 che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Đối tượng có bị cắt bởi mép ảnh hay không. |
| `can_review` | Tick / không | Đánh dấu khi không chắc chắn về class, box hoặc quyết định gán nhãn. |

## 3. Quy tắc vẽ Bounding Box

1. Gán nhãn các đối tượng thuộc phạm vi và đáp ứng điều kiện gán nhãn.
2. Mỗi đối tượng có một Bounding Box riêng, bao sát phần đối tượng nhìn thấy được.
3. Không gộp nhiều đối tượng vào cùng một box hoặc tạo box trùng lặp.
4. Đối tượng bị che khuất vẫn được gán nhãn nếu có đủ bằng chứng nhận diện.
5. Đối tượng bị cắt bởi mép ảnh: box chỉ bao phần nằm trong ảnh và đặt `truncated = yes`.
6. Box không bao gồm phần bị che khuất hoặc phần không quan sát được.
7. Người và phương tiện được gán box riêng, không gộp thành một đối tượng.

### 3.1. Quy tắc gán nhãn người

- Người đi bộ, đứng hoặc di chuyển bằng cách đi bộ trên đường phố: `person`.
- Người điều khiển hoặc ngồi sau xe máy, xe đạp: `rider`.
- Người dắt xe máy hoặc xe đạp khi đang đi bộ: `person`.
- Mỗi người được gán một box riêng nếu có thể phân biệt.
- Không gán nhãn hành khách bên trong ô tô, xe bus, xe tải và xe van, kể cả khi nhìn thấy qua cửa kính.

## 4. Các trường hợp khó (Edge Cases)

| Tình huống | Quyết định |
|---|---|
| Xe bị che khuất một phần | Gán nhãn nếu nhận diện được, thiết lập `occluded_level`. |
| Xe bị cắt bởi mép ảnh | Gán phần nhìn thấy, đặt `truncated = yes`. |
| Người ngồi sau xe máy | `rider` |
| Người đi xe đạp | `rider` |
| Người đứng cạnh hoặc dắt xe | `person` |
| Hành khách trong ô tô, xe bus | Không gán nhãn |
| Tượng hoặc ma-nơ-canh giống người | Không gán `person`. |
| Xe chở xe khác | Chỉ gán xe chở, không gán các xe được chở. |
| Phương tiện chưa có class phù hợp | Đánh dấu `can_review`, đề xuất nhóm thống nhất cách xử lý. |
| Đối tượng quá nhỏ hoặc không nhận diện được | không gán nhãn |

## 5. Kiểm tra chất lượng (QC)

Sau khi gán nhãn, kiểm tra:

- Không bỏ sót hoặc gán thừa đối tượng.
- Class được lựa chọn chính xác.
- Bounding Box bao sát đối tượng.
- Attributes được thiết lập đúng.
- Không nhầm lẫn giữa `person` và `rider`.
- Các trường hợp chưa chắc chắn được đánh dấu `can_review`.

Các lỗi QC được phân loại thành:
- `bo_sot`: Bỏ sót đối tượng.
- `thua`: Gán thừa đối tượng.
- `sai_lop`: Sai class.
- `sai_vi_tri_kich_thuoc`: Sai Bounding Box.
- `sai_thuoc_tinh`: Sai attribute.
- `khong`: Không có lỗi.

## 6. Calibration và quản lý phiên bản

- Khi các annotator có kết quả khác nhau, cần đối chiếu ảnh và guideline để thống nhất quyết định.
- Ghi lại các trường hợp bất đồng và kết quả xử lý trong `calibration_decisions.csv`.
- Mọi thay đổi ảnh hưởng đến kết quả gán nhãn phải được ghi nhận và cập nhật phiên bản guideline.
- Không tự ý thay đổi quy tắc chung trong quá trình annotation.

| Phiên bản | Nội dung |
|---|---|
| v0 | Guideline ban đầu của giảng viên. |
| v0.1 | Bổ sung class `rider` và các quy tắc phân loại, gán nhãn, QC cá nhân. |
| v1.0 | Guideline được nhóm thống nhất. |
| v1.1 | Guideline được hiệu chỉnh sau calibration. |

---

**Ghi chú:** Đây là guideline cá nhân, cần được đối chiếu với RULES.md và taxonomy chính thức, sau đó thảo luận với nhóm trước khi áp dụng chung.