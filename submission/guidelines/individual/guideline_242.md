# Guideline gán nhãn — bản cá nhân v0.1 — 242

- Phiên bản: v0.1 cá nhân
- Ngày: 2026-11-17
- Người viết / duyệt: thành viên 242
- Trạng thái: đề xuất cá nhân trước khi nhóm thống nhất

## 1. Mục tiêu

Gán bounding box cho người và phương tiện trong ảnh đường phố để huấn luyện mô hình phát hiện đối tượng. Theo tôi, nhãn cần giúp mô hình phân biệt các loại phương tiện chính và không học nhầm tượng, đồ chơi hoặc hình phản chiếu thành vật thật.

## 2. Lớp và thuộc tính

### Lớp đề xuất

| Lớp | Định nghĩa cá nhân | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con và SUV. | Ô tô đang chạy hoặc được chở trên xe tải. | `van` |
| `truck` | Xe tải, xe có phần thùng hàng riêng. | Xe tải chở ô tô. | `van`, xe bán tải |
| `van` | Xe thùng kín cỡ nhỏ dùng chở người hoặc hàng. | Van cỡ nhỏ. | `car`, `truck` |
| `bus` | Xe buýt hoặc xe khách cỡ lớn chở nhiều người. | Xe buýt thành phố. | `van` |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ. | Xe máy có người ngồi. | `bicycle` |
| `bicycle` | Xe đạp có bàn đạp, không có động cơ. | Xe đạp thường hoặc phần xe đạp của xe kéo. | `motorcycle` |
| `person` | Người thật nhìn thấy trực tiếp trong ảnh. | Người đi bộ hoặc ngồi trên xe. | Tượng, mannequin, hình in |
| `three_wheeler` | Phương tiện ba bánh có động cơ. | Tuk-tuk, xe lam. | `motorcycle` |
| `ignore_region` | Vùng có nhiều vật nhỏ/chồng lấn, khó gán từng vật nhất quán. | Cụm đối tượng quá nhỏ ở xa. | Không dùng cho một vật rõ ràng |

Vật không thuộc lớp nào thì tôi đề xuất không gán và ghi lại để nhóm thảo luận, thay vì ép vào lớp gần giống.

### Thuộc tính đề xuất

| Thuộc tính | Giá trị | Cách hiểu cá nhân |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che. |
| `occluded_level` | `1` | Bị che khoảng 1–25%. |
| `occluded_level` | `2` | Bị che khoảng 26–50%. |
| `occluded_level` | `3` | Bị che trên 50% nhưng vẫn nhận ra đối tượng. |
| `truncated` | `no` / `yes` | `yes` nếu đối tượng bị cắt bởi mép ảnh. |
| `can_review` | `false` / `true` | `true` nếu tôi vẫn vẽ được box nhưng chưa chắc lớp, ranh giới hoặc thuộc tính. |

## 3. Quy tắc vẽ box cá nhân

1. Vẽ box ôm sát phần đối tượng nhìn thấy, không đoán phần nằm sau vật che.
2. Người và phương tiện là các đối tượng riêng. Người ngồi trên xe máy có box `person`; xe có box `motorcycle`.
3. Xe tải chở xe: gán xe tải và từng xe được chở nếu còn nhìn thấy đủ để nhận diện.
4. Đối tượng có chiều cao box dưới 20 px thì không gán riêng; vùng có nhiều vật nhỏ có thể đề xuất `ignore_region`.
5. Không gán tượng, mannequin, đồ chơi, hình in hoặc vật chỉ xuất hiện trong gương/phản chiếu.
6. Nếu chưa rõ một ca có thuộc taxonomy hay không, ghi lại để họp nhóm; không tự tạo class trong lúc gán.
7. Xe nhìn qua kính trong suốt: đề xuất không gán; cần phân biệt vật nhìn trực tiếp với ảnh phản chiếu.
8. Người và phương tiện có box cao dưới 20 px: không gán riêng theo ngưỡng taxonomy.
9. Với đám đông dày đặc, đề xuất dùng `ignore_region`; không tự tạo lớp mới khi chưa được nhóm chốt.
10. Với một hàng xe máy chồng lấn, cần nhóm chốt khi nào có thể tách và nhận diện từng xe, nhất là khi chỉ thấy phần đuôi.

## 4. Ca khó và ý kiến ban đầu

| Ảnh | Tình huống | Rule ID | Quyết định cá nhân | Lý do / điểm cần nhóm chốt |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở ô tô | P242-01 | Gán một box `truck` và box `car` riêng cho từng ô tô nhìn thấy rõ. | Các xe vẫn là đối tượng thật độc lập. |
| `img_016.jpg`, `img_040.jpg` | Tuk-tuk / xe ba bánh | P242-02 | Đề xuất lớp `three_wheeler`; không gán thêm `motorcycle` cho cùng xe. | Hình dạng và công năng khác xe máy hai bánh. |
| `img_003.jpg`, `img_008.jpg` | Người đi xe máy chồng lấn | P242-03 | Gán riêng người và xe; cho phép box chồng nhau. | Mô hình cần phát hiện hai loại đối tượng. |
| `img_017.jpg`, `img_019.jpg`, `img_027.jpg` | Tượng giống người hoặc phương tiện | P242-04 | Không gán. | Không phải đối tượng thật trong cảnh giao thông. |
| `img_032.jpg` | Xe nhìn thấy được qua kính | P242-05 | Tạm thời không gán; đưa ra họp nhóm. | Cần thống nhất cách phân biệt đối tượng nhìn trực tiếp qua kính với hình phản chiếu. |
| `img_022.jpg` | Xe máy có gắn thêm một xe có chỗ ngồi bên cạnh | P242-06 | Đề xuất nhóm xem xét có cần lớp riêng hay không; không tự thêm lớp khi gán. | Cấu hình này chưa được taxonomy cá nhân mô tả đủ rõ. |
| `img_038.jpg` | Ô tô đồ chơi | P242-07 | Không gán. | Không phải phương tiện thật. |
| `img_010.jpg` | Xe đạp có gắn thêm quầy bán hoa quả | P242-08 | Chỉ gán phần xe đạp là `bicycle`; không gán quầy. | Quầy không thuộc các lớp mục tiêu. |

## 5. QC và phiên bản

- Giữ nguyên các ngưỡng trong `taxonomy.json.rules`, đặc biệt ngưỡng chiều cao 20 px.
- Ca có thể gán nhưng còn phân vân thì dùng `can_review=true`.
- Ca không thể xác định đối tượng thì ghi vấn đề cấp ảnh để nhóm thảo luận.
- Sau khi các thành viên hoàn tất bản cá nhân, nhóm so sánh quyết định và chốt guideline v1.0 chung.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 cá nhân | Ghi nhận đề xuất về `bus`, `bicycle`, xe ba bánh, người đi xe và vật mô phỏng. | Chuẩn bị ý kiến của thành viên 242 cho buổi thống nhất nhóm. |
