# Guideline gán nhãn — v1.0 (bản thống nhất của nhóm)

> Bản chính thức dùng cho lượt gán độc lập (Pha 3). **Đã khóa**: không sửa trong lúc gán.
> Ca chưa rõ → tick `can_review` hoặc ghi vấn đề cấp ảnh; thay đổi chỉ đưa vào v1.1 sau calibration (Pha 5).
> Mỗi quy tắc có mã `R-xx` để dẫn lại trong `calibration_decisions.csv` và nhật ký QC.

- Phiên bản: **v1.0**
- Ngày: 2026-11-16
- Người viết / duyệt: 054 (bổ sung mã các thành viên khác sau họp)
- Taxonomy đi kèm: `submission/taxonomy.json` (`version: v1.0`)
- Dựa trên: `guideline/guideline_v0_starter.md`, các bản cá nhân trong `submission/guidelines/individual/`

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện **mô hình phát hiện vật trên ảnh 2D** (bounding box) cho **người và phương tiện
cơ giới đường bộ** trong ảnh đường phố: camera giao thông, hỗ trợ lái xe.

| Lỗi | Hậu quả | Quy tắc ưu tiên |
|---|---|---|
| **Bỏ sót người đi bộ / người lái xe hai bánh** | mô hình không thấy người → nguy cơ va chạm | R-03, R-05, R-10, R-19 |
| Nhầm `person` ↔ `rider` | mô hình dự đoán sai hướng / tốc độ di chuyển của người | R-19 |
| Gán vật **không thật** (tượng, ảnh in, phản chiếu, đồ chơi) | báo động giả, học sai đặc trưng | R-12 |
| Gộp người và xe vào một box | không đếm được số người, box lệch | R-03 |
| Nhầm `car` / `van` / `truck` | giảm IAA lớp | R-16 |

Nguyên tắc chung: **phân vân giữa “gán” và “không gán” một người thật (ngoài các ca loại trừ ở R-06) → gán và tick `can_review`.**

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

Mỗi vật thuộc **đúng một** lớp. Chỉ gán vật **thật, cỡ thật, có mặt trong cảnh** (R-01).

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người ≤ 9 chỗ, thân liền khối (sedan, hatchback, SUV, crossover, wagon, taxi, MPV), kể cả xe độ thân vỏ vẫn chạy được. | sedan đen img_023; taxi xanh img_040; Mazda xám img_012; xe trưng bày img_024; xe Dangermouse img_019 | thân vuông cao kiểu Hiace → `van`; xe đồ chơi img_038 → không gán (R-12); xe đang **được chở** trên xe khác → không gán (R-14) |
| `truck` | Xe chở hàng có **cabin tách riêng khỏi thùng hàng** (thùng hở, kín, bạt), gồm cả **bán tải**, xe tải nhẹ, xe đầu kéo + rơ-moóc. | Suzuki Carry img_001; bán tải img_012, img_028; Veryca img_036; bán tải gắn nắp thùng img_037; Isuzu ELF, Hilux thùng kín img_007; xe chở ô tô img_002 | thân liền một khối → `van`; tượng bán tải img_027 → không gán |
| `van` | Xe thân hộp **liền một khối** (cabin và khoang chở chung vỏ), mui cao, thường có cửa trượt; cỡ Hiace / Transit. | Hiace img_005, img_016; xe cứu thương thân hộp img_011 | minibus có cửa hành khách riêng img_009 → ngoài phạm vi (R-02); SUV → `car` |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, tay ga, mô tô), kể cả thùng / thuyền **gắn cố định** bên hông. | tay ga img_010; xe số img_006; Vespa img_016; mô tô có thuyền img_022 | xe đạp → ngoài phạm vi; tuk-tuk, xe ba bánh img_004, img_035 → ngoài phạm vi |
| `person` | Người thật **không** đang điều khiển / ngồi trên xe máy, xe đạp trong giao thông: người đi bộ, đứng, ngồi ở quán / vỉa hè, người ngồi trên xe hai bánh **đang đỗ**, người trên xe hở khác (tuk-tuk, xích lô, thùng bán tải). | người qua đường img_014, img_034; khách ngồi quán img_001; người gánh hàng img_003; tài xế tuk-tuk img_004; người ngồi trên xe đỗ bên phải img_010 | người đi xe máy / xe đạp trong dòng xe → `rider`; người trong **xe mui trần** img_019, **xe buýt** img_009, cabin kín → không gán (R-06); tượng img_017, ma-nơ-canh img_033 → không gán |
| `rider` | Người đang **điều khiển hoặc ngồi trên xe máy / xe đạp đang tham gia giao thông** (đang chạy, hoặc dừng tạm trong dòng xe: đèn đỏ, kẹt xe, dừng bên đường mà vẫn ngồi trên yên, xe không dựng chân chống). | 2 người trên xe máy img_026; dòng xe img_021; người đi xe đạp img_006; người lái + người ngồi sau xe tay ga trái img_010 | người ngồi trên xe **đã đỗ** (chân chống, trong bãi / hàng xe đỗ) → `person`; người đứng cạnh xe → `person` |
| `ignore_region` | Vùng **không phải vật**: cụm vật trong phạm vi không tách được từng cái (quá nhỏ / quá dày / mờ). Bỏ sót trong vùng này không tính lỗi. | đám xe máy xa img_021; bãi xe dày img_030 | **không** dùng cho vật ngoài phạm vi (bus, tuk-tuk) hay vật giả (tượng) |

**Vật không thuộc lớp nào** (R-02): xe buýt / minibus / xe khách, tuk-tuk, auto rickshaw, xích lô, xe đạp,
xe đẩy, động vật, cọc tiêu… → **không vẽ box, không vẽ `ignore_region`**. Người trên các xe này xử lý theo R-06 và R-19.

### 2.2 Thuộc tính

Áp dụng cho mọi lớp trừ `ignore_region`.

#### `occluded_level` — phần vật bị **vật khác** che (ước theo diện tích)

Tưởng tượng toàn bộ hình dáng vật nếu không bị che (trong khung ảnh), ước tỉ lệ diện tích bị che.

| Mức | Bị che | Dấu hiệu nhận nhanh |
|---|---|---|
| `0` | < 10 % | thấy gần trọn vật |
| `1` | 10 % – < 40 % | mất chân, bánh xe, một góc |
| `2` | 40 % – < 80 % | mất khoảng nửa vật trở lên, vẫn đoán được hình dáng |
| `3` | ≥ 80 % | chỉ thấy một mảnh (đầu, nóc xe) nhưng chắc chắn là vật đó |

- **`rider` và chiếc xe họ đang đi không tính là che nhau.**
- Hàng rào lưới / song sắt mảnh nhìn xuyên được (img_018): tối đa mức `1` nếu đường viền vật vẫn rõ.
- Phần bị mép ảnh cắt **không** tính vào che khuất (dùng `truncated`).
- Không dùng `3` với nghĩa “không rõ”. Sát ngưỡng → chọn mức gần nhất và tick `can_review`.

#### `truncated` — vật bị mép ảnh cắt

| Giá trị | Khi nào |
|---|---|
| `yes` | box chạm mép ảnh (≤ 3 px) **và** vật rõ ràng còn tiếp ra ngoài khung (xe đỏ, SUV bạc img_015; người áo đỏ img_016). |
| `no` | các trường hợp còn lại. |

#### `can_review` — tick khi

1. Không chắc vật có thật / trong phạm vi (xe máy độ nghệ thuật img_011).
2. Không chắc lớp sau khi áp dụng R-16 (xe) hoặc R-19 (`person` / `rider`).
3. `occluded_level` sát ngưỡng.
4. Vật mờ chuyển động, không chắc biên (R-15).

Ca đã có quy tắc rõ (tượng, xe trên xe tải, người trong xe buýt…) → **không** tick, chỉ cần không gán.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Vật thật | Chỉ gán vật **thật, cỡ thật, có mặt trong cảnh**, chụp trực tiếp (không qua gương / kính phản chiếu / ảnh in). |
| **R-02** Ngoài phạm vi | Bus, minibus, tuk-tuk, xe ba bánh, xích lô, xe đạp, xe đẩy, động vật → **không gán gì**. |
| **R-03** Người trên xe hai bánh | **Tách box**: mỗi người một box `rider` (hoặc `person` nếu xe đang đỗ, R-19); chiếc xe máy một box `motorcycle`. Xe đạp không có box. Không bao giờ gộp người và xe. |
| **R-04** Phạm vi box xe máy | Bánh trước tới bánh sau, từ mặt đường tới điểm cao nhất của **thân xe gắn cố định** (gương, kính chắn gió, tay lái, giỏ, baga, thùng / thuyền gắn cố định). **Không** bao người, **không** bao hàng chở thò ra ngoài thân xe (ống nhôm img_029). |
| **R-05** Phạm vi box người | Bao toàn bộ phần nhìn thấy của cơ thể kèm thứ **đang mặc / đội / đeo** (mũ bảo hiểm, nón lá, ba lô, túi đeo). **Không** bao vật cầm / gánh thò xa khỏi người (quang gánh img_003, ống nhôm img_029, ô dù). Áp dụng cho cả `person` và `rider`. |
| **R-06** Người trong / trên xe bốn bánh và xe khách | **Không gán** người ngồi trong: **xe buýt / minibus** (img_009), **xe mui trần** (tài xế xe Dangermouse img_019), ô tô / van / tải có cabin kín. **Gán `person`** cho người trên xe hở còn lại: tuk-tuk (img_004, img_016), xích lô (img_023), thùng bán tải. |
| **R-07** Ôm sát | Box ôm sát **phần nhìn thấy** (modal box), sai lệch ≤ 2 px mỗi cạnh. Không đoán phần bị che. Vật bị che tách hai mảnh → **một** box bao cả hai. |
| **R-08** Che khuất | Gán `occluded_level` theo mục 2.2. Vẫn vẽ box nếu thỏa R-10 và nhận ra lớp. |
| **R-09** Cắt mép | Gán `truncated=yes` theo mục 2.2. Box dừng ở mép ảnh. Phần còn lại không đủ nhận ra lớp → không gán. |
| **R-10** Vật nhỏ | Box cao < **20 px** → không vẽ box riêng. Cụm ≥ 3 vật nhỏ trong phạm vi → gộp `ignore_region` (R-11). Vật ≥ 20 px phải gán dù ở xa. |
| **R-11** Đám đông / `ignore_region` | Vẽ `ignore_region` khi cụm ≥ 3 vật trong phạm vi **không tách được biên từng vật** hoặc phần lớn < 20 px. Vật tiền cảnh tách được vẫn gán riêng. Người trong đám đông “tách được” khi thấy rõ **ít nhất đầu và vai**. Vùng ignore **không phủ quá 50 %** diện tích bất kỳ box nào đã gán. |
| **R-12** Vật không thật | Tượng, mô hình, đồ chơi, ma-nơ-canh, linh vật nhồi bông, người / xe trên biển quảng cáo, poster, tranh tường, màn hình, phản chiếu trong gương / kính / vũng nước → **không gán, không ignore**. |
| **R-13** Xe chủ (ego) | Phần của xe người chụp đang ngồi (gương, khung cửa, capo, táp-lô) → không gán (img_020). Xe ngay trước camera mà người chụp đứng ngoài (img_012) **không** phải xe chủ → gán bình thường. |
| **R-14** Xe trên xe tải / xe khác | **Không gán** xe đang được chở trên xe tải, rơ-moóc hoặc trên nóc xe khác (7 ô tô trên xe chở xe img_002; xe vàng trên nóc img_024). Xe chở gán một box `truck` bao **cabin + khung chở + rơ-moóc gắn liền**, không bao phần xe được chở nhô cao hơn khung. Không đổi `occluded_level` của xe chở vì hàng trên nó. |
| **R-15** Mờ chuyển động | Mờ nhưng **xác định được biên** (sai ≤ 10 px) → gán, tick `can_review`. Vệt mờ bán trong suốt, không xác định được biên → `ignore_region` (img_014, img_015). |
| **R-16** `car` / `van` / `truck` | Hỏi theo thứ tự: (1) cabin tách riêng khỏi thùng hàng (kể cả có nắp thùng)? → `truck`. (2) thân hộp liền khối, mui cao, cỡ Hiace / Transit? → `van`. (3) còn lại, chở người ≤ 9 chỗ → `car`. Xe khách có cửa hành khách riêng → ngoài phạm vi. |
| **R-17** Ảnh không có vật | Vẫn giữ ảnh trong COCO với 0 annotation (img_017, img_020, img_027, img_033, img_038). |
| **R-18** Phần lắp thêm | Thùng / thuyền **gắn cố định** vào xe máy là một phần của `motorcycle` (img_022, img_025). Rơ-moóc gắn vào đầu kéo là một phần của `truck` (img_002). |
| **R-19** `rider` hay `person` | Người trên **xe máy / xe đạp**: xe **đang chạy** hoặc **dừng tạm trong dòng xe** (đèn đỏ, kẹt xe, chống chân xuống đường nhưng vẫn ngồi trên yên, xe không dựng chân chống) → `rider`. Xe **đã đỗ** (dựng chân chống, nằm trong bãi / hàng xe đỗ trên vỉa hè) → người ngồi trên đó là `person`. Người đứng / dắt xe → `person`. Không chắc → `rider` + `can_review`. |

### Ví dụ áp dụng R-03, R-04, R-05, R-19

| Ảnh | Tình huống | Số box dự kiến |
|---|---|---|
| img_010 | Xe tay ga trái chở 2 người dừng bên đường mua hàng; cô gái bên phải ngồi nghiêng trên xe tay ga đã dựng chân chống | trái: 2 `rider` + 1 `motorcycle`; phải: 1 `person` + 1 `motorcycle` |
| img_026 | 2 người trên xe máy đỏ đang chạy, tranh tường phía sau | 2 `rider` + 1 `motorcycle`; tranh tường không gán |
| img_029 | Người đi xe máy vác ống nhôm, người đi xe đạp đỡ đầu kia, cả hai dừng ở vạch qua đường | 2 `rider` + 1 `motorcycle` (không bao ống); xe đạp không box |
| img_008 | Tài xế xe ôm áo cam đang chạy bên phải; người áo cam đứng cạnh hàng xe đỗ bên trái | phải: 1 `rider` + 1 `motorcycle`; trái: 1 `person`; các xe đỗ: `motorcycle` |
| img_005 | Cận cảnh nhiều người đội mũ trên xe máy dừng đèn đỏ | mỗi người 1 `rider`; `motorcycle` chỉ khi thấy thân xe hoặc bánh |

---

## 4. Ca khó

Cột **Loại**: `QT` = quyết định theo quy tắc, không cần hỏi; `HỎI` = gán theo quyết định tạm và tick `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Loại | Lý do |
|---|---|---|---|---|---|
| img_001 | Bán tải Suzuki Carry chở ống sắt; khách ngồi quán bị xe che; poster hot dog | R-16, R-08, R-12 | 1 `truck` `truncated=yes`; mỗi người → `person` (ông lão bị xe che `occluded_level` 1–2); poster không gán | QT | khách ngồi quán, không trên xe |
| img_002 | Xe đầu kéo chở 7 ô tô; công nhân đứng cuối xe | R-14, R-18 | 1 `truck` (cabin + khung + rơ-moóc); **không** gán 7 ô tô được chở; công nhân → `person` | QT | xe trên xe tải không gán |
| img_003 | Người gánh quang gánh, người dắt xe đạp, xe máy đỗ | R-05, R-02, R-19 | người gánh và người dắt xe → `person` (không bao thúng); xe đạp không gán; xe máy đỗ → `motorcycle` | QT | dắt xe ≠ ngồi trên xe |
| img_004 | 2 tuk-tuk có tài xế ngồi trong | R-02, R-06 | không gán tuk-tuk; 2 tài xế → `person` (`occluded_level` 2–3) | QT | tuk-tuk là xe hở, không phải xe máy / xe đạp |
| img_007 | Kho: Hilux thùng kín, Isuzu ELF, người đứng khuất giữa hai xe | R-16, R-08 | 2 `truck`; người → `person` `occluded_level=2` | QT | Hilux có cabin tách thùng |
| img_008 | Bãi xe máy đêm, xe ôm, songthaew phía xa | R-19, R-11, R-16 | xe ôm đang chạy → `rider`; người đứng → `person`; cụm người xa → `ignore_region`; songthaew → `truck` + `can_review` | HỎI | songthaew là bán tải cải tạo chở khách |
| img_009 | Minibus 29 chỗ, hành khách qua cửa kính; người đi xe máy xa | R-02, R-06, R-19 | không gán minibus và hành khách; người đi xe máy → `rider` nếu ≥ 20 px | QT | bus ngoài phạm vi, người trong bus không gán |
| img_011 | Tượng Predator; xe máy độ phế liệu trưng bày; xe cứu thương | R-12, R-01 | tượng không gán; khách lễ hội → `person`; xe cứu thương → `van`; xe máy độ → `motorcycle` + `can_review` | HỎI | chưa chắc xe độ còn chạy được |
| img_012 | Chụp từ sau thùng bán tải chở hàng | R-13, R-09 | bán tải → `truck` `truncated=yes`; Mazda → `car`; xe tải phía trước → `truck` | QT | người chụp đứng ngoài xe → không phải xe chủ |
| img_013 | Đêm, tượng rồng vàng, 2 người đi bộ, xe máy đỗ, ô tô xa | R-12, R-10 | tượng không gán; 2 `person`; xe máy đỗ → `motorcycle`; ô tô xa gán nếu ≥ 20 px | QT | |
| img_014 | Hai người qua đường rõ nét, xe máy mờ hai bên | R-15 | 2 `person`; vệt người + xe mờ → `ignore_region` | HỎI | khó xác định biên vật mờ |
| img_015 | Ô tô mờ phía sau, 2 xe tiền cảnh cắt mép | R-15, R-09 | xe đỏ và SUV bạc → `car` `truncated=yes`; xe đỗ rõ → `car`; vệt mờ → `ignore_region` | QT | |
| img_016 | Tuk-tuk, Vespa chở hàng, Hiace, bán tải, người cắt mép | R-02, R-04, R-06, R-16, R-19 | người lái Vespa → `rider`, Vespa → `motorcycle` (không bao cuộn hàng); Hiace → `van`; bán tải → `truck`; người áo đỏ → `person` `truncated=yes`; người trên tuk-tuk → `person` | QT | |
| img_017 | Tượng người cưỡi ngựa | R-12, R-17 | 0 box | QT | tượng |
| img_018 | Người trên biển bầu cử; xe sau hàng rào song sắt | R-12, R-08, R-16 | không gán người trên biển; các `car` và 1 `truck` (bán tải), `occluded_level=1` | QT | hàng rào mảnh → tối đa mức 1 |
| img_019 | Xe Dangermouse mui trần có tài xế; linh vật chuột; khán giả sau rào | R-06, R-12, R-11 | xe → `car`; **không** gán tài xế (xe mui trần); không gán linh vật; khán giả tách được → `person`, phần sau → `ignore_region` | QT | người trong xe mui trần không gán |
| img_020 | Xe phía sau nhìn trong gương chiếu hậu xe chủ | R-12, R-13, R-17 | 0 box | QT | phản chiếu + xe chủ |
| img_021 | Ngã tư hàng trăm xe máy | R-03, R-19, R-11 | cặp `rider` + `motorcycle` tiền cảnh gán riêng; phần xa → `ignore_region`, không phủ quá 50 % box đã gán | QT | |
| img_022 | Mô tô có thuyền đỗ trên đường rừng, người ngồi phía thuyền | R-18, R-19 | 1 `motorcycle` bao cả thuyền; người → `rider` + `can_review` nếu ≥ 20 px | HỎI | không rõ xe đang chạy hay đỗ |
| img_023 | Ô tô đen, xích lô có người đạp, xe máy, người ngồi lề | R-06, R-19, R-02 | ô tô → `car`; người đạp xích lô → `person`; người đi xe máy → `rider`; người ngồi lề → `person` | QT | xích lô không phải xe đạp hai bánh |
| img_024 | Bảo tàng xe: Pacer đen, xe vàng trên nóc | R-14 | xe đen và xe trưng bày → `car`; xe vàng trên nóc → **không gán** | QT | xe trên xe khác không gán |
| img_027 | Bia mộ hình bán tải bằng đá | R-12, R-17 | 0 box | QT | |
| img_028 | Bán tải thật chở tượng ngựa, SUV đỏ phía sau | R-12, R-16 | 1 `truck` + 1 `car` (`occluded_level=1`); không gán tượng ngựa | QT | |
| img_030 | Bãi gửi xe máy dày, bảo vệ, poster | R-11, R-12, R-19 | xe hàng đầu → `motorcycle`; phía sau → `ignore_region`; bảo vệ đi bộ → `person`; poster không gán | QT | |
| img_031 | Đêm mưa, tranh hoạt hình, ô tô, xe buýt, xe máy | R-12, R-02, R-19 | tranh và buýt không gán; ô tô → `car`; người đi xe máy → `rider`; người đi bộ → `person` (≥ 20 px) | QT | |
| img_032 | Hàng ~30 người chờ buýt, ô tô xanh cắt mép | R-11, R-09 | người thấy rõ đầu + vai → `person`; người khuất phía sau → `ignore_region`; ô tô → `car` `truncated=yes` | QT | ngưỡng “tách được” ở R-11 |
| img_033 | Ma-nơ-canh, ảnh in, phản chiếu xe | R-12, R-17 | 0 box | QT | |
| img_035 | Auto rickshaw chở hàng, xe máy, ô tô | R-02, R-19 | rickshaw không gán; người lái xe máy → `rider` + `motorcycle`; ô tô → `car` | QT | |
| img_037 | Bán tải gắn nắp thùng, 2 xe buýt, người đi bộ | R-16, R-02 | bán tải → `truck`; buýt không gán; người → `person` | QT | |
| img_038 | Tủ kính ô tô đồ chơi | R-12, R-17 | 0 box | QT | |
| img_040 | Người đội mũ cận cảnh, chỉ thấy gương xe máy; taxi, tuk-tuk, SUV, buýt | R-19, R-03, R-06 | người → `rider` `truncated=yes`; không vẽ `motorcycle` (chỉ thấy gương); taxi, SUV → `car`; tài xế taxi không gán | HỎI | xe máy hầu như không thấy |

---

## 5. QC và phiên bản

### Ngưỡng QC (lấy từ `taxonomy.json` → `rules`, không sửa)

| Ngưỡng | Giá trị | Dùng để |
|---|---|---|
| `min_box_height_px` | 20 | R-10 |
| `match_iou` | 0.5 | ghép cặp box giữa hai người gán |
| `iou_thr_large` / `iou_thr_small` | 0.85 / 0.70 | IoU tối thiểu cho box lớn (≥ `large_px` = 50 px) / nhỏ |
| `iaa_iou` / `iaa_class` | 0.85 / 0.95 | mục tiêu đồng thuận box / lớp |
| `error_rate_img_max` | 0.02 | > 2 % ảnh lỗi → Rework |

### Ai quyết ca mơ hồ

1. Người gán: tick `can_review` hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv` (`image,annotator,region,note`), **không** tự đổi quy tắc.
2. Nhóm chốt ở Pha 5, mỗi ca một dòng trong `calibration_decisions.csv` kèm mã `R-xx`.
3. Thay đổi làm đổi cách chọn vật / lớp / box → `semantic`, tăng phiên bản, kiểm lại cả lô.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | Hợp nhất các bản cá nhân. So với v0.1-054: (1) thêm lớp **`rider`** cho người trên xe máy / xe đạp tham gia giao thông (R-19); (2) `person` **không** tính người trong xe mui trần và xe buýt (R-06); (3) **không gán xe được chở** trên xe tải / xe khác (R-14); (4) ngưỡng “tách được” trong đám đông = thấy đầu + vai (R-11) | img_010, img_026 (rider vs person); img_019 (mui trần); img_009 (bus); img_002, img_024 (xe trên xe); img_032 (đám đông) |

---

## Phụ lục — Bảng rà nhanh 40 ảnh

“—” = không có vật cần gán.

| Ảnh | Nội dung chính | Lớp dự kiến | Quy tắc cần chú ý |
|---|---|---|---|
| img_001 | Quán vỉa hè, bán tải chở ống, khách ngồi | `truck`, `person` | R-08, R-12 |
| img_002 | Xe đầu kéo chở 7 ô tô | `truck`, `person` | R-14 |
| img_003 | Người gánh mít, xe đạp, xe máy | `person`, `rider`, `motorcycle`, `car` (xa) | R-05, R-19 |
| img_004 | 2 tuk-tuk có tài xế | `person` | R-02, R-06 |
| img_005 | Cận cảnh người trên xe máy, van, ô tô | `rider`, `motorcycle`, `van`, `car` | R-03, R-09 |
| img_006 | Dòng xe máy + xe đạp nhìn từ sau, taxi | `rider`, `motorcycle`, `car` | R-02, R-19 |
| img_007 | Kho bốc hàng, 2 xe tải | `truck`, `person` | R-16 |
| img_008 | Bãi xe máy đêm, xe ôm | `motorcycle`, `rider`, `person`, `ignore_region` | R-11, R-19 |
| img_009 | Minibus, xe máy xa | `motorcycle`, `rider`, `person` | R-02, R-06 |
| img_010 | Người bán chuối, xe tay ga | `rider`, `person`, `motorcycle` | R-19 |
| img_011 | Tượng Predator, xe máy độ, lễ hội | `person`, `van`, (`motorcycle`?) | R-12, `can_review` |
| img_012 | Thùng bán tải chở hàng, Mazda | `truck`, `car` | R-09, R-13 |
| img_013 | Đêm, tượng rồng, người đi bộ | `person`, `motorcycle`, `car` | R-12, R-10 |
| img_014 | Người qua đường, xe mờ | `person`, `ignore_region` | R-15 |
| img_015 | Ô tô mưa, xe mờ phía sau | `car`, `ignore_region` | R-09, R-15 |
| img_016 | Tuk-tuk, Vespa, Hiace | `motorcycle`, `rider`, `person`, `van`, `truck`, `car` | R-02, R-04, R-16 |
| img_017 | Tượng người cưỡi ngựa | — | R-12, R-17 |
| img_018 | Biển bầu cử, bãi xe sau hàng rào | `car`, `truck` | R-12, R-08 |
| img_019 | Xe Dangermouse mui trần, đám đông | `car`, `person`, `ignore_region` | R-06, R-11 |
| img_020 | Gương chiếu hậu xe chủ | — | R-12, R-13 |
| img_021 | Ngã tư đông xe máy | `rider`, `motorcycle`, `car`, `ignore_region` | R-03, R-11 |
| img_022 | Mô tô có thuyền | `motorcycle`, `rider` | R-18, R-19 |
| img_023 | Ô tô đen, xích lô, xe máy | `car`, `person`, `rider`, `motorcycle` | R-06, R-19 |
| img_024 | Bảo tàng xe, xe vàng trên nóc | `car` | R-14 |
| img_025 | Phố biển, người đi bộ, xe máy kéo xe hàng | `person`, `rider`, `car`, `truck`, `motorcycle`, `ignore_region` | R-10, R-18 |
| img_026 | 2 người trên xe máy, tranh tường | `rider`, `motorcycle` | R-03, R-12 |
| img_027 | Bia mộ hình bán tải | — | R-12, R-17 |
| img_028 | Bán tải chở tượng ngựa, SUV | `truck`, `car` | R-12 |
| img_029 | Xe máy chở ống nhôm, xe đạp | `rider`, `motorcycle`, `person` | R-04, R-05, R-19 |
| img_030 | Bãi gửi xe máy dày, poster | `motorcycle`, `person`, `ignore_region` | R-11, R-12 |
| img_031 | Phố đêm mưa | `car`, `motorcycle`, `rider`, `person` | R-02, R-12 |
| img_032 | Hàng người chờ buýt | `person`, `car`, `ignore_region` | R-11, R-09 |
| img_033 | Ma-nơ-canh, phản chiếu | — | R-12, R-17 |
| img_034 | Người đi bộ qua đường, ô tô, xe máy | `person`, `rider`, `car`, `motorcycle` | R-19, R-09 |
| img_035 | Auto rickshaw, xe máy | `motorcycle`, `rider`, `person`, `car` | R-02 |
| img_036 | Xe tải nhẹ Veryca, xe máy đỗ | `truck`, `motorcycle`, `car` (xa) | R-16 |
| img_037 | Bán tải gắn nắp thùng, xe buýt | `truck`, `person` | R-16, R-02 |
| img_038 | Tủ kính ô tô đồ chơi | — | R-12, R-17 |
| img_039 | Phố Huế, xe máy, người đi bộ | `person`, `rider`, `motorcycle`, `car` (xa) | R-19, R-10 |
| img_040 | Người đội mũ cận cảnh, taxi, tuk-tuk | `rider`, `car` | R-19, R-06 |
