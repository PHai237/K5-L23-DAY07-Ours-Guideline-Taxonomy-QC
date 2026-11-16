# Guideline gán nhãn — v0.2-DRAFT — mã 047

> **DỰ THẢO HỢP NHẤT ĐỂ KIỂM TRA VÀ THẢO LUẬN, KHÔNG PHẢI BẢN NHÓM ĐÃ DUYỆT.** Tài liệu đối chiếu `guideline_047.md` với bản tham khảo `guideline_101_DRAFT_tham_khao.md`, mẫu `guideline/TEMPLATE_guideline.md`, `guideline_v0_starter.md` và ảnh `img_001`–`img_040`. Những quyết định, cách phân lớp và tình huống chưa chắc cần người gán **xác minh trên ảnh gốc và tự chốt**. Theo `RULES.md`, AI không thay người gán đưa ra quyết định cá nhân hoặc vẽ box.

- **Phiên bản:** `v0-DRAFT`
- **Ngày biên tập:** 2026-11-16
- **Mã thành viên tham chiếu:** `047`; **người duyệt:** chưa có
- **Loại nhãn:** bounding box 2D, ảnh RGB đường phố; **công cụ:** CVAT; **xuất:** COCO 1.0
- **Phạm vi xem xét:** 40 ảnh `data/images/img_001.jpg` – `img_040.jpg`
- **Trạng thái taxonomy:** mô tả **phương án cá nhân giữ 6 lớp của taxonomy v0**; có đề xuất khác để nhóm quyết trong CP2

## 1. Mục tiêu

Chuẩn hóa cách gán bounding box để huấn luyện và đánh giá mô hình **phát hiện người và những loại phương tiện được định nghĩa trong taxonomy** trên ảnh đường phố từ camera giao thông hoặc camera gắn trên xe. Mỗi vật thể thật thuộc phạm vi và đủ điều kiện có **một box, một lớp**; người điều khiển/phụ lái và phương tiện là các vật thể khác nhau. Box mô tả **phần nhìn thấy**, không dựng lại phần khuất.

Các lỗi gây hậu quả đáng chú ý: (1) bỏ sót người/xe thật khiến mô hình phát hiện thiếu; (2) nhầm tượng, hình in, ma-nơ-canh, đồ chơi, phản chiếu thành mục tiêu làm tăng cảnh báo giả; (3) nhầm lớp `car`/`truck`/`van` hoặc loại phương tiện ba bánh; (4) gộp người với xe, gộp nhiều xe, hoặc lấy cả hàng chở vào box khiến định vị sai; (5) gán khác nhau giữa các thành viên làm giảm chất lượng dữ liệu.

**Giới hạn phạm vi của phương án v0.2:** không có lớp `bus`, `tuk_tuk`, `bicycle`, `animal` trong 6 lớp đang dùng. Vì vậy mục tiêu **không** phải phát hiện *mọi* loại phương tiện; các đối tượng ngoài taxonomy được xử lý theo mục 2 và R-11, không ép sang lớp gần nhất một cách tùy tiện. Ở CP2, nhóm có thể quyết định mở rộng taxonomy để phù hợp mục tiêu dự án.

## 2. Lớp và thuộc tính

### Lớp

**Phương án A — ưu tiên tương thích taxonomy v0:** 5 lớp vật thể (`car`, `truck`, `van`, `motorcycle`, `person`) và một nhãn vùng kỹ thuật (`ignore_region`). Mỗi vật thể thực chỉ nhận một **lớp vật thể**; `ignore_region` **không phải lớp vật thể**. Các ngưỡng ở `submission/taxonomy.json` → `rules` phải giữ nguyên.

| Lớp | Định nghĩa (một câu) | Ví dụ trong ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con thực, gồm sedan, hatchback, wagon, SUV và taxi dùng nền tảng ô tô con. | Ô tô được chở `img_002`; SUV/ô tô `img_007`, `img_023`, taxi `img_040`. | Pickup (`truck`), van thân hộp (`van`), tượng pickup `img_027`, đồ chơi `img_038`, thân vỏ trang trí `img_024`. |
| `truck` | Xe thực có cabin và phần/sàn/thùng chuyên chở hàng riêng, gồm pickup, xe tải thùng và xe chuyên chở ô tô. | `img_001`, `img_002`, `img_012`, `img_036`, pickup có canopy `img_037`. | Van thân kín liền khối; hàng rời nằm trên xe; mô hình/tượng xe `img_027`. |
| `van` | Xe thực thân hộp kín hoặc khoang chở người/hàng liền với cabin kiểu van/minivan; trong phương án A gồm minibus nhỏ kiểu van. | Van `img_005`, minibus nhỏ `img_009`, xe van `img_016`. | SUV thông thường (`car`); pickup có canopy (`truck`); xe buýt cỡ lớn (`ngoài taxonomy` theo phương án A). |
| `motorcycle` | Xe cơ giới hai bánh kiểu mô tô/scooter hoặc cụm ba bánh kiểu tuk-tuk/mô tô gắn sidecar trong phương án A. | Xe máy `img_005`, `img_006`, `img_026`; tuk-tuk `img_004`, `img_035`; sidecar `img_022`. | Xe đạp, xích lô đạp bằng sức người `img_023`, `img_039`; điêu khắc xe `img_011`. |
| `person` | Người thật nhìn trực tiếp trong cảnh, kể cả người đi bộ, ngồi/lái xe hay mặc trang phục hóa trang, nếu đủ bằng chứng là người thật. | `img_001`, `img_005`, `img_026`, `img_034`, `img_040`. | Tượng `img_017`; chân dung biển quảng cáo `img_018`; ma-nơ-canh `img_033`; đồ chơi `img_038`; nhân vật nghi là hình nộm `img_019`. |
| `ignore_region` | Vùng chữ nhật trong ảnh chứa nhiều mục tiêu thuộc taxonomy nhưng không thể tách/gán từng đối tượng ổn định; không biểu diễn một vật riêng. | Một phần cụm xa rất dày ở `img_021`, `img_030` nếu thực sự không thể tách. | Vật đơn lẻ khó phân lớp, vật nhỏ lẻ, vật ngoài taxonomy, tượng, phản chiếu; không dùng để che lỗi bỏ sót. |

**Vật ngoài taxonomy theo phương án A:** xe đạp, xích lô đạp không động cơ, bus lớn, động vật, xe kéo tay, hàng hóa rời, tượng, hình in, ma-nơ-canh, mô hình/đồ chơi, vật chỉ có trong phản chiếu: **không gán đối tượng và không tạo `ignore_region` chỉ vì bị loại**. Người thật đứng/ngồi trên một phương tiện ngoài taxonomy vẫn được gán `person` nếu đạt điều kiện. Xe đỗ và xe đang được chở vẫn thuộc phạm vi nếu **bản thân xe** là phương tiện thật hoàn chỉnh đủ điều kiện.

**Phương án B để cả nhóm cân nhắc tại CP2 (chưa áp dụng):** thêm `bus` (xe buýt chở khách, kể cả minibus nếu chọn theo công năng) và `tuk_tuk` (auto-rickshaw ba bánh), giữ các lớp còn lại và ngưỡng QC. Nếu chọn B, nhóm phải viết lại ranh giới `bus`/`van`, `tuk_tuk`/`motorcycle`, cập nhật `taxonomy.json` và `cvat/labels_generated.json` trước khi gán độc lập; **không dùng đồng thời hai phương án**.

### Thuộc tính

Các thuộc tính sau dành cho **box vật thể thật**. `occluded_level` đo **ước lượng tỷ lệ phần thân vật bị một vật khác trong cảnh che**, trên hình thể toàn vật *nếu có thể ước lượng*, không tính phần bị cắt ra ngoài khung vào tỷ lệ che. Chỉ ước lượng khi có căn cứ hình dạng, không cố đo chính xác từ phần không nhìn thấy.

| Thuộc tính | Giá trị cố định | Nghĩa, ngưỡng và cách chọn |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che (0% phần thân). |
| | `1` | Bị che **trên 0% đến 30%**, gồm 30%. |
| | `2` | Bị che **trên 30% đến 70%**, gồm 70%. |
| | `3` | Bị che **trên 70%**, nhưng vẫn đủ bằng chứng nhận diện vật/lớp; nếu không đủ, xử lý theo R-17. |
| `truncated` | `no` | Không có phần nào của vật nằm ngoài khung hình; vật bị che **bên trong** ảnh vẫn có thể `no`. |
| | `yes` | Một phần của vật **bị cắt thực sự bởi mép ảnh**; không chỉ vì box chạm mép do sát khung. |
| `can_review` | `false` / không tick | Người gán đủ chắc về đối tượng, lớp, biên box và thuộc tính. |
| | `true` / tick | Vẫn vẽ được box hợp lệ nhưng chưa chắc về lớp, ranh giới, mức che, tính thật/giả; **bắt buộc** ghi ảnh/vùng/lý do để reviewer phân xử. |

Ví dụ: người bị xe che khoảng nửa thân, toàn thân vẫn nằm trong ảnh ⇒ `occluded_level=2`, `truncated=no`; người bị mép phải cắt nhưng không bị vật khác che ⇒ `occluded_level=0`, `truncated=yes`. **Nhòe chuyển động, thiếu sáng không tự bằng `occluded_level>0`**.

Ngưỡng kích thước **bắt buộc**: `min_box_height_px=20` theo taxonomy. Chiều cao đo theo **box của phần nhìn thấy**, không phải chiều cao toàn vật tưởng tượng. Dưới 20 px: không vẽ box vật thể riêng; từ 20 px: gán khi còn xác định được là vật thật đúng lớp.

## 3. Quy tắc vẽ box

| Rule ID | Quy tắc có thể thao tác và kiểm lại |
|---|---|
| **R-01 — Đối tượng thật** | Chỉ gán vật thể thật, trực tiếp trong không gian ảnh và thuộc taxonomy đã khóa. Xe đang đỗ hoặc xe thật đang được chở vẫn xét gán; không phụ thuộc đang chuyển động. |
| **R-02 — Một vật, một lớp, một box** | Mỗi người/xe đủ điều kiện có một box riêng. Không nhập hai xe, hai người, hoặc cả cụm đông vào một box vật thể. |
| **R-03 — Box phần nhìn thấy** | Vẽ hình chữ nhật không xoay, ôm sát cực trị các **phần nhìn thấy** của chính vật đó, không dựng phần khuất; phần lộ ra rời nhau của cùng vật vẫn dùng một box bao chúng. Không lấy bóng đổ, nền, vật che hoặc hàng rời. Ưu tiên mép chính xác khi nhìn rõ; mục tiêu ±2 px/cạnh chỉ là tham chiếu thực hành, **không phải ngưỡng chấm riêng**. |
| **R-04 — Người và phương tiện độc lập** | Người lái/khách: mỗi `person` riêng và một box xe riêng nếu xe đáp ứng điều kiện. Box `person` bao đường viền cơ thể, **quần áo và mũ bảo hiểm đang đội trực tiếp** (đề xuất cần nhóm chốt); không bao túi xách, đồ cầm, xe và hàng. Không tự suy ra một xe bị che hoàn toàn chỉ vì nhìn thấy người ngồi. |
| **R-05 — Che khuất** | Vật khác che một phần ⇒ chỉ box phần lộ ra; `occluded_level` theo ngưỡng ở mục 2. Bóng đổ hoặc motion blur không phải che khuất. |
| **R-06 — Cắt mép ảnh** | Box chỉ nằm trong ảnh. Nếu vật vượt khỏi mép khung thì `truncated=yes` dù có/không có che khuất; hai thuộc tính độc lập. |
| **R-07 — Xe chở xe** | `img_002`: một box `truck` cho cấu trúc xe tải/chuyên chở nhìn thấy; một box `car` **cho từng ô tô thật** trên sàn đủ điều kiện. Box truck không mở rộng để ôm những ô tô đó như một phần thân xe. |
| **R-08 — Hàng và bộ phận gắn cố định** | Box phương tiện gồm cabin, khung, bánh, phần thân/nóc/canopy cố định thuộc xe; không ôm bao tải, hàng, ống/thanh dài, tượng hay đồ đặt/chằng trên xe. Canopy có thể tính trong box `truck` nhưng không tự làm xe đổi thành `van`. |
| **R-09 — Ba bánh và sidecar** | Theo phương án A: mỗi tuk-tuk có động cơ là một `motorcycle`; mô tô gắn sidecar cố định là một `motorcycle` bao khung và sidecar. Người lái/hành khách đủ điều kiện là `person` riêng. Xích lô đạp/xe đạp không được gọi là tuk-tuk. |
| **R-10 — Phân lớp theo kết cấu** | `car` cho ô tô con/SUV/taxi; `truck` cho xe tải, pickup và xe có sàn/thùng chở hàng riêng; `van` cho van/minivan thân kín, gồm minibus kiểu van nếu phương án A. Với xe kết cấu lai, xét thân xe gốc và ghi vấn đề nếu không chắc. |
| **R-11 — Ngoài taxonomy** | Không gán vật ngoài lớp (trong phương án A gồm bus lớn, xe đạp/xích lô đạp, hàng hóa và động vật); không dùng `ignore_region` như lớp `other`. Muốn thêm lớp chỉ quyết tại CP2, rồi đồng bộ schema. |
| **R-12 — Kích thước tối thiểu** | Vật cao theo box nhìn thấy `<20 px` không vẽ; `≥20 px` gán nếu nhận diện chắc. Không phóng box hoặc gộp nhiều vật để vượt ngưỡng. |
| **R-13 — Đám đông / vùng ignore** | Tách và gán từng vật ≥20 px có thể nhận diện độc lập. `ignore_region` chỉ khoanh phần nhiều đối tượng thuộc taxonomy quá chồng lấn/thiếu thông tin, không thể gán từng vật nhất quán; khoanh vùng nhỏ nhất phù hợp và ghi lý do. |
| **R-14 — Tượng / ảnh in / đồ chơi / mô hình** | Không gán tượng người/xe, ma-nơ-canh, hình trên quảng cáo/màn hình, đồ chơi, mô hình hoặc điêu khắc xe. **Trường hợp người hóa trang hay xe trưng bày có thể là người/xe thật:** xác minh bằng dấu hiệu trực quan và chuyển R-17 nếu chưa rõ. |
| **R-15 — Xe chủ và phản chiếu** | Không gán bộ phận phương tiện gắn camera (*ego*) lọt vào mép ảnh. Không gán đối tượng **chỉ thấy trong gương/bề mặt phản chiếu**; đối tượng thật ở ngoài xe chủ vẫn được xét bình thường. |
| **R-16 — Tối/nhòe** | Gán vật khi còn đủ dấu hiệu xác định lớp và phần hình thể nhìn được. Không mở box để bao vệt motion blur dài; ranh giới không chắc thì `can_review` nếu box vẫn hợp lệ, hoặc ghi image issue nếu không thể vẽ. |
| **R-17 — Ca chưa chắc / reviewer** | Vật chắc thuộc taxonomy, vẽ được nhưng còn phân vân ⇒ box dự kiến + `can_review=true`, ghi ảnh/vùng/lý do. Không đủ bằng chứng để tạo box hợp lệ hoặc không có lớp tương ứng ⇒ không tự đoán, ghi **vấn đề cấp ảnh/vùng** (để thu vào `image_issues.csv` sau khóa nhãn). Không dùng ignore thay cho can_review. |
| **R-18 — Vật đặc biệt và trang trí** | Xe diễu hành, xe cổ, xe đỗ hoặc xe được chở: xét phần **phương tiện thật có chức năng xe**, loại lớp trang trí/hình nhân/hàng trên xe; thiếu bằng chứng thật/giả thì xử lý R-17, không khẳng định chỉ vì giống hình dạng. |

**Quy ước kiểm box:** tọa độ nằm trong biên ảnh; box có độ rộng/chiều cao dương; chiều cao tối thiểu 20 px; không vẽ tách mỗi bộ phận thành các box riêng; không suy đoán phần không nhìn thấy. Các quy tắc ưu tiên thống nhất giữa 5 người **trước khi** bắt đầu gán độc lập.

## 4. Ca khó

Bảng này là **phiếu rà soát đủ 40 ảnh** với nhận định đề xuất, không phải danh sách nhãn hoặc bằng chứng đã vẽ box. Dòng `[XÁC MINH]` / `[HỌP CP2]` là nơi người gán cần xem lại ảnh gốc hoặc cả nhóm quyết định; nếu không có box hợp lệ, ghi vấn đề cấp ảnh theo R-17.

| Ảnh | Tình huống | Rule ID | Quyết định đề xuất theo phương án A | Lý do / điểm cần kiểm |
|---|---|---|---|---|
| `img_001` | Pickup chở vật dụng, nhiều người quanh thùng. | R-02, R-04, R-08, R-10 | Pickup là `truck`; từng người thật đủ ngưỡng là `person`; không lấy hàng vào box. | Phân biệt thân xe, hàng và người bị che. |
| `img_002` | Xe tải hai tầng chở nhiều ô tô. | R-02, R-07 | Một `truck` cho xe chở và mỗi ô tô thật lộ ra một `car` riêng nếu đủ 20 px. | Không gộp cả đoàn xe thành một box. |
| `img_003` | Gánh hàng, người đi bộ, xe đạp, xe máy. | R-04, R-11, R-12 | Người thật và xe máy đủ điều kiện gán riêng; xe đạp không gán; không ôm giỏ/quang gánh trong `person`. | Phân biệt người với đồ mang theo và xe không động cơ. |
| `img_004` | Hai tuk-tuk có động cơ, cabin và người bên trong. | R-04, R-09 | Mỗi tuk-tuk một `motorcycle`, người thật thấy rõ đủ ngưỡng gán `person`. **[HỌP CP2]** có tách `tuk_tuk`? | Lớp `motorcycle` v0 không nói rõ xe ba bánh. |
| `img_005` | Người đi xe máy chen nhau, van phía sau. | R-04, R-05, R-10 | Mỗi người/xe máy một box khi đủ điều kiện; xe van thân kín là `van`; mức che tính riêng. | Ca chính kiểm tách người–xe và occlusion. |
| `img_006` | Nhiều xe máy, xe đạp và người điều khiển. | R-04, R-09, R-11 | `motorcycle` cho xe cơ giới, `person` cho từng người đủ ngưỡng; xe đạp không gán. | Một người đi xe đạp vẫn là `person`. |
| `img_007` | Pickup và xe tải trong garage. | R-01, R-10 | Gán xe thật theo cấu trúc dù đang đỗ; pickup/xe tải là `truck`, ô tô con là `car` nếu có. | Xe đỗ không bị loại; không gán phần không nhìn được. |
| `img_008` | Xe máy đỗ dày, nhiều người và hàng quán. | R-02, R-12, R-13 | Tách xe/người nhận diện được; `ignore_region` chỉ nếu có phần thật sự không thể tách. | Không tạo một box cho cả dãy xe. |
| `img_009` | Minibus nhỏ dạng van ở phố. | R-10, R-11 | Phương án A: `van` nếu cấu trúc đúng van/minibus; **[HỌP CP2]** có bổ sung `bus` và đổi cách phân lớp? | Cần ngưỡng nhận biết bus/van nhất quán. |
| `img_010` | Xe máy cùng người và quầy hàng rau. | R-04, R-08, R-11 | Người, xe máy được tách riêng; xe đẩy/quầy hàng/rau không gán. | Đồ chở khác với vật thuộc taxonomy. |
| `img_011` | Hình nhân Predator rất lớn, các mô tô điêu khắc và người xa. | R-14, R-17 | Điêu khắc xe không gán; người thật ở nền đủ ngưỡng gán; **[XÁC MINH]** hình nhân lớn là người mặc đồ hay tượng. | Ảnh tĩnh chưa đủ để khẳng định hình nhân là người thật. |
| `img_012` | Pickup chở thùng giấy, túi hàng. | R-08, R-10 | Một `truck` cho xe; loại hàng rời khỏi box; xe thật khác đủ điều kiện gán riêng. | Box xe không bao khối hàng vượt thùng. |
| `img_013` | Đường phố tối, đèn và đối tượng ở xa. | R-12, R-16, R-17 | Chỉ gán người/xe còn phân lớp được và cao ≥20 px; vùng không đủ bằng chứng ghi issue. | Thiếu sáng khác che khuất. |
| `img_014` | Người đi bộ rõ, xe máy lao qua bị nhòe. | R-04, R-16, R-17 | Người rõ là `person`; xe/người chuyển động chỉ gán nếu nhận diện được, không kéo box theo vệt mờ. | Motion blur không tự thành occlusion. |
| `img_015` | Ô tô ở hai mép ảnh, nhiều xe đỗ phía đối diện. | R-06, R-10 | Xe sát mép **thực sự bị cắt** đặt `truncated=yes`; các xe thật khác đủ điều kiện vẫn gán. | Đây là ca **truncation**, không phải ca motion blur chính. |
| `img_016` | Scooter, xe ba bánh chở hàng/khách, van phía sau. | R-04, R-09, R-10 | Xe máy và tuk-tuk dùng `motorcycle` theo A, van gán `van`, người gán riêng. **[HỌP CP2]** ranh giới tuk-tuk/xe ba bánh. | Nhiều dạng xe có thể bị nhầm lớp. |
| `img_017` | Tượng người cưỡi ngựa. | R-11, R-14 | Không gán tượng thành `person`; ngựa ngoài taxonomy. | Cảnh có hình dạng người nhưng không phải người thật. |
| `img_018` | Chân dung trên biển quảng cáo, ô tô ở dưới. | R-01, R-14 | Không gán chân dung; gán xe thật đủ điều kiện. | Hình in không phải mục tiêu trực tiếp. |
| `img_019` | Ô tô diễu hành màu vàng, hình chuột và đám đông. | R-14, R-17, R-18 | Xe nền thật có chức năng ô tô: `car`; người thật đủ điều kiện `person`; **[XÁC MINH]** nhân vật chuột là người hóa trang hay hình nộm. | Không lấy cả hình nhân trang trí vào box xe, không đoán bản chất nhân vật. |
| `img_020` | Phần xe chủ và ảnh giao thông trong gương. | R-15 | Không gán bộ phận ego và vật chỉ nằm trong phản chiếu. | Không biến ảnh trong gương thành đối tượng trực tiếp. |
| `img_021` | Cụm kẹt xe lớn, nhiều người và xe máy chồng nhau. | R-02, R-12, R-13 | Tách từng mục tiêu đủ 20 px/nhận diện được; ignore riêng vùng xa không thể tách. | Không ignore toàn bộ đám đông; thống nhất ranh vùng ignore. |
| `img_022` | Xe máy cổ với sidecar trên đường, ảnh đen trắng. | R-03, R-09 | Một `motorcycle` bao kết cấu sidecar cố định; người thật đủ điều kiện gán riêng. **[HỌP CP2]** xác nhận phạm vi sidecar. | Một tổ hợp xe vận hành cùng nhau hay hai đơn vị? |
| `img_023` | Người chạy xích lô đạp, ô tô và xe máy. | R-04, R-10, R-11 | Người thật `person`; ô tô `car`, xe máy `motorcycle`; thân xích lô đạp không gán. | Không ép xích lô sức người thành tuk-tuk. |
| `img_024` | Xe vàng/thân vỏ đặt trên nóc xe đen trong triển lãm. | R-14, R-17, R-18 | Gán các ô tô thật hoàn chỉnh; **[XÁC MINH]** phần xe vàng là phương tiện thật hoàn chỉnh hay thân vỏ trang trí: thân vỏ rời thì loại. | Không thể chỉ dựa màu sắc/kích thước để kết luận. |
| `img_025` | Phố ven biển, người đi dạo, xe xa. | R-11, R-12, R-13 | Gán các người/xe thật cao ≥20 px, chắc lớp; không ignore các đối tượng nhỏ lẻ chỉ vì dưới ngưỡng. | Kiểm ngưỡng 20 px và khoảng cách. |
| `img_026` | Hai người trên xe máy trước bức tranh tường. | R-04, R-14 | Hai box `person` và một `motorcycle` nếu đủ điều kiện; không gán hình vẽ trên tường. | Test tách người–xe và loại ảnh in. |
| `img_027` | Tượng pickup đặt trên bệ. | R-14, R-18 | Không gán `truck` cho tượng/mô hình xe. | Hình dáng và kích thước giống xe thật không đủ. |
| `img_028` | Pickup cổ chở tượng động vật. | R-08, R-10, R-14 | Pickup thật `truck`; tượng/hàng chở không gán. | Không mở box truck để ôm tượng. |
| `img_029` | Xe đạp và xe máy chở thanh dài, người gần đó. | R-04, R-08, R-11 | Người thật và xe máy đủ ngưỡng được gán riêng; xe đạp không gán; thanh dài không thuộc box xe/người. | Ngoại vật dài làm lệch box. |
| `img_030` | Bãi xe máy đỗ rất dày. | R-02, R-12, R-13 | Mỗi xe máy tách được một box; ignore phần xa chồng kín nếu không thể tách. | Đừng gộp nhiều xe thành một `motorcycle`. |
| `img_031` | Phố đêm đông xe, có thể có bus ở xa. | R-10, R-11, R-16 | Gán các ô tô/xe thuộc taxonomy đủ điều kiện; bus lớn loại theo A. **[XÁC MINH]** xe nghi bus phải nhìn lại ảnh gốc. | Ảnh tối và xe xa dễ nhầm van/bus. |
| `img_032` | Đám đông bên kia đường, ô tô cắt mép. | R-06, R-12, R-13 | Người tách được đủ điều kiện gán; chỉ ignore phần dày không thể tách; ô tô bị mép cắt đặt `truncated=yes`. | Kết hợp đông người và truncation. |
| `img_033` | Ma-nơ-canh tại tủ/trưng bày. | R-14 | Không gán `person` cho ma-nơ-canh. | Vật không sống dù giống người thật. |
| `img_034` | Người qua đường và nhiều xe ở giao lộ. | R-02, R-04, R-05 | Người thật và xe thật tách riêng; vật bị che gắn mức occluded cho chính vật đó. | Người đứng gần/đi trước xe không được gộp box. |
| `img_035` | Tuk-tuk chở bao tải chồng rất cao. | R-08, R-09 | Tuk-tuk có động cơ `motorcycle` theo A; box chỉ ôm cấu trúc xe, không ôm hàng. **[HỌP CP2]** nếu đổi thành `tuk_tuk`. | Hàng đội cao là nguyên nhân IoU thấp khi box không thống nhất. |
| `img_036` | Xe tải nhỏ cabin xanh, thùng hàng tối. | R-08, R-10 | Một `truck`, bao cabin và thùng cố định, không phụ thuộc kích thước xe nhỏ. | Cấu trúc có khu vực tải riêng. |
| `img_037` | Bus lớn và pickup có canopy cao. | R-08, R-10, R-11 | Theo A: bus lớn **không gán**; pickup là `truck` với canopy cố định trong box. **[HỌP CP2]** nếu thêm `bus`. | Canopy không mặc định biến pickup thành van. |
| `img_038` | Tủ kính đầy ô tô và nhân vật đồ chơi. | R-14 | Không gán `car` hay `person` cho đồ chơi. | Không phải người/xe thật trong cảnh giao thông. |
| `img_039` | Phố có xe đạp, xe máy và người đi đường. | R-04, R-11, R-12 | Gán người và xe máy đủ ngưỡng; loại xe đạp/xích lô đạp. | Xác minh cơ giới hay sức người khi xe ở xa. |
| `img_040` | Người đội mũ sát camera, taxi, tuk-tuk, bus ở nền. | R-04, R-06, R-09, R-11, R-15 | `person` thật đủ điều kiện, taxi là `car`, tuk-tuk là `motorcycle`, bus lớn ngoài A; không gán phần ego. **[HỌP CP2]** mũ có nằm trong box person và taxonomy xe ba bánh. | Giao nhau của trang phục, cắt mép và phân lớp phương tiện. |

**Các quyết định nên ưu tiên họp CP2:**

1. **Taxonomy:** giữ phương án A hay bổ sung `bus`/`tuk_tuk` (các ảnh `004`, `009`, `016`, `031`, `035`, `037`, `040`)? Ai quyết, cập nhật nhãn CVAT ra sao?
2. **Phụ kiện đeo trên người:** có gồm mũ bảo hiểm đang đội trực tiếp vào box `person` không (`005`, `040`)? Quy tắc về đồ đội/đồ cầm phải thống nhất.
3. **Đối tượng thật hay mô hình:** `img_011` (hình nhân lớn), `019` (nhân vật chuột), `024` (thân xe vàng), `027` (tượng pickup).
4. **Hình học của phương tiện phức tạp:** box truck và xe được chở `002`, sidecar `022`, xe có canopy `037`, tuk-tuk chất hàng `035`.
5. **Ranh giới vùng ignore:** đánh dấu cụ thể trên ảnh `021`, `030`, `032`; không dùng ignore thay cho box đủ tiêu chuẩn.

## 5. QC và phiên bản

### Ngưỡng QC — lấy từ `submission/taxonomy.json` v0, **không chỉnh mục `rules`**

| Chỉ tiêu | Giá trị | Quy tắc kiểm |
|---|---:|---|
| Chiều cao box tối thiểu `min_box_height_px` | `20 px` | Đối tượng có box phần nhìn thấy thấp hơn ngưỡng bị loại. |
| Ngưỡng vật lớn `large_px` | `50 px` | Vật có chiều cao box ≥50 px là lớn cho bài kiểm hình học. |
| IoU vật lớn `iou_thr_large` | `0.85` | So box với tham chiếu/box kiểm tương ứng; không đồng nhất với ngưỡng ghép IAA. |
| IoU vật nhỏ `iou_thr_small` | `0.70` | Cho vật từ 20 đến dưới 50 px. |
| IoU để ghép cặp `match_iou` | `0.50` | Ghép các box đối chiếu 1–1; box không ghép được phải xem riêng. |
| IAA IoU trung bình `iaa_iou` | `0.85` | Báo cả **số cặp ghép**; không tính chỉ số khi mẫu số bằng 0 (ghi N/A). |
| Đồng thuận lớp IAA `iaa_class` | `0.95` | Tính trên cặp box ghép và ghi tử số/mẫu số. |
| Tỉ lệ ảnh lỗi tối đa `error_rate_img_max` | `0.02` (2%) | Duyệt 40/40 ảnh theo quy trình; 1 ảnh lỗi / 40 ảnh = 2,5%, vượt 2%. |

- Với **5 thành viên**, dùng 3 người gán/ảnh; 40 ảnh tương ứng **120 lượt gán**, chia cân đối sẽ có 24 ảnh/người; chỉ tính khi `assignment.csv` xác nhận phân công thực tế. Một ảnh có 3 người tạo 3 cặp so sánh người–người, **không phải** mặc định có 3 cặp box khớp.
- Trước gán, khóa `guideline_v1.0_agreed.md`, `taxonomy.json` v1.0 và sinh nhãn CVAT; **không trộn hai phiên bản taxonomy trong lượt gán độc lập**.
- Chạy `scripts/validate_labels.py` để kiểm COCO/schema/lớp/thuộc tính/hộp; `scripts/qc_compare.py` và `scripts/viz_disagreement.py` để đọc IAA và ca lệch. Các chỉ số phải ghi **mẫu số thật**, kể cả không ghép box.
- Người QC bằng mắt phải xem lại **box đúng vật, đủ đối tượng, hàng hóa, người đi xe, vật không thật, ignore, mũ/canopy, ca nghi bus/tuk-tuk**. Script không tự xác nhận mọi vật thật đều được gán.
- `can_review` phải đi cùng lý do; nếu không tạo được box thì ghi `image_issues.csv` có `image,annotator,region,note` sau khi chốt/gom nhãn, không giả tạo nhãn để phục vụ QC.
- **Hold** nếu thiếu bằng chứng, chưa khóa quy tắc hoặc có vấn đề dữ liệu; **Rework** khi tỉ lệ lỗi vượt ngưỡng hay lỗi có tính hệ thống; **Release** chỉ khi bằng chứng đủ và reviewer xác nhận. Báo hai tỷ lệ lỗi **theo ảnh** và **theo đối tượng** với tử số/mẫu số, không sáng tác kết quả khi chưa review.
- Khi calibration: mỗi ca bất đồng phải có quyết định, ảnh, Rule ID, người chốt, phiên bản, phạm vi ảnh/cả lô, file sửa và người kiểm lại. Nếu thay đổi ý nghĩa quy tắc, tăng lên `v1.1`, ghi `guideline_change.json` và kiểm lại cả lô theo quy định.

**Người quyết định ca mơ hồ:** người gán độc lập ghi trước; ở CP2/CP5, nhóm họp và chốt theo phân công trong `TEAM.md`, ghi mã thành viên chịu trách nhiệm và reviewer. Người biên tập tài liệu này **không** tự xác nhận quyết định thay 5 thành viên.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Starter của lab: 5 lớp vật thể + vùng ignore, định nghĩa còn mơ hồ. | Không đủ tiêu chí cho người đi xe, ba bánh, vật giả, che/cắt mép. |
| `v0.1-individual` (`guideline_047.md`) | 15 quy tắc, ngưỡng thuộc tính, 22 dòng tình huống, phương án giữ 6 lớp. | Các ca `002`, `004`, `005`, `017`, `020`, `021`, `035`, `037`. |
| `v0.1-DRAFT` (`guideline_101_DRAFT_tham_khao.md`) | Đề xuất bổ sung `bus`/`tuk_tuk`, có bảng đủ 40 ảnh. | Ca minibus/tuk-tuk và xe đặc biệt. |
| `v0.2-DRAFT` (tài liệu này) | So sánh hai bản, mở rộng 40 ảnh, giữ một phương án taxonomy đang áp dụng, thêm điều kiện xác minh và quy tắc phụ kiện. **Chưa được nhóm duyệt.** | `009`, `011`, `015`, `019`, `024`, `035`, `037`, `040`. |
| `v1.0-agreed` | **Chưa tạo.** Chỉ lập sau khi đủ 5 bản guideline độc lập và họp nhóm. | Không ghi khống phiên bản hoặc người duyệt. |

> **Lưu ý thực hiện lab:** bản này là tài liệu AI hỗ trợ **so sánh, giải thích và biên tập**; thành viên mã 047 cần tự rà 40 ảnh, xác nhận/chỉnh các quyết định cá nhân, lưu bản cá nhân đúng vị trí và ghi cách đã kiểm AI trong `submission/individual_notes/047.md`. Gửi riêng trước khi so bản của các thành viên khác theo `RULES.md`. Không dùng bản dự thảo làm bằng chứng đã gán box hoặc đã chạy QC.
