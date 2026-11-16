# Guideline gán nhãn — v0.1 (bản cá nhân) — <mã thành viên>

> Bản cá nhân viết ở Pha 1 sau khi xem cả 40 ảnh trong `data/images/`. Mỗi quy tắc có mã `R-xx`
> để dẫn lại trong nhật ký QC. Mỗi ví dụ ghi ID ảnh để người đọc mở lại được.

- Phiên bản: v0.1-<mã>
- Ngày: 2026-11-16
- Người viết: <mã> · Người duyệt: (chờ họp nhóm Pha 2)
- Dựa trên: `guideline/guideline_v0_starter.md`, `submission/taxonomy.json` (v0)

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện **mô hình phát hiện vật trên ảnh 2D** (bounding box) cho **người và phương tiện
cơ giới đường bộ** trong ảnh đường phố: camera giao thông, hỗ trợ lái xe.

Sai ở đâu thì nguy hiểm nhất:

| Lỗi | Hậu quả | Quy tắc ưu tiên |
|---|---|---|
| **Bỏ sót người** (người đi bộ, người ngồi trên xe máy) | mô hình không thấy người → nguy cơ va chạm | gán **mọi** `person` thật đủ cỡ (R-03, R-05, R-10) |
| Gán vật **không thật** (tượng, ảnh in, phản chiếu, đồ chơi) | mô hình báo động giả, học sai đặc trưng | R-12 |
| Gộp người và xe vào một box | mô hình không đếm được số người, box lệch | R-03 |
| Nhầm lớp `car` / `van` / `truck` | ảnh hưởng ít hơn bỏ sót, nhưng làm giảm IAA lớp | R-16 |

Nguyên tắc chung: **khi phân vân giữa “gán” và “không gán” một người thật → gán và tick `can_review`.**
Bỏ sót người nguy hiểm hơn thừa một box.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

Mỗi vật thuộc **đúng một** lớp. Chỉ gán vật **thật, kích thước thật, đang có mặt trong cảnh** (R-01).

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người ≤ 9 chỗ, thân liền khối (sedan, hatchback, SUV, crossover, wagon, taxi, MPV), kể cả xe độ thân vỏ vẫn chạy được. | sedan đen img_023; taxi xanh img_040; Mazda xám img_012; xe trưng bày trong bảo tàng img_024; xe Dangermouse diễu hành img_019 | MPV thân vuông cao như Hiace → `van`; xe đồ chơi / mô hình img_038 → **không gán** (R-12) |
| `truck` | Xe chở hàng có **cabin tách riêng khỏi thùng hàng** (thùng hở, thùng kín, thùng bạt), gồm cả **bán tải (pickup)**, xe tải nhẹ, xe đầu kéo + rơ-moóc. | Suzuki Carry img_001; bán tải img_012, img_028; Veryca thùng bạt img_036; bán tải gắn nắp thùng img_037; Isuzu ELF và Hilux thùng kín img_007; xe chở ô tô img_002 | thân xe liền một khối với khoang hàng → `van`; tượng bán tải bằng đá img_027 → **không gán** |
| `van` | Xe thân hộp **liền một khối** từ đầu tới đuôi (cabin và khoang chở chung một vỏ), mui cao, thường có cửa trượt; cỡ Toyota Hiace / Ford Transit. | Hiace bạc img_005; Hiace trắng img_016; xe cứu thương thân hộp img_011 | xe khách có cửa lên xuống riêng cho hành khách (minibus img_009) → **ngoài phạm vi** (R-02); SUV → `car` |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, xe tay ga, mô tô, xe đạp điện có yên kiểu xe máy), kể cả bộ **thùng / thuyền gắn cố định** bên hông. | tay ga img_010; xe số img_006; Vespa img_016; mô tô có thuyền img_022; xe đẩy hàng gắn xe máy img_025 | xe đạp img_006, img_029 → **ngoài phạm vi**; tuk-tuk / xe ba bánh img_004, img_035 → **ngoài phạm vi** |
| `person` | Một người thật bất kỳ: đi bộ, đứng, ngồi, ngồi trên xe máy / xe đạp / xích lô / tuk-tuk, kể cả trẻ em. | người qua đường img_014, img_034; người lái + người ngồi sau img_026; người bán hàng gánh img_003 | tượng img_017; ma-nơ-canh img_033; người trên biển quảng cáo img_018; người trong cabin kín → **không gán** (R-06) |
| `ignore_region` | Vùng **không phải vật**: nơi có nhiều vật trong phạm vi nhưng không tách được từng cái (quá nhỏ / quá dày / mờ). Bỏ sót trong vùng này không tính lỗi. | đám xe máy phía xa img_021; bãi xe dày img_030 | **không** dùng để che vật ngoài phạm vi (bus, tuk-tuk) hay vật giả (tượng) — những thứ đó chỉ cần không gán |

**Vật không thuộc lớp nào** (R-02): xe buýt / minibus / xe khách, tuk-tuk, xe ba bánh chở khách (auto rickshaw),
xích lô, xe đạp, xe đẩy tay, động vật, cọc tiêu… → **không vẽ box, không vẽ ignore_region**.
Người ngồi trên các xe này **vẫn gán `person`** nếu thỏa R-06. Đề xuất cho họp nhóm: cân nhắc thêm lớp `bus`
và `three_wheeler` vì xuất hiện nhiều (img_004, img_009, img_016, img_031, img_035, img_037, img_040).

### 2.2 Thuộc tính

Áp dụng cho mọi lớp trừ `ignore_region`.

#### `occluded_level` — phần vật bị **vật khác** che (ước theo diện tích)

Cách đo: tưởng tượng toàn bộ hình dáng vật nếu không bị che (chỉ trong khung ảnh), ước tỉ lệ diện tích bị che.

| Mức | Bị che | Dấu hiệu nhận nhanh |
|---|---|---|
| `0` | < 10 % | thấy gần trọn vật |
| `1` | 10 % – < 40 % | mất một phần nhỏ: chân, bánh xe, một góc |
| `2` | 40 % – < 80 % | mất khoảng nửa vật trở lên, vẫn đoán được hình dáng |
| `3` | ≥ 80 % | chỉ thấy một mảnh (đầu, nóc xe) nhưng vẫn chắc là vật đó |

Quy ước bổ sung:
- **Người lái / người ngồi trên xe và chính chiếc xe đó không tính là che nhau** (R-03). Ví dụ img_026: người
  ngồi che yên xe máy → xe vẫn `0` nếu không có vật khác che.
- **Hàng rào lưới / song sắt mảnh** nhìn xuyên được (img_018): tối đa mức `1` nếu đường viền vật vẫn rõ.
- Phần bị **mép ảnh** cắt **không** tính vào che khuất (đã có `truncated`).
- Không dùng `3` với nghĩa “không rõ”. Không chắc mức nào → chọn mức gần nhất và tick `can_review`.

#### `truncated` — vật bị mép ảnh cắt

| Giá trị | Khi nào |
|---|---|
| `yes` | box chạm mép ảnh (≤ 3 px) **và** vật rõ ràng còn tiếp tục ra ngoài khung. Ví dụ: ô tô đỏ và SUV bạc img_015; người phụ nữ áo đỏ img_016. |
| `no` | các trường hợp còn lại, kể cả vật nằm sát mép nhưng đã trọn vẹn. |

#### `can_review` — tick khi

1. Không chắc vật có thật / có trong phạm vi không (ví dụ xe độ nghệ thuật img_011, xe vàng trên nóc img_024).
2. Không chắc lớp (`car`/`van`/`truck`) sau khi đã áp dụng R-16.
3. Không chắc mức `occluded_level` vì nằm sát ngưỡng.
4. Vật mờ chuyển động, không chắc biên (R-15).

**Không** tick `can_review` cho ca đã có quy tắc rõ (ví dụ tượng → không gán, không cần hỏi).

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Vật thật | Chỉ gán vật **thật, cỡ thật, có mặt trong cảnh** mà camera chụp trực tiếp (không qua gương / kính phản chiếu / ảnh in). |
| **R-02** Ngoài phạm vi | Bus, minibus, tuk-tuk, xe ba bánh, xích lô, xe đạp, xe đẩy, động vật → **không gán gì**. Người ngồi trên đó vẫn là `person`. |
| **R-03** Người đi xe máy | **Tách box**: mỗi người là một box `person`; chiếc xe là một box `motorcycle`. Không bao giờ gộp người và xe vào một box. Người và xe của họ không tính là che nhau. |
| **R-04** Phạm vi box xe máy | Bao bánh trước tới bánh sau, từ mặt đường tới điểm cao nhất của **thân xe gắn cố định** (gương, kính chắn gió, tay lái, giỏ, baga, thùng/thuyền gắn cố định). **Không** bao người, **không** bao hàng hóa chở thêm thò ra ngoài thân xe (ống nhôm img_029, bao hàng). |
| **R-05** Phạm vi box người | Bao toàn bộ phần nhìn thấy của cơ thể kèm thứ **đang mặc / đội / đeo sát người** (mũ bảo hiểm, nón lá, ba lô, túi đeo vai). **Không** bao vật cầm hoặc gánh thò xa khỏi người (quang gánh và thúng img_003, ống nhôm img_029, ô dù). |
| **R-06** Người trong xe | Người trong **cabin kín** nhìn qua kính (ô tô, van, tải, bus) → **không gán**. Người trên xe **hở** (xe máy, xe đạp, xích lô, tuk-tuk, thùng bán tải, xe mui trần img_019) → gán `person` nếu thỏa R-10. |
| **R-07** Ôm sát | Box ôm sát **phần nhìn thấy** (modal box), sai lệch ≤ 2 px mỗi cạnh. Không đoán phần bị che. Vật bị che tách làm hai mảnh → **một** box bao cả hai mảnh. |
| **R-08** Che khuất | Gán `occluded_level` theo bảng ở mục 2.2. Vẫn vẽ box nếu thỏa R-10 và nhận ra được lớp. |
| **R-09** Cắt mép | Gán `truncated=yes` theo mục 2.2. Box dừng ở mép ảnh. Nếu phần còn lại không đủ nhận ra lớp → không gán. |
| **R-10** Vật nhỏ | Box có chiều cao < **20 px** (`min_box_height_px`) → **không vẽ box riêng**. Nếu chỗ đó có từ 3 vật nhỏ trở lên trong phạm vi → gộp vào `ignore_region` (R-11). Vật ≥ 20 px phải gán dù ở xa. |
| **R-11** Đám đông / `ignore_region` | Vẽ `ignore_region` (một hình chữ nhật) khi một cụm ≥ 3 vật trong phạm vi mà **không tách được biên từng vật** hoặc phần lớn < 20 px. Vật ở tiền cảnh tách được thì vẫn gán riêng. Vùng ignore **không được phủ quá 50 % diện tích** của bất kỳ box nào đã gán (script coi box phủ ≥ 50 % là bị bỏ qua khi so sánh). |
| **R-12** Vật không thật | Tượng, mô hình, đồ chơi, ma-nơ-canh, người/xe trên biển quảng cáo, poster, tranh tường, màn hình, phản chiếu trong gương / kính / vũng nước → **không gán, không ignore**. |
| **R-13** Xe chủ (ego) | Phần của xe mà người chụp đang ngồi (gương, khung cửa, nắp capo, táp-lô) → không gán (img_020). |
| **R-14** Xe chở xe / chở hàng | Xe chở là một box `truck` bao **cabin + khung chở + rơ-moóc gắn liền**. Mỗi xe thật được chở trên đó gán **box riêng** theo lớp của nó, có `occluded_level` do khung che (img_002). |
| **R-15** Mờ chuyển động | Vật mờ nhưng **xác định được biên** (ước sai ≤ 10 px) → gán, tick `can_review`. Vật mờ thành vệt bán trong suốt, không xác định được biên → `ignore_region` (img_014, img_015). |
| **R-16** Phân biệt `car` / `van` / `truck` | Hỏi theo thứ tự: (1) Cabin tách riêng khỏi thùng hàng (kể cả có nắp thùng)? → `truck`. (2) Thân hộp liền khối, mui cao, cỡ Hiace/Transit? → `van`. (3) Còn lại là xe chở người ≤ 9 chỗ → `car`. Xe khách có cửa hành khách riêng → ngoài phạm vi. |
| **R-17** Ảnh không có vật | Vẫn giữ ảnh trong file COCO với 0 annotation (img_017, img_020, img_027, img_033, img_038). |
| **R-18** Vật lắp thêm | Thùng/thuyền **gắn cố định** vào xe máy là một phần của `motorcycle` (img_022, img_025). Rơ-moóc gắn vào đầu kéo là một phần của `truck` (img_002). |

### Thử quy tắc người đi xe máy (R-03, R-04, R-05) trên các ảnh khác nhau

| Ảnh | Tình huống | Dự đoán số box |
|---|---|---|
| img_026 | 2 người trên 1 xe máy đỏ, tranh tường phía sau | 2 `person` + 1 `motorcycle`; tranh tường không gán (R-12) |
| img_010 | xe tay ga bên trái chở 2 người đang dừng mua hàng | 2 `person` + 1 `motorcycle` cho xe đó |
| img_029 | người đi xe máy vác ống nhôm, người đi xe đạp đỡ đầu kia | xe máy: 1 `person` + 1 `motorcycle` (không bao ống); xe đạp: 1 `person`, không box cho xe đạp |
| img_022 | mô tô có thuyền, một người ngồi | 1 `motorcycle` bao cả thuyền (R-18) + 1 `person` nếu cao ≥ 20 px |
| img_005 | cận cảnh nhiều người đội mũ, xe gần như khuất | mỗi người 1 `person`; chỉ vẽ `motorcycle` khi thấy được thân xe hoặc bánh (không vẽ nếu chỉ thấy gương / tay lái) |

---

## 4. Ca khó

Cột **Loại**: `QT` = loại / quyết định theo quy tắc, không cần hỏi; `HỎI` = cần reviewer hoặc nhóm chốt (tick `can_review`).

| Ảnh | Tình huống | Rule ID | Quyết định | Loại | Lý do |
|---|---|---|---|---|---|
| img_002 | Xe đầu kéo chở 7 ô tô trên 2 tầng, công nhân đứng cuối xe | R-14, R-08 | 1 `truck` (cabin + khung + rơ-moóc); mỗi ô tô thấy được 1 `car` có `occluded_level` theo khung che; công nhân 1 `person` | QT | các xe được chở là ô tô thật, mô hình cần thấy chúng |
| img_017 | Tượng người cưỡi ngựa đội cọc tiêu, có con mòng biển | R-12, R-17 | 0 box, giữ ảnh trong COCO | QT | tượng không phải người thật |
| img_027 | Bia mộ hình xe bán tải bằng đá | R-12, R-17 | 0 box | QT | hình dáng giống `truck` nhưng không phải xe |
| img_028 | Bán tải thật chở tượng ngựa, SUV đỏ phía sau | R-12, R-16 | 1 `truck` + 1 `car` (`occluded_level=1`); không gán tượng ngựa | QT | xe thật dù chở vật trang trí |
| img_033 | Ma-nơ-canh, ảnh in người mẫu, phản chiếu xe trên kính tiệm | R-12, R-17 | 0 box | QT | không có vật thật nào chụp trực tiếp |
| img_038 | Tủ kính trưng bày ô tô đồ chơi | R-12, R-17 | 0 box | QT | đồ chơi, không phải cỡ thật |
| img_020 | Ô tô phía sau nhìn trong gương chiếu hậu của xe chủ | R-12, R-13, R-17 | 0 box | QT | phản chiếu + xe chủ |
| img_018 | Người trên biển bầu cử; ô tô và bán tải sau hàng rào song sắt; chụp qua kính | R-12, R-08, R-16 | không gán người trên biển; gán các `car` và 1 `truck` (bán tải), `occluded_level=1` do hàng rào | QT | hàng rào mảnh nhìn xuyên được → tối đa mức 1 |
| img_019 | Xe Dangermouse diễu hành có tài xế; hình chuột khổng lồ; đám đông khán giả sau rào | R-01, R-06, R-11 | 1 `car`, 1 `person` (tài xế, xe mui trần); không gán hình chuột; khán giả tiền cảnh tách được → `person`, khối phía sau → `ignore_region` | HỎI | xe độ thân vỏ vẫn chạy trên đường → coi là `car`; cần nhóm xác nhận |
| img_024 | Bảo tàng xe: Pacer wagon đen, trên nóc chở xe Pacer vàng thu nhỏ | R-01, R-12 | xe đen và các xe trưng bày khác → `car`; xe vàng trên nóc → không gán, tick vấn đề cấp ảnh | HỎI | xe vàng nhỏ hơn cỡ thật (mô hình?) — không chắc chạy được |
| img_011 | Tượng Predator khổng lồ; xe máy độ bằng phế liệu trưng bày; xe cứu thương | R-12, R-01 | tượng không gán; người đi lễ hội → `person`; xe cứu thương → `van`; xe máy độ → tạm gán `motorcycle` + `can_review` | HỎI | không chắc xe độ còn là phương tiện chạy được hay chỉ là tác phẩm |
| img_004 | 2 tuk-tuk có tài xế ngồi trong | R-02, R-06 | không gán tuk-tuk; 2 tài xế → `person` (`occluded_level` 2–3) | QT | tuk-tuk ngoài phạm vi, nhưng xe hở → người vẫn gán |
| img_035 | Auto rickshaw chở bao hàng cao; xe máy; ô tô trắng | R-02, R-03 | rickshaw không gán; người lái xe máy `person` + `motorcycle`; ô tô → `car` | QT | |
| img_009 | Minibus 29 chỗ, hành khách nhìn qua cửa kính | R-02, R-06 | không gán minibus, không gán hành khách; người đi xe máy xa → gán nếu ≥ 20 px | QT | xe khách có cửa hành khách riêng → ngoài phạm vi |
| img_008 | Bãi xe máy đêm, tài xế xe ôm áo cam, songthaew phía xa | R-03, R-11, R-16 | xe và người tiền cảnh gán riêng; cụm người xa → `ignore_region`; songthaew (bán tải có khoang khách) | HỎI | songthaew là bán tải cải tạo chở khách: `truck` hay ngoài phạm vi? |
| img_021 | Ngã tư hàng trăm xe máy | R-03, R-10, R-11 | khoảng 15–20 cặp người + xe ở tiền cảnh gán riêng; phần xa → 1–2 `ignore_region` không phủ quá 50 % box đã gán | QT | không tách được biên từng xe ở xa |
| img_030 | Bãi gửi xe máy dày đặc, bảo vệ áo xanh, poster người | R-11, R-12 | các xe hàng đầu → `motorcycle` (`occluded_level` 1–2); phía sau → `ignore_region`; bảo vệ → `person`; poster không gán | QT | |
| img_032 | Hàng người chờ xe buýt (~30 người), ô tô xanh cắt mép trái | R-10, R-11, R-09 | người tách được → `person`; người đứng chồng sau không thấy biên → gộp `ignore_region`; ô tô → `car`, `truncated=yes` | HỎI | cần thống nhất ngưỡng “tách được” cho đám đông đứng sát nhau |
| img_014 | Hai người qua đường rõ nét, xe máy mờ chuyển động hai bên | R-15 | 2 `person` rõ; vệt mờ bán trong suốt bên trái/phải → `ignore_region` | HỎI | khó xác định biên vật mờ |
| img_015 | Xe ô tô mờ chuyển động phía sau, 2 xe tiền cảnh cắt mép | R-15, R-09 | xe đỏ và SUV bạc → `car`, `truncated=yes`; xe đỗ rõ phía sau → `car`; vệt xe mờ → `ignore_region` | QT | |
| img_003 | Người gánh quang gánh, người dắt xe đạp, xe máy đỗ | R-05, R-02 | `person` không bao thúng gánh; xe đạp không gán; xe máy đỗ → `motorcycle` | QT | |
| img_007 | Kho bốc hàng: Hilux gắn thùng kín, Isuzu ELF, người đứng khuất giữa hai xe | R-16, R-08 | 2 `truck`; người → `person` `occluded_level=2` | QT | Hilux có cabin tách khỏi thùng → `truck`, không phải `van` |
| img_037 | Bán tải gắn nắp thùng cao kín; 2 xe buýt; người đi bộ | R-16, R-02 | bán tải → `truck`; buýt không gán; người → `person` | QT | nắp thùng không biến xe thành `van` vì cabin vẫn tách riêng |
| img_016 | Tuk-tuk, Vespa chở hàng, Hiace, bán tải sau tuk-tuk, người cắt mép | R-02, R-04, R-16, R-09 | Vespa → `motorcycle` (không bao cuộn hàng thò ra) + `person`; Hiace → `van`; bán tải → `truck` (`occluded_level` 2); người áo đỏ → `person` `truncated=yes`; tài xế/khách tuk-tuk → `person` | QT | |
| img_022 | Ảnh đen trắng, mô tô có thuyền | R-18 | 1 `motorcycle` bao cả thuyền + `person` nếu ≥ 20 px | QT | thuyền gắn cố định là một phần của xe |
| img_040 | Người đội mũ bảo hiểm cận cảnh; chỉ thấy gương xe máy; taxi, tuk-tuk, SUV, buýt | R-03, R-06, R-02 | người → `person` `truncated=yes`; **không** vẽ `motorcycle` (chỉ thấy gương); taxi và SUV → `car`; tài xế trong taxi không gán | HỎI | xe máy hầu như không thấy — cần nhóm chốt có gán hay không |
| img_013 | Đêm, tượng rồng vàng, 2 người đi bộ, xe máy đỗ, ô tô xa | R-12, R-10 | tượng không gán; 2 `person`; xe máy đỗ `motorcycle`; ô tô xa gán nếu ≥ 20 px | QT | |
| img_031 | Đêm mưa, bức vẽ hoạt hình trên tòa nhà, ô tô, xe buýt, xe máy | R-12, R-02, R-10 | tranh không gán; buýt không gán; các ô tô → `car`; người đi xe máy và người đi bộ → gán nếu ≥ 20 px | QT | |
| img_001 | Bán tải Suzuki Carry chở ống sắt; người ngồi quán bị xe che | R-16, R-08, R-12 | xe → `truck` `truncated=yes`; mỗi người ngồi → `person` (ông lão bị xe che `occluded_level` 1–2); poster hot dog không gán | QT | |
| img_012 | Chụp từ sau thùng bán tải chở hàng | R-13, R-09 | bán tải → `truck` `truncated=yes` (không phải xe chủ: người chụp đứng ngoài xe); Mazda → `car`; xe tải phía trước → `truck` | HỎI | phân biệt “xe chủ” với “xe ngay trước camera” |

---

## 5. QC và phiên bản

### Ngưỡng QC áp dụng (lấy từ `submission/taxonomy.json` → `rules`, không sửa)

| Ngưỡng | Giá trị | Dùng để |
|---|---|---|
| `min_box_height_px` | 20 | R-10: box < 20 px không gán riêng |
| `match_iou` | 0.5 | ghép cặp box giữa hai người gán |
| `iou_thr_large` / `iou_thr_small` | 0.85 / 0.70 | IoU tối thiểu cho box lớn (≥ `large_px` = 50 px) / box nhỏ |
| `iaa_iou` / `iaa_class` | 0.85 / 0.95 | mục tiêu đồng thuận box / lớp |
| `error_rate_img_max` | 0.02 | > 2 % ảnh lỗi → Rework |

### Ai quyết ca mơ hồ

1. Người gán: tick `can_review` hoặc ghi vấn đề cấp ảnh (`image,annotator,region,note`), **không** tự đổi quy tắc.
2. Nhóm chốt tại họp Pha 2 (trước khi gán) hoặc Pha 5 (calibration), ghi vào `calibration_decisions.csv` kèm mã `R-xx`.
3. Quyết định làm đổi cách chọn vật / lớp / box → tăng phiên bản (`semantic`), kiểm lại cả lô.

### Câu hỏi mở mang ra họp Pha 2

1. Có thêm lớp `bus` và `three_wheeler` không? (img_004, img_009, img_016, img_035, img_037, img_040)
2. Songthaew (img_008) là `truck` hay ngoài phạm vi?
3. Xe độ / xe nghệ thuật (img_011, img_019, img_024): tiêu chí “thật” là “chạy được trên đường”?
4. Ngưỡng “tách được” trong đám đông (img_019, img_021, img_032): cần thấy ít nhất đầu + vai?
5. Xe máy chỉ thấy gương / tay lái (img_005, img_040): gán hay không?

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-<mã> | Định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng % cho `occluded_level`; định nghĩa `truncated`; 18 quy tắc R-01…R-18; bảng ca khó 30 ảnh | v0 mơ hồ ở: người đi xe máy (img_026, img_010), vật không thật (img_017, img_027, img_033, img_038), xe chở xe (img_002), đám đông (img_021, img_030), xe ngoài taxonomy (img_004, img_009) |

---

## Phụ lục — Bảng rà nhanh 40 ảnh

Ghi lại lúc xem ảnh (chưa vẽ box). “—” = không có vật cần gán.

| Ảnh | Nội dung chính | Lớp dự kiến | Quy tắc cần chú ý |
|---|---|---|---|
| img_001 | Quán vỉa hè, bán tải Suzuki chở ống, người ngồi | `truck`, `person` | R-08, R-12 (poster) |
| img_002 | Xe đầu kéo chở 7 ô tô | `truck`, `car`, `person` | R-14 |
| img_003 | Phố Hà Nội, người gánh mít, xe đạp, xe máy | `person`, `motorcycle`, `car` (xa) | R-05, R-02 |
| img_004 | 2 tuk-tuk ban đêm có tài xế | `person` | R-02, R-06 |
| img_005 | Cận cảnh người đội mũ trên xe máy, van, ô tô | `person`, `motorcycle`, `van`, `car` | R-03, R-09 |
| img_006 | Dòng xe máy + xe đạp nhìn từ sau, taxi | `person`, `motorcycle`, `car` | R-02, R-03 |
| img_007 | Kho bốc hàng, 2 xe tải | `truck`, `person` | R-16 |
| img_008 | Bãi xe máy đêm, xe ôm | `motorcycle`, `person`, `ignore_region` | R-11 |
| img_009 | Minibus, xe máy xa | `motorcycle`, `person` | R-02, R-10 |
| img_010 | Người bán chuối, xe tay ga | `person`, `motorcycle` | R-03 |
| img_011 | Tượng Predator, xe máy độ, lễ hội | `person`, `van`, (`motorcycle`?) | R-12, `can_review` |
| img_012 | Thùng bán tải chở hàng, Mazda | `truck`, `car` | R-09, R-13 |
| img_013 | Đêm, tượng rồng, người đi bộ | `person`, `motorcycle`, `car` | R-12, R-10 |
| img_014 | Người qua đường, xe mờ | `person`, `ignore_region` | R-15 |
| img_015 | Ô tô mưa, xe mờ phía sau | `car`, `ignore_region` | R-09, R-15 |
| img_016 | Bangkok: tuk-tuk, Vespa, Hiace | `motorcycle`, `person`, `van`, `truck`, `car` | R-02, R-04, R-16 |
| img_017 | Tượng người cưỡi ngựa | — | R-12, R-17 |
| img_018 | Biển bầu cử, bãi xe sau hàng rào | `car`, `truck` | R-12, R-08 |
| img_019 | Xe Dangermouse diễu hành, đám đông | `car`, `person`, `ignore_region` | R-01, R-11 |
| img_020 | Gương chiếu hậu xe chủ | — | R-12, R-13 |
| img_021 | Ngã tư đông nghẹt xe máy | `person`, `motorcycle`, `car`, `ignore_region` | R-03, R-11 |
| img_022 | Mô tô có thuyền (đen trắng) | `motorcycle`, `person` | R-18 |
| img_023 | Ô tô đen, xích lô, xe máy | `car`, `person`, `motorcycle` | R-02, R-06 |
| img_024 | Bảo tàng xe, xe vàng thu nhỏ trên nóc | `car` | R-12, `can_review` |
| img_025 | Phố biển, người đi bộ, xe máy kéo xe hàng | `person`, `car`, `truck`, `motorcycle`, `ignore_region` | R-10, R-18 |
| img_026 | 2 người trên xe máy, tranh tường | `person`, `motorcycle` | R-03, R-12 |
| img_027 | Bia mộ hình bán tải | — | R-12, R-17 |
| img_028 | Bán tải chở tượng ngựa, SUV | `truck`, `car` | R-12 |
| img_029 | Xe máy chở ống nhôm, xe đạp | `person`, `motorcycle` | R-04, R-05, R-02 |
| img_030 | Bãi gửi xe máy dày, poster | `motorcycle`, `person`, `ignore_region` | R-11, R-12 |
| img_031 | Phố đêm mưa Đài Bắc | `car`, `motorcycle`, `person` | R-02, R-12 |
| img_032 | Hàng người chờ buýt | `person`, `car`, `ignore_region` | R-11, R-09 |
| img_033 | Ma-nơ-canh, phản chiếu | — | R-12, R-17 |
| img_034 | Người đi bộ qua đường, ô tô, xe máy | `person`, `car`, `motorcycle` | R-03, R-09 |
| img_035 | Auto rickshaw chở hàng, xe máy | `motorcycle`, `person`, `car` | R-02 |
| img_036 | Xe tải nhẹ Veryca thùng bạt | `truck`, `motorcycle`, `car` (xa) | R-16 |
| img_037 | Bán tải gắn nắp thùng, xe buýt | `truck`, `person` | R-16, R-02 |
| img_038 | Tủ kính ô tô đồ chơi | — | R-12, R-17 |
| img_039 | Phố Huế, xe máy, người đi bộ | `person`, `motorcycle`, `car` (xa) | R-10 |
| img_040 | Người đội mũ cận cảnh, taxi, tuk-tuk | `person`, `car` | R-03, R-06, R-02 |
