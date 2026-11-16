# Guideline gán nhãn — v1.0 cá nhân — 353

- Phiên bản: v0.1_353
- Ngày: 2026-11-16
- Người viết / duyệt: 353 / chưa duyệt chéo
- Loại nhãn: bounding box trên ảnh đường phố; công cụ CVAT, xuất COCO 1.0

## 1. Mục tiêu

Gán một box cho từng **người thật** và **phương tiện thật** nhìn thấy trong 40 ảnh đường phố để huấn luyện mô hình phát hiện đối tượng. Nhãn phải cho biết đúng lớp, phần vật thể nhìn thấy và mức bị che/cắt. Không gán vật không thật (tượng, hình in, ma-nơ-canh, đồ chơi), vật chỉ xuất hiện trong gương/phản chiếu, hoặc vật quá nhỏ; các vùng đó dùng `ignore_region` khi cần ghi rõ vùng không gán.

Sai nguy hiểm nhất là bỏ sót người/xe thật, gán box lên vật giả hoặc ảnh phản chiếu, gộp nhiều xe thành một box, và dùng box bao cả hàng hóa/đồ vật không thuộc thân xe. Các ảnh không có đối tượng hợp lệ vẫn phải giữ trong Task và COCO với 0 annotation.

## 2. Lớp và thuộc tính

### Lớp

Mỗi đối tượng thật thuộc đúng một lớp.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV hoặc ô tô con cổ/đang trưng bày nhưng là xe thật. | Xe con trên đường; các xe con trên xe chở ô tô trong `img_002`. | Xe tải/xe van; ô tô đồ chơi `img_019`, `img_038`; xe chỉ là hình/ảnh. |
| `truck` | Xe tải hoặc xe chuyên chở có cabin và thùng/sàn hàng riêng, kể cả xe chở ô tô. | Xe tải ở `img_001`, xe chở ô tô ở `img_002`, xe tải xanh `img_036`. | Xe van kín; hàng hóa trên thùng không phải box riêng nếu không phải người/xe. |
| `van` | Xe nhỏ/thùng kín, thân liền hoặc khoang hàng kín, không có thùng hàng hở riêng. | Xe van trắng ở `img_037`. | Xe tải có thùng hở; SUV/ô tô con. |
| `motorcycle` | Xe máy, mô tô hoặc xe ba bánh cơ giới kiểu tuk-tuk; box ôm phần xe, không ôm người ngồi trên xe hay hàng hóa. | Xe máy `img_005`, tuk-tuk `img_004` và `img_035`. | Xe đạp; người đi bộ; xe đồ chơi. |
| `person` | Người thật đang đi bộ, đứng, ngồi hoặc lái/ngồi trên phương tiện. | Người bán hàng `img_003`, người đi xe máy `img_005`, người qua đường `img_014`. | Tượng `img_017`, ma-nơ-canh `img_033`, hình in, người trong ảnh/phản chiếu. |
| `ignore_region` | Vùng không gán từng vật: cụm vật quá nhỏ, vùng vật giả/phản chiếu hoặc vùng không thể tách đáng tin cậy. | Cụm xe quá nhỏ ở xa; vùng phản chiếu gương `img_020`. | Không dùng để che một vật thật đủ lớn chỉ vì khó nhìn; vật thật phải được gán box. |

Tuk-tuk được quy về `motorcycle` vì là phương tiện ba bánh cơ giới nhỏ; không tạo lớp mới. Trên xe chở ô tô, gán `truck` cho xe chở và một `car` riêng cho từng ô tô thật nhìn thấy (`img_002`). Hàng hóa, biển quảng cáo, thú/đồ vật không thuộc taxonomy không có box riêng.

### Thuộc tính

| Thuộc tính | Các mức cố định | Cách quyết định |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: thấy gần như toàn bộ, bị che <10%; `1`: che 10–30%; `2`: che >30% nhưng vẫn nhận ra lớp; `3`: chỉ còn phần nhỏ/không đủ chắc chắn, đánh dấu `can_review`. |
| `truncated` | `no`, `yes` | `yes` nếu box chạm/cắt bởi bất kỳ mép ảnh nào và phần vật thể còn tiếp tục ngoài ảnh; nếu chỉ bị che bởi vật khác thì `no`. |
| `can_review` | tick / không tick | Tick khi không chắc vật là thật, lớp hoặc ranh giới box; dùng cho người/reviewer xem lại, không dùng thay cho quyết định loại vật giả đã rõ. |

Nếu vật bị che nhưng phần nhìn thấy vẫn xác định được, vẫn gán box quanh toàn bộ phần vật thể có thể suy ra trong ảnh và đặt mức che phù hợp; không kéo box vào vùng không có bằng chứng.

## 3. Quy tắc vẽ box

1. Chỉ gán đối tượng **thật** thuộc `car`, `truck`, `van`, `motorcycle`, `person`. Box ôm sát đường bao nhìn thấy của từng vật, không lấy thêm nền, bóng, ánh sáng hoặc hàng hóa rời.
2. Mỗi người và mỗi phương tiện là một box riêng. Người ngồi trên xe nhận thêm box `person`; box xe không gộp người. Xe chở xe nhận box cho xe chở và từng xe con thật nhìn thấy trên sàn/thùng.
3. Với xe có hàng hóa: box phương tiện chỉ bao thân phương tiện; không bao tải, sọt, rau hoặc đồ chất cao nếu chúng không phải bộ phận cố định của xe. Quy tắc này áp dụng cho xe bán hàng `img_001` và tuk-tuk chở hàng `img_035`.
4. Với tuk-tuk, gán một box `motorcycle` quanh thân xe và khoang xe nhìn thấy; người lái/hành khách là `person` riêng. Không gán riêng mái che, thùng hàng hay xe đạp đi kèm.
5. Với vật bị cắt mép, vẽ box sát phần nhìn thấy đến đúng mép ảnh và đặt `truncated=yes`. Với vật bị che, không nối box qua vật che nếu không có bằng chứng; đặt `occluded_level` theo tỷ lệ phần bị che.
6. Vật có chiều cao box <20 px không gán box riêng; nếu có nhiều vật nhỏ hoặc vùng không thể tách thì dùng một `ignore_region`. Vật cao từ 20 px trở lên nhưng mờ vẫn gán nếu nhận diện được, đồng thời đặt `can_review` nếu chưa chắc.
7. Không gán tượng, tượng sáp, ma-nơ-canh, đồ chơi, mô hình, hình in, tranh, ảnh trên màn hình, bóng hoặc phản chiếu. Các ví dụ không gán: tượng ở `img_017`, xe đồ chơi/xe diễu hành ở `img_019`, tượng xe ở `img_027–028`, ma-nơ-canh ở `img_033`, xe đồ chơi trong cửa hàng `img_038`, và hình trong gương `img_020`. Nếu vùng này gây nhầm lẫn cho người gán, khoanh `ignore_region`.
8. Không gán xe/ người chỉ là ảnh phản chiếu trong kính, gương hoặc mặt nước (`img_020`); nếu vật thật nằm ngoài vùng phản chiếu và nhìn thấy thì gán vật thật đó như bình thường.
9. Với ảnh nhòe do chuyển động (`img_014`, `img_015`), vẫn gán người/xe thật khi còn nhận ra được hình dạng và lớp; box ôm vùng nhìn thấy, không mở rộng theo vệt nhòe. Nếu không thể xác định là vật thật hoặc lớp, tick `can_review`.
10. Khi chưa chắc giữa hai lớp, chọn lớp theo định nghĩa trên và tick `can_review`; không tự tạo lớp mới. Khi không thể phân biệt từng vật trong một cụm, không dùng một box lớp thật bao cả cụm; dùng `ignore_region` cho phần không tách được.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe chở nhiều ô tô | R-01 | 1 `truck` cho xe chở + 1 `car` cho từng ô tô thật nhìn thấy | Mỗi phương tiện thật là một đối tượng; không gộp các xe con vào box xe tải. |
| `img_003` | Người bị xe đạp/hàng che một phần | R-02 | Gán `person` nếu còn nhận ra người; đặt `occluded_level=1/2` | Che khuất không biến người thật thành vật loại bỏ. |
| `img_004`, `img_035` | Tuk-tuk, có mái/hàng hóa | R-03 | Gán `motorcycle` cho tuk-tuk; người là `person` riêng; không bao hàng rời | Tuk-tuk được quy về lớp gần nhất; box phải ổn định theo thân xe. |
| `img_005`, `img_006`, `img_021`, `img_030`, `img_040` | Nhiều người đi xe máy, có chồng lấp | R-04 | Mỗi xe và mỗi người nhìn thấy nhận box riêng; dùng `occluded_level` | Không gộp đám đông thành một box. |
| `img_014`, `img_015` | Người/xe bị nhòe chuyển động | R-05 | Gán nếu nhận ra vật thật; giữ box theo hình nhìn thấy, tick `can_review` nếu không chắc | Nhòe không phải phản chiếu hay vật giả. |
| `img_017`, `img_019`, `img_027`, `img_028`, `img_033`, `img_038` | Tượng, đồ chơi, ma-nơ-canh, mô hình | R-06 | Không gán lớp thật; dùng `ignore_region` khi cần | Đây không phải đối tượng thật trong phạm vi mô hình. |
| `img_020` | Vật trong gương/ảnh phản chiếu | R-07 | Không gán vật chỉ xuất hiện trong phản chiếu | Tránh nhân đôi vật thể và học nhầm hình ảnh. |
| `img_001` | Xe tải có người và hàng hóa chất cao | R-08 | Gán `truck`, box riêng cho người; không bao toàn bộ hàng hóa vào box xe | Box thể hiện phương tiện, không phải khối hàng. |
| `img_037` | Xe van kín cạnh xe khác | R-09 | Xe kín là `van`; xe tải có thùng riêng là `truck` | Phân lớp dựa trên cấu trúc thân xe, không dựa màu/kích thước. |
| `img_009`, `img_010`, `img_016`, `img_031`, `img_034` | Cụm đông hoặc vật ở xa | R-10 | Gán vật cao >=20 px nếu tách được; phần nhỏ không tách được dùng `ignore_region` | Ngưỡng kích thước giúp hai người không gán tùy ý. |

Nếu một ca chưa thể quyết định theo các rule trên, đánh dấu `can_review` trên box liên quan hoặc ghi `image, region, note` vào nhật ký vấn đề cấp ảnh; không tự bỏ qua một vật thật đủ lớn.

## 5. QC và phiên bản

- Ngưỡng kiểm tra box: chiều cao tối thiểu `20 px`; IoU ghép box `>=0.5`; IAA mục tiêu IoU trung bình `>=0.85` và cùng lớp `>=95%` trên các cặp box ghép 1–1.
- Với box cao từ `50 px`, sai lệch IoU chấp nhận tối thiểu là `0.85`; box nhỏ hơn `50 px` dùng ngưỡng `0.7`. Không có cặp box ghép thì ghi `N/A`, không tự coi là đạt.
- QC phải kiểm cả lỗi bỏ sót, box thừa trên vật giả/phản chiếu, sai lớp, sai thuộc tính và box quá rộng/hẹp. Release chỉ khi duyệt đủ 40/40 ảnh và tỷ lệ ảnh lỗi `<=2%`; một ảnh lỗi trong lô 40 ảnh đã là `2.5%`, nên chưa Release.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v1.0-individual | Viết lại taxonomy, thuộc tính, ngưỡng và rule R-01–R-10; xác định tuk-tuk, xe chở xe, vật giả, phản chiếu, vật nhỏ và nhòe | Quan sát toàn bộ `img_001`–`img_040`, đặc biệt các ca trong bảng mục 4 |

Các thay đổi sau khi họp nhóm phải tăng phiên bản và ghi rõ ảnh/rule bị ảnh hưởng; nếu thay đổi cách chọn vật, lớp, box hoặc thuộc tính thì phải rà lại toàn bộ lô.
