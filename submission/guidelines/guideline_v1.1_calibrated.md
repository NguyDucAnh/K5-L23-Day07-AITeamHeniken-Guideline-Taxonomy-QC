# Guideline gán nhãn — v1.1 — TEAMAIHEINEKEN

- Phiên bản: v1.1
- Ngày: 2026-11-17
- Người viết / duyệt: Nhóm TEAMAIHEINEKEN

## 1. Mục tiêu

Gán bounding box cho người và phương tiện thật nhìn thấy trong ảnh đường phố để xây dựng dữ liệu huấn luyện mô hình phát hiện đối tượng. Mỗi người và mỗi phương tiện là một đối tượng riêng. Chỉ gán phần nhìn thấy; không suy đoán phần bị che khuất.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ / ranh giới |
|---|---|---|
| `car` | Ô tô con, gồm SUV và ô tô thật dùng trong giao thông hoặc diễu hành. | Ô tô trên xe chở ô tô vẫn là một đối tượng riêng nếu nhìn thấy rõ. |
| `truck` | Xe tải hoặc xe có thùng chở hàng riêng. | Xe tải chở ô tô được gán `truck`; các ô tô chở trên đó gán riêng `car`. |
| `van` | Xe thùng kín cỡ nhỏ. | Không dùng cho xe tải có thùng hàng riêng. |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ. | Mỗi xe là một box; không gộp xe với người ngồi trên xe. |
| `person` | Người đi bộ, lái xe hoặc người ngồi trên phương tiện. | Gán riêng từng người nhìn thấy, kể cả người đi xe máy. |
| `three_wheeler` | Phương tiện ba bánh có động cơ như tuk-tuk, xe lam, ba gác máy. | Không gán cùng phương tiện đó thêm một box `motorcycle`. |
| `bus` | Xe buýt hoặc xe khách cỡ lớn được thiết kế để chở nhiều hành khách. | Không dùng cho `van` hoặc xe tải có thùng hàng. |
| `bicycle` | Xe đạp chạy bằng sức người, có bàn đạp và không có động cơ. | Gán xe và người đi xe bằng hai box riêng; không dùng `motorcycle`. |
| `ignore_region` | Vùng bỏ qua, không phải một vật thể. Dùng cho vùng cục bộ không thể gán từng vật nhất quán. | Không dùng để che một đối tượng rõ ràng hoặc thay cho việc chọn lớp. |

Không thêm lớp ngoài các lớp trong bảng. Không ép một vật vào lớp gần giống nếu vật đó không thỏa định nghĩa lớp.

### Thuộc tính

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị che khuất (0%). |
| `occluded_level` | `1` | Bị che khuất 1–25%. |
| `occluded_level` | `2` | Bị che khuất 26–50%. |
| `occluded_level` | `3` | Bị che khuất trên 50% nhưng vẫn nhận diện được là đối tượng nào. |
| `truncated` | `no` / `yes` | Chọn `yes` khi vật bị mép ảnh cắt mất; nếu có, box chỉ ôm phần nằm trong ảnh. |
| `can_review` | tick / không | Tick khi còn phân vân nhưng vẫn có thể gán box; nếu không thể xác định đối tượng, không tự đoán. |
Nếu không thể nhận diện đối tượng do bị che quá nhiều, không tạo box. Ước lượng phần trăm che khuất theo phần diện tích vật bị vật khác che, không tính phần nằm ngoài khung ảnh.

## 3. Quy tắc vẽ box

1. Box chữ nhật ôm sát đường bao của phần nhìn thấy được; không mở rộng box qua vật che khuất để đoán hình dạng.
2. Gán box riêng cho từng người và từng phương tiện. Người ngồi trên xe có box `person` riêng; xe có box theo lớp phương tiện tương ứng. Box có thể chồng lấn.
3. Với xe chở xe, gán xe chở và từng xe được chở nhìn thấy rõ thành các đối tượng riêng.
4. Với vật có chiều cao box dưới 20 px, không gán box riêng; có thể dùng `ignore_region` cho cụm vật nhỏ hoặc vùng dày đặc không thể gán từng vật nhất quán. Không sửa các ngưỡng trong mục `rules` của taxonomy.
5. Không gán tượng, mannequin, đồ chơi, hình in/quảng cáo hoặc vật chỉ xuất hiện trong phản chiếu. Không gán mascot hay đạo cụ; người điều khiển nhìn thấy rõ vẫn được gán `person`.
6. Giữ nguyên ảnh không có đối tượng phù hợp; không tạo box giả.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở ô tô | R-01 | Gán `truck` cho xe chở và `car` riêng cho từng ô tô nhìn thấy rõ | Mỗi phương tiện thật là một đối tượng riêng. |
| `img_004.jpg`, `img_035.jpg` | Tuk-tuk / xe lam ba bánh | R-02 | Gán `three_wheeler`; người nhìn thấy gán `person` riêng | Đúng lớp ba bánh có động cơ, không gộp người với xe. |
| `img_005.jpg`, `img_006.jpg` | Nhiều người đi xe máy, chồng lấn | R-03 | Gán riêng từng `motorcycle` và từng `person` nhìn thấy | Box có thể chồng lấn; chỉ lấy phần người nhìn thấy. |
| `img_017.jpg`, `img_027.jpg` | Tượng có hình người / phương tiện | R-04 | Không gán tượng | Không phải người hoặc phương tiện thật đang được gán nhãn. |
| `img_019.jpg` | Xe dạng ô tô trong đoàn diễu hành, có người điều khiển và mascot | R-05 | Gán `car` nếu là phương tiện thật có người điều khiển; không gán mascot/đạo cụ | Phân biệt phương tiện thật với nhân vật hoặc đồ trang trí. |
| `img_020.jpg` | Người/phương tiện chỉ thấy qua gương | R-06 | Không gán hình phản chiếu | Phản chiếu không phải đối tượng trực tiếp trong cảnh. |
| `img_023.jpg` | Xe đạp hoặc xe đạp kéo | R-07 | Gán phần xe đạp là `bicycle`; gán người điều khiển riêng là `person` | Taxonomy v1.1 có lớp xe đạp riêng. Phần xe kéo không thuộc lớp nào thì không gán. |
| `img_033.jpg` | Mannequin và hình phản chiếu trong cửa kính | R-04, R-06 | Không gán mannequin hoặc hình phản chiếu | Không phải người thật trong cảnh. |
| `img_038.jpg` | Ô tô đồ chơi trong tủ trưng bày | R-04 | Không gán đồ chơi | Không phải phương tiện thật. |

## 5. QC và phiên bản

- Các ngưỡng IAA, IoU, lỗi theo ảnh và ngưỡng vật nhỏ giữ nguyên theo `submission/taxonomy.json`.
- Khi không chắc lớp hoặc box, dùng `can_review` nếu vẫn có thể gán; ghi vấn đề cấp ảnh nếu không thể tạo box.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Làm rõ lớp, cách gán người đi xe, xe ba bánh, phương tiện khác, vật bị che/cắt và các ca mô phỏng. | Thống nhất quy tắc trước lượt gán độc lập. |
| v1.1 | Thêm `bus`, `bicycle`; bỏ `other_vehicle`; giữ `ignore_region` cho vùng không thể gán từng vật nhất quán. | Tách rõ xe buýt và xe đạp, loại bỏ lớp phương tiện quá rộng nhưng vẫn giữ cơ chế bỏ qua của quy trình QC. |
