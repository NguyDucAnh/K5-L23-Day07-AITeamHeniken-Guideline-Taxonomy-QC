# Guideline gán nhãn — bản cá nhân v0.2 — 203

- Phiên bản: v0.2 cá nhân
- Ngày: 2026-11-17
- Người viết / duyệt: ChuVietHung-2A202603203
- Trạng thái: đề xuất cá nhân trước khi nhóm thống nhất
- Giả định: nhiệm vụ là vẽ bounding box (object detection) cho người và phương tiện trên ảnh đường phố. Các mục ghi *(đề xuất)* là phần bổ sung, cần nhóm xác nhận.

## 1. Mục tiêu

Gán bounding box cho người và phương tiện trong ảnh đường phố để huấn luyện mô hình phát hiện đối tượng dùng cho việc đếm và nhận dạng phương tiện, người đi bộ, người đi xe. Theo tôi, nhãn cần giúp mô hình phân biệt các loại phương tiện chính và không học nhầm tượng, đồ chơi hoặc hình phản chiếu thành vật thật.

Sai ở đâu thì nguy hiểm:

- Bỏ sót `person` và `motorcycle` là lỗi nặng nhất, vì ảnh đường phố thường có xe máy chạy dày và người ngồi chồng lên xe.
- Nhầm lớp giữa các xe nhiều bánh (`car`, `van`, `truck`, `bus`) ảnh hưởng ở mức vừa phải.
- Nhầm xe ba bánh với `motorcycle` làm mô hình học sai hình dạng, nên cần lớp `three_wheeler` riêng.
- Gán nhãn vật không thật (tượng, mannequin, đồ chơi, hình in, phản chiếu) làm mô hình học sai, cần tránh tuyệt đối.

Nguyên tắc chung: chỉ gán vật thật, đang hiện diện và nhìn thấy trực tiếp trong ảnh. Không đoán phần bị che. Không chắc thì gắn `can_review`, không tự tạo lớp mới.

## 2. Lớp và thuộc tính

### Lớp đề xuất

| Lớp | Định nghĩa cá nhân | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con và SUV, khoang kín liền thân, gầm thấp. | Ô tô đang chạy hoặc được chở trên xe tải . | `van`, xe bán tải (chưa có lớp riêng) |
| `truck` | Xe tải, xe có phần thùng hàng riêng hoặc giàn chở xe lớn hơn cabin. | Xe tải chở ô tô. | `van`, xe bán tải |
| `van` | Xe thùng kín cỡ nhỏ dùng chở người hoặc hàng. | Van cỡ nhỏ. | `car`, `truck` |
| `bus` | Xe buýt hoặc xe khách cỡ lớn, thân dài, chở nhiều người. | Xe buýt thành phố. | `van` |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ (xe số, tay ga, xe côn, xe điện). | Xe máy có người ngồi. | `bicycle`, xe ba bánh |
| `bicycle` | Xe đạp có bàn đạp, không có động cơ. | Xe đạp thường hoặc phần xe đạp của xe kéo . | `motorcycle` |
| `person` | Người thật nhìn thấy trực tiếp trong ảnh, đang đứng, đi, ngồi hoặc đi trên xe. | Người đi bộ, người ngồi trên xe máy, tài xế trong xe. | Tượng , mannequin, hình in, hình trong gương |
| `three_wheeler` | Phương tiện ba bánh có động cơ. | Tuk-tuk, xe lam. | `motorcycle` |
| `ignore_region` | Vùng có nhiều vật nhỏ hoặc chồng lấn, khó gán từng vật nhất quán. Mô hình đoán trong vùng này không bị tính là lỗi. | Cụm đối tượng quá nhỏ ở xa. | Không dùng cho một vật rõ ràng |

Vật không thuộc lớp nào thì tôi đề xuất không gán và ghi lại để nhóm thảo luận, thay vì ép vào lớp gần giống.

### Thuộc tính đề xuất

| Thuộc tính | Giá trị | Cách hiểu cá nhân |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| `occluded_level` | `1` | Bị che khoảng 1–25%. |
| `occluded_level` | `2` | Bị che khoảng 26–50%. |
| `occluded_level` | `3` | Bị che trên 50% nhưng vẫn nhận ra đối tượng. *(đề xuất)* Che trên 85% hoặc không nhận ra lớp thì không vẽ. |
| `truncated` | `no` / `yes` | `yes` nếu đối tượng bị cắt bởi mép ảnh. *(đề xuất)* Chỉ vẽ nếu phần còn lại từ 25% trở lên và nhận ra lớp. |
| `can_review` | `false` / `true` | `true` nếu tôi vẫn vẽ được box nhưng chưa chắc lớp, ranh giới hoặc thuộc tính. Ghi 1 dòng lý do. |

### Thuộc tính cấp ảnh *(đề xuất)*

| Thuộc tính | Giá trị | Cách hiểu cá nhân |
|---|---|---|
| `non_real_objects` | `false` / `true` | `true` khi ảnh có tượng, mannequin, đồ chơi, hình in hoặc vật phản chiếu dễ gây nhầm. |
| `face_visible` | `false` / `true` | `true` khi có ít nhất một mặt người rõ, dùng làm cờ làm mờ trước khi chia sẻ. |
| `child_present` | `false` / `true` | `true` khi có trẻ em, dùng làm cờ riêng tư. |
| `image_issue` | text | Ghi 1 dòng khi không xác định được đối tượng trong ảnh. |

## 3. Quy tắc vẽ box cá nhân

1. Vẽ box ôm sát phần đối tượng nhìn thấy, không chừa viền, không đoán phần nằm sau vật che. (P203-01)
2. Người và phương tiện là các đối tượng riêng. Người ngồi trên xe máy hoặc xe đạp có box `person` cho từng người (kể cả người ngồi sau, trẻ em); xe có box `motorcycle` hoặc `bicycle`. Các box được phép chồng nhau. (P203-02)
3. Xe tải chở xe: một box `truck` cho xe tải (ôm cả cabin và giàn), và một box `car` riêng cho mỗi xe được chở nếu còn nhìn thấy đủ để nhận diện. Gắn `occluded_level` theo phần bị giàn che. (P203-03)
4. Xe ba bánh gán `three_wheeler`, không gán thêm `motorcycle` cho cùng xe. Người lái và hành khách vẫn là `person` riêng. (P203-04)
5. Xe đạp kéo / xe đẩy: phần xe đạp gán `bicycle`, người điều khiển gán `person`. Phần kéo chưa có lớp, gắn `can_review` và đưa ra họp nhóm. (P203-05)
6. Đối tượng có chiều cao box dưới 20 px thì không gán riêng; vùng có nhiều vật nhỏ có thể đề xuất `ignore_region`. *(đề xuất)* Từ 3 vật nhỏ nằm sát nhau trở lên thì vẽ một `ignore_region` bao quanh. (P203-06)
7. Vật bị che hoặc bị cắt mép ảnh: chỉ vẽ phần nhìn thấy, gắn `occluded_level` theo bảng, vật chạm mép ảnh vẽ sát mép và gắn `truncated=yes`. (P203-07)
8. Không gán tượng, mannequin, đồ chơi, hình in. Gắn cờ ảnh `non_real_objects=true`. (P203-08)
9. Không gán vật chỉ xuất hiện trong gương, kính hoặc phản chiếu. (P203-09)
10. Nếu chưa rõ một ca có thuộc taxonomy hay không, ghi lại để họp nhóm; không tự tạo class trong lúc gán. Ca vẽ được nhưng còn phân vân thì dùng `can_review=true`. (P203-10)

## 4. Ca khó và ý kiến ban đầu

| Ảnh | Tình huống | Rule ID | Quyết định cá nhân | Lý do / điểm cần nhóm chốt |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở ô tô | P203-03 | Gán một box `truck` và box `car` riêng cho từng ô tô nhìn thấy rõ, gắn `occluded_level` theo phần bị giàn che. | Các xe vẫn là đối tượng thật độc lập. |
| `img_004.jpg`, `img_035.jpg` | Tuk-tuk / xe ba bánh | P203-04 | Đề xuất lớp `three_wheeler`; không gán thêm `motorcycle` cho cùng xe; tài xế gán `person`. | Hình dạng và công năng khác xe máy hai bánh. |
| `img_005.jpg`, `img_006.jpg` | Người đi xe máy chồng lấn | P203-02 | Gán riêng người và xe; cho phép box chồng nhau. | Mô hình cần phát hiện hai loại đối tượng. |
| `img_017.jpg`, `img_027.jpg` | Tượng giống người hoặc phương tiện | P203-08 | Không gán; gắn cờ ảnh `non_real_objects=true`. | Không phải đối tượng thật trong cảnh giao thông. |
| `img_020.jpg` | Người/xe trong gương | P203-09 | Tạm thời không gán; đưa ra họp nhóm. | Có nguy cơ đếm trùng đối tượng phản chiếu. |
| `img_023.jpg` | Xe đạp kéo / xe đẩy | P203-05 | Đề xuất gán phần xe đạp là `bicycle`, người điều khiển là `person`; phần kéo gắn `can_review`, cần nhóm chốt. | Không nên ép xe đạp thành `motorcycle`. |
| `img_038.jpg` | Ô tô đồ chơi | P203-08 | Không gán. | Không phải phương tiện thật. |

## 5. QC và phiên bản

- Giữ nguyên các ngưỡng trong `taxonomy.json.rules`, đặc biệt ngưỡng chiều cao 20 px.
- Ca có thể gán nhưng còn phân vân thì dùng `can_review=true`.
- Ca không thể xác định đối tượng thì ghi vấn đề cấp ảnh để nhóm thảo luận.
- *(đề xuất)* Kiểm tra lại 10% số ảnh của mỗi người gán, chọn ngẫu nhiên; ảnh có `can_review=true` luôn được kiểm tra.
- *(đề xuất)* Lỗi nặng: bỏ sót `person` hoặc `motorcycle` rõ ràng; gán nhãn vật không thật (tượng, đồ chơi, hình in, phản chiếu).
- Người gán chỉ gắn `can_review` và ghi 1 dòng lý do, không tự đổi quy tắc. Quyết định ca mơ hồ do nhóm chốt khi họp.
- Sau khi các thành viên hoàn tất bản cá nhân, nhóm so sánh quyết định và chốt guideline v1.0 chung.

Việc cần nhóm thống nhất:

1. Xe bán tải: tạo lớp riêng hay gộp vào `truck` hoặc `car`.
2. Ca phản chiếu (img_020): không gán hẳn hay có ngoại lệ.
3. Phần thùng kéo của xe đạp kéo (img_023).
4. Có thêm lớp `license_plate` và cờ làm mờ riêng tư hay không.
5. Ngưỡng `ignore_region` (số vật tối thiểu) và ngưỡng `truncated`.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 cá nhân | Đề xuất ban đầu về `bus`, `bicycle`, xe ba bánh, người đi xe và vật mô phỏng. | Chuẩn bị ý kiến cho buổi thống nhất nhóm. |
| v0.2 cá nhân | Bổ sung mục tiêu chi tiết, ví dụ/phản ví dụ cho từng lớp, thuộc tính cấp ảnh, Rule ID P203-01 đến P203-10, ngưỡng QC và danh sách việc cần nhóm chốt. | img_002, img_004, img_020, img_023. |