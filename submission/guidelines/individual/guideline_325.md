# Guideline gan nhan - v0 ca nhan - 325

- Phien ban: v0-ca-nhan
- Ngay: 2026-11-16
- Nguoi viet / duyet: 325 / chua duyet

## 1. Muc tieu

Gan bounding box cho cac doi tuong nguoi va phuong tien nhin thay trong anh
duong pho de phuc vu bai toan phat hien doi tuong. Nhan phai nhat quan giua
cac anh; khong suy dien them phan bi che, phan nam ngoai khung hoac doi tuong
chi ton tai trong phan tu, anh phan chieu hay anh in.

## 2. Lop va thuoc tinh

### Lop

| Lop | Dinh nghia | Vi du | Phan vi / de nham |
|---|---|---|---|
| `car` | O to con hoac SUV co than xe cho nguoi, ke ca xe co dang dac biet nhung van la xe that | O to trong `img_009`, `img_015`, `img_023` | Khong dung cho xe tai, xe van, xe mo hinh hoac xe do choi |
| `truck` | Xe co cabin va thung/san hang rieng, kich thuoc lon hoac dang xe tai | Xe cho nhieu xe trong `img_002`, xe cho hang trong `img_035` | Neu la xe thung kin nho thi dung `van`; phan hang khong tach thanh box |
| `van` | Xe thung kin nho hoac xe van, co than lien khoi va khong phai xe tai lon | Xe van trong `img_007`, `img_036`, `img_037` | Khong dung cho xe buyt, xe tai co thung hang rieng |
| `motorcycle` | Xe hai banh co dong co, mo to, scooter; box chi bao quanh phuong tien | `img_005`, `img_006`, `img_040` | Khong dung cho xe dap; xe ba banh/tuk-tuk duoc gan `motorcycle` neu khong co lop phu hop hon |
| `person` | Co the nguoi nhin thay, dang di bo, ngoi, dung, hoac dang lai/ngoi tren phuong tien | Nguoi di bo trong `img_014`, `img_034` | Nguoi in tren bien, tuong, mannequin va hinh phan chieu khong gan |
| `ignore_region` | Vung gom cac vat the/khuc canh khong the gan tung doi tuong mot cach tin cay | Dam dong rat day trong `img_021` | Khong dung de che mot vat ma van co the gan box rieng |

#### Quy tac chon lop

- Moi doi tuong that chi co mot lop. Neu khong chac giua `car` va `van`,
  chon theo dang than xe: van la than thung kin cao/lien khoi; car la xe con
  hoac SUV.
- Xe cho xe van la mot `truck` duy nhat cho toan bo phuong tien ke ca cabin,
  san va cac xe dang cho, khong gan box rieng cho tung xe nam tren san.
- Tuk-tuk/xe ba banh va phuong tien dang xe khong co lop rieng duoc gan
  `motorcycle` neu nhin ro than phuong tien; khong gan tung ghe, hang hoa hay
  nguoi khong nhin thay ro.
- Xe co hinh dang la xe nhung la mo hinh, tuong, do choi, xe trung bay, anh
  in hoac vat trung bay khong phai doi tuong that thi khong gan lop. Vi du:
  xe mo hinh trong `img_038`, xe trang tri/gia tuong trong `img_019`, xe co
  tinh chat tuong trong `img_027` va `img_028` khong gan box xe.
- Nguoi that van gan `person` du bi che mot phan. Mannequin trong `img_033`,
  tuong trong `img_017` va nguoi trong tranh/poster khong gan `person`.
- Xe trong guong/phan chieu khong gan lai. Vi du vung guong trong `img_020`
  chi la phan phan chieu, khong tao box cho xe hay nguoi trong do.
- Neu mot doi tuong nam mot phan trong `ignore_region`, chi gan phan co the
  xac dinh ro neu van dat nguong kich thuoc; khong gan chong len vung ignore.

### Thuoc tinh

#### `occluded_level`

Chi tinh phan doi tuong bi vat khac che, khong tinh phan bi cat boi mep anh.

- `0`: thay ro it nhat 90% dien tich doi tuong, khong co vat khac che dang ke.
- `1`: bi che tren 0% den 25% dien tich nhin thay duoc cua doi tuong.
- `2`: bi che tren 25% den 60%.
- `3`: bi che tren 60%, nhung van nhan biet duoc doi tuong va co the dat box.
- Neu khong con du thong tin de nhan biet doi tuong, khong gan box thay vi
  doan phan bi che.

#### `truncated`

- `no`: toan bo doi tuong nam trong khung anh, ke ca khi mot phan bi vat khac
  che.
- `yes`: bat ky phan nao cua doi tuong bi cat boi mot trong bon mep anh. Box
  keo den mep anh, khong mo rong ra ngoai anh.

#### `can_review`

- Dat `true` khi khong chac lop, ranh gioi box, muc occluded, hoac quyet dinh
  co gan mot ca ngoai pham vi hay khong.
- Dat `false` khi quy tac tren da cho ket qua ro rang.
- `can_review` khong thay the cho viec bo box: van gan box neu doi tuong dat
  nguong va ghi lai ca can xem.

## 3. Quy tac ve box

1. Gan box cho moi nguoi va phuong tien that co chieu cao box toi thieu 20 px.
   Vat nho hon nguong nay bo qua, khong gan box manh.
2. Box bao quanh phan nhin thay cua doi tuong, sat nhat co the; khong bao gom
   bong, mat duong, khoang trong hoac hang hoa khong thuoc than xe.
3. Doi tuong bi che van co mot box cho phan nhin thay. Khong tao box rieng cho
   phan bi che khong co bang chung.
4. Nguoi dang lai/ngoi tren xe:
   - gan mot box `person` cho toan bo co the nguoi nhin thay;
   - gan mot box phuong tien cho xe;
   - khong gom nguoi vao box xe va khong tach chan/tay thanh doi tuong.
5. Nguoi dung gan xe nhung la hai doi tuong tach roi thi gan hai box rieng.
6. Xe dang cho tren xe tai chi duoc gan rieng neu phan xe do la doi tuong that
   nhin ro va nam tren mat dat/duoc xem la phuong tien doc lap; trong ca `img_002`,
   uu tien gan box cho xe tai cho hang va khong gan nhung xe con xep tren san.
7. Vat nam ngoai mep anh: gan phan nhin thay, keo box den mep va dat
   `truncated=yes`.
8. Anh co nhieu vat gan nhau nhung van tach duoc bien thi tao box rieng. Chi
   dung `ignore_region` khi mat do/che lap khien khong the gan tung vat mot
   cach tin cay.
9. Khong gan box cho logo, bien bao, tranh, anh in, man hinh, guong, bong,
   phan chieu, tuong, mannequin, do choi hay xe mo hinh.
10. Xe chu/phan xe nam o sat camera chi gan neu nhin thay ro than xe trong anh;
    khong tu suy dien phan nam ngoai khung.

## 4. Ca kho

| Anh | Tinh huong | Rule ID | Quyet dinh | Ly do |
|---|---|---|---|---|
| `img_001` | Xe nho/xe tai cho hang o gan nguoi va do vat | `R-01` | Gan box phuong tien that va nguoi that; khong gan hang hoa rieng | Hang hoa khong phai mot lop trong taxonomy; tach nguoi neu bien ro |
| `img_002` | Xe tai cho nhieu xe | `R-02` | Gan mot box `truck` cho xe cho xe; khong gan cac xe tren san | Phuong tien van tai la doi tuong chinh, cac xe tren san khong du ro de gan doc lap |
| `img_003` | Nguoi bi vat/ganh hang che | `R-03` | Gan `person` phan nhin thay, gan `occluded_level` theo ty le che | Nguoi that van la doi tuong du bi che |
| `img_004` | Tuk-tuk/xe ba banh | `R-04` | Gan `motorcycle` cho tung phuong tien ro; gan nguoi tach rieng | Taxonomy khong co lop tuk-tuk; khong bo qua phuong tien that |
| `img_005` | Hai nguoi tren xe may | `R-05` | Gan box xe `motorcycle` va box `person` cho moi nguoi nhin thay | Nguoi va phuong tien la hai doi tuong khac nhau |
| `img_011` | Tuong/voi trang tri ngoai troi | `R-06` | Khong gan `person` hay lop xe cho tuong | Khong phai nguoi/phuong tien that |
| `img_014` | Nguoi di bo bi mo chuyen dong | `R-07` | Gan `person`, box theo bien doi tuong nhin thay; dat `can_review` neu ranh gioi khong ro | Mo anh khong phai ly do bo qua nguoi that |
| `img_017` | Tuong hinh nguoi | `R-06` | Khong gan `person` | Tuong khong phai nguoi that |
| `img_019` | Xe mau vang dang trinh dien/xe gia tuong | `R-08` | Gan `car` chi neu xac dinh la xe that; neu la mo hinh/trang tri thi bo qua va dat `can_review` neu con nghi ngo | Khong tao nhan cho vat the khong thuoc pham vi |
| `img_020` | Phan chieu trong guong | `R-09` | Khong gan doi tuong trong guong; chi gan vat that ngoai guong neu co | Phan chieu khong phai doi tuong doc lap trong anh |
| `img_021` | Dam dong xe may/nguoi | `R-10` | Gan cac doi tuong co bien ro; dat `ignore_region` cho vung qua chen neu khong tach duoc | Khong doan box cho tung vat khi khong co bien |
| `img_022` | Anh den/trang co vat nho va kho nhan dien | `R-11` | Chi gan vat dat nguong 20 px va co the nhan dien; vat con lai bo qua | Tranh box doan mo ho |
| `img_024` | Nhieu xe trong showroom, xe co dang cu | `R-12` | Gan `car` cho xe that nhin ro; khong gan mannequin/do vat trang tri | Dang cu khong lam doi tuong thanh lop moi |
| `img_027` | Xe co/tuong trung bay trong bai co | `R-08` | Khong gan `car` | Day la vat trung bay/trang tri, khong phai xe dang luu thong |
| `img_028` | Xe co va ngua trang tri | `R-08` | Khong gan `car`, khong gan `person` | Vat trung bay/trang tri khong thuoc muc tieu |
| `img_029` | Xe dap va nguoi tren duong | `R-13` | Gan `person`; xe dap khong co lop nen khong gan | Taxonomy chi co xe may va xe dong co |
| `img_033` | Mannequin trong cua hang | `R-06` | Khong gan `person` | Khong phai nguoi that |
| `img_035` | Xe cho hang va nguoi bi che/cat mep | `R-02`/`R-03` | Gan `truck`, `person` neu ro; dung `truncated=yes` khi cat mep | Tach box theo doi tuong, khong mo rong ngoai anh |
| `img_037` | Xe van/xe thung kin | `R-14` | Gan `van` cho moi xe van that | Than thung kin nho, khong phai `truck` |
| `img_038` | Ke trung bay nhieu xe mo hinh | `R-08` | Khong gan cac xe mo hinh | Do choi/mo hinh khong phai phuong tien that |
| `img_040` | Nguoi doi mu bao hiem gan xe may | `R-05` | Gan `person` va `motorcycle` rieng; `occluded_level` theo phan bi che | Mu va xe khong lam mat nhan nguoi |

Neu gap ca khong co trong bang tren, gan theo quy tac tong quat, dat
`can_review=true` tren box lien quan va ghi `image, region, note` vao
`submission/qc/image_issues.csv` neu khong the gan hop le.

## 5. QC va phien ban

### Nguong ap dung

- Kiem tra bang script: box khong vuot kich thuoc anh, lop/thuoc tinh phai nam
  trong taxonomy, chieu cao box toi thieu 20 px.
- IAA tham chieu: ghep box voi `IoU >= 0.5`; muc mong muon theo taxonomy la
  `iaa_iou=0.85`, `iaa_class=0.95`.
- Khong ket luan anh sach chi tu IAA: van phai xem anh goc vi ca nhom co the
  cung bo sot hoac cung hieu sai.
- Khi co bat dong, nguoi review khac nguoi gan nhan se chot theo guideline va
  ghi ma Rule ID, anh, box/vung va ket qua.

### Lich su thay doi

| Phien ban | Thay doi | Ly do |
|---|---|---|
| `v0-ca-nhan` | Viet lai dinh nghia lop, nguong thuoc tinh va quy tac ca bien | Nhan dien ca kho trong bo 40 anh, dac biet tuk-tuk, vat trang tri, phan chieu va xe cho xe |
