# Báo cáo Day 13 — Đinh Thành Vinh, 2A202603044

## 1. Kết luận trước
- **Thông tin chung**: Mã số sinh viên `2A202603044`, task `D13-2A202603044`, gồm 10 frame: `s01f21`, `s01f43`, `s02f20`, `s02f53`, `s03f02`, `s03f32`, `s04f31`, `s04f51`, `s05f00`, `s05f45`. Môi trường thực hành: Windows 11, trình duyệt Google Chrome, CVAT phiên bản `2.82.1` (chạy qua Docker local). MSSV bạn ngồi cạnh: `[Điền MSSV bạn ngồi cạnh]`.
- **Tổng kết kết quả**: Toàn bộ 10 frame đã vẽ tổng cộng `58` cuboid (hoặc `[cập nhật tổng số mới nếu bạn vẽ thêm]`), trong đó hai frame `s03f32` và `s04f51` có ít cuboid nhất (`3` cuboid) do mật độ phương tiện tham gia giao thông trên đoạn đường tại thời điểm đó thưa thớt hơn các frame còn lại. Công cụ tự kiểm (`selfcheck`) cờ `17` cuboid có cảnh báo trên tổng số `58` cuboid (chủ yếu là cảnh báo điểm mặt đường/vật thể lân cận nằm sát ngoài mép hộp). Phần tôi tin tưởng nhất là xác định đúng class và hướng di chuyển (mũi tên `yaw`) của các phương tiện ở gần; phần còn yếu nhất là căn chỉnh độ cao đáy hộp chạm sát mặt đường ở các xe cỡ lớn (`Bus`, `Truck`) và ước lượng kích thước các vật thể ở xa bị thưa điểm.

**Cách dùng CVAT 3D:**
- **Đổi góc nhìn**: Giữ chuột phải (hoặc dùng phím `U`, `O`) để xoay góc nhìn 3D, giữ chuột giữa để kéo dời khung hình (pan), lăn con lăn chuột để phóng to/thu nhỏ và kết hợp các phím `I`, `J`, `K`, `L` để di chuyển góc nhìn từ trên xuống hoặc ngang tầm xe.
- **Vẽ và chỉnh cuboid**: Chọn công cụ vẽ cuboid, chọn đúng `Label` và bấm `Shape` (nháy đúp vào điểm đặt hộp hoặc chọn hộp để hiện khung điều khiển), sau đó kéo các chấm đỏ ở 3 khung nhìn `Top`, `Side`, `Front` (hoặc nhập trực tiếp thông số `L, W, H` ở bảng `DETAILS`) để hộp ôm khít vật thể và đáy chạm mặt đường.
- **Bật mũi tên hướng và xem trên camera**: Tích chọn mục `Cuboid orientation` trong bảng `Appearance` ở góc dưới bên phải để hiện mũi tên xanh dương chỉ đầu xe; sau khi bấm `Save`, chạy lệnh `python scripts/lab.py preview` (hoặc `sync`) để kiểm tra khung cuboid chiếu đè lên 7 ảnh camera trong thư mục `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f21-3d.png`, `t1-s01f21-top.png`, `t1-s01f43-3d.png`, `t1-s01f43-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f21` | Phía trước xe hướng lên cạnh trên của ảnh (ngược trục `Y`); trục `+x` (đỏ) chỉ sang bên trái, trục `+y` (xanh lá) chỉ xuống cạnh dưới (phía sau xe). | Mặt đường tương đối phẳng, thấp hơn tâm LiDAR khoảng `1,7 m`. | Khoảng `45–50 m` | Ô tô con (`Car`), xe buýt (`Bus`), dải phân cách, hàng cây ven đường. | Bên trái (`+x`) có 1 xe con trắng (`cam05`) và 1 xe buýt đỏ phía trước; bên phải (`-x`) có 1 xe con đỗ sát lề; phía sau (`+y`) có các xe nối đuôi. | Các cụm điểm mờ trên vỉa hè và tán cây ở khoảng cách trên `45 m` quá thưa, không đủ hình dạng để nhận diện. |
| `s01f43` | Phía trước xe hướng lên cạnh trên của ảnh (ngược trục `Y`); trục `+x` chỉ sang trái, trục `+y` chỉ xuống dưới (phía sau xe). | Mặt đường phẳng, thấp hơn tâm LiDAR khoảng `1,7 m`. | Khoảng `45–50 m` | Ô tô con (`Car`), xe hai bánh (`Cyclist`), cột đèn, cây xanh. | Hai bên hông (`cam05`, `cam06`) có xe con và xe hai bánh di chuyển song song; phía sau (`cam07`) có xe con bám đuôi. | Một số cụm điểm nhỏ sát lề đường phía xa (`> 40 m`) bị lẫn vào bóng cây/tường rào nên chưa xác định rõ loại vật. |

**Đối chiếu giữa Camera và LiDAR (từ hai frame `s01f21` và `s01f43`):**
- **3 điều chỉ camera cho biết**: (1) Màu sắc, đèn tín hiệu, biển báo và vạch kẻ phân làn đường; (2) Phân biệt chính xác vật thể nhỏ ở xa hoặc sát lề đường là người đi bộ, xe máy hay trụ biển báo; (3) Nhận diện các vật thể bị khuất hoàn toàn khỏi tia LiDAR hoặc nằm trên bề mặt kính/tối màu ít phản xạ.
- **3 điều chỉ LiDAR cho biết**: (1) Tọa độ không gian 3D chính xác (`x, y, z`) và khoảng cách thực tế tính bằng mét từ xe tới vật thể; (2) Kích thước hình học thật (`dài × rộng × cao`) không bị biến dạng bởi phối cảnh gần-xa như ảnh 2D; (3) Quan sát toàn cảnh 360° từ trên xuống (`Top view`) để biết chính xác vị trí tương đối giữa các xe trên từng làn đường.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f20` | Cạnh trên của ảnh (hướng `-Y`) | Giao thông đông ở cả hai chiều, nhiều xe bị che khuất một phần thân phía sau. |
| `s02f53` | Cạnh trên của ảnh (hướng `-Y`) | Có 2 xe chạy ngược chiều ở làn bên trái rõ nét qua `cam02` và cụm điểm LiDAR. |
| `s03f02` | Cạnh trên của ảnh (hướng `-Y`) | Các điểm quét mặt đường tạo thành các vòng tròn đồng tâm rất rõ quanh tâm xe. |
| `s03f32` | Cạnh trên của ảnh (hướng `-Y`) | Đường thoáng, ít phương tiện gần, có 1 xe buýt kích thước dài nổi bật. |
| `s04f31` | Cạnh trên của ảnh (hướng `-Y`) | Mật độ xe cao (`8` cuboid), có xe tải/buýt lớn gây bóng che vùng phía sau nó. |
| `s04f51` | Cạnh trên của ảnh (hướng `-Y`) | Ít phương tiện (`3` cuboid), xuất hiện nhiều điểm nhiễu từ tán cây hai bên đường. |
| `s05f00` | Cạnh trên của ảnh (hướng `-Y`) | Có xe tải (`Truck`) kích thước lớn ở làn bên cạnh, điểm tập trung dày ở mặt hông. |
| `s05f45` | Cạnh trên của ảnh (hướng `-Y`) | Nhiều xe con và xe tải di chuyển gần nhau, mép điểm bánh xe sát mặt đường. |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f21`, `#1` | `Car` | Đúng (mũi tên xanh dương chỉ thẳng đầu xe) | Có (chạm sát vạch mặt đường) | Đã kiểm đủ 3 góc nhìn và ảnh `work/xem/` | Có (`L ≈ 4,39 m`, `W ≈ 2,13 m`, `H ≈ 1,55 m`) |
| (b) xe buýt hoặc xe tải | `s01f21`, `#2` | `Bus` | Đúng (mũi tên chỉ thẳng đầu xe buýt) | Đã kéo nâng đáy lên sát mặt đường | Đã kiểm đủ 3 góc nhìn và ảnh `work/xem/` | Có (`L ≈ 13,61 m`, `W ≈ 3,1–3,2 m`, `H ≈ 3,1 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f43`, `#6` | `Cyclist` | Đúng (mũi tên chỉ theo hướng xe chạy) | Có (chạm sát mặt đường) | Đã kiểm đủ 3 góc nhìn và ảnh `work/xem/` | Hơi rộng nhẹ (`W ≈ 1,58–1,74 m` do bao gồm cả tay lái và người lái) |

- **Vòng hiệu chuẩn & Vật bỏ qua**: Trong quá trình hiệu chuẩn ban đầu, cuboid của tôi bị xoay ngang trục `90°` làm hoán đổi `length` và `width` (mũi tên `yaw` chĩa sang hông xe thay vì đầu xe); sau khi đối chiếu guideline, cả lớp thống nhất quy tắc cạnh có mũi tên xanh dương luôn là chiều dài `length` và chỉ thẳng về phía đầu xe. Các vật thể bỏ qua không gán nhãn gồm những cụm điểm quá thưa ở khoảng cách trên `45–50 m` hoặc bị khuất sau xe lớn, không đủ cơ sở phân biệt giữa `Car` và vật cản tĩnh (`LOW_VISIBILITY` / `UNCERTAIN_CLASS`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
*(Lưu ý: Bạn lưu 4 ảnh chụp màn hình tương ứng vào `report/individual/images/` với tên `phenom-1-occlusion.png`, `phenom-2-sparse.png`, `phenom-3-low-reflect.png`, `phenom-4-noise.png`)*

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s04f31` / `phenom-1-occlusion.png` / Task cá nhân | Xe buýt/xe tải lớn nằm gần cảm biến chặn toàn bộ tia laser chiếu tới các vật thể phía sau nó. | Một vùng nón tối màu (mất hoàn toàn điểm quét) kéo dài phía sau thân xe buýt. | Không được đoán mù vật bị khuất hoàn toàn; chỉ gán các xe còn nhìn thấy rõ ít nhất một mặt hông/đầu và đối chiếu camera. |
| 2. Thưa ở xa | `s02f20` / `phenom-2-sparse.png` / Task cá nhân | Mật độ tia quét LiDAR phân kỳ theo khoảng cách, ở tầm `35–50 m` chỉ còn vài tia quét trúng mục tiêu. | Chiếc ô tô ở xa chỉ hiện lên bằng `15–30` điểm rời rạc hình chữ L mờ nhạt thay vì nguyên khối thân xe. | Dựa vào ảnh camera (`cam03`) để xác nhận đúng loại xe và giữ kích thước hộp theo chuẩn trung bình của class đó thay vì bóp hộp quá nhỏ. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f21` / `phenom-3-low-reflect.png` / Task cá nhân | Bề mặt xe sơn đen bóng hoặc cửa kính hấp thụ/khúc xạ tia laser khiến tia phản hồi yếu hơn ngưỡng thu. | Phần thân xe tối màu và kính lái bị rỗng điểm, chỉ thấy rõ phần biển số, đèn phản quang và bánh xe. | Vẫn phải kéo bao trùm toàn bộ kích thước thực của thân xe dựa trên khung viền ngoài và ảnh camera, không cắt cụt phần kính/nóc xe. |
| 4. Điểm lạ | `s04f51` / `phenom-4-noise.png` / Task cá nhân | Phản xạ đa đường (multipath) từ mặt đường ướt/kim loại hoặc bụi, tán cây ven đường. | Xuất hiện các cụm điểm nhiễu lơ lửng trên không hoặc rải rác bất thường không khớp với vật thể nào trên camera. | Bỏ qua hoàn toàn các điểm nhiễu lơ lửng, không kéo đáy hoặc nóc cuboid trùm lên các điểm rác này để tránh sai lệch kích thước. |

## 5. Số từ cuboid của tôi (Bước 5)
*(Mở file `submission/D13-2A202603044-bangA.csv` và `submission/D13-2A202603044-tukiem.csv`, chọn 3 cuboid bất kỳ để điền các cột số tương ứng vào bảng dưới)*

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f21`, `#1`, `Car` | `[xem bangA]` | `[xem bangA]` | `[tính z - h/2]` | `[xem bangA]` | `[tính r]` | `[xem bangA]` | Chạm đường (độ lệch `< 0,3 m`) |
| `s02f53`, `#1`, `Car` | `[xem bangA]` | `[xem bangA]` | `[tính z - h/2]` | `[xem bangA]` | `[tính r]` | `[xem bangA]` | Chạm đường (độ lệch `< 0,3 m`) |
| `s05f00`, `#4`, `Truck` | `[xem bangA]` | `[xem bangA]` | `[tính z - h/2]` | `[xem bangA]` | `[tính r]` | `[xem bangA]` | `[Ghi chạm đường hoặc chìm ~0,30 m]` |

- **(a) Cuboid gần nhất và xa nhất**:
  - Cuboid gần nhất: `[frame, idx, class]` có `dist_m = [xem bangA] m` và `r = [tính r] m`.
  - Cuboid xa nhất: `[frame, idx, class]` có `dist_m = [xem bangA] m` và `r = [tính r] m`.
