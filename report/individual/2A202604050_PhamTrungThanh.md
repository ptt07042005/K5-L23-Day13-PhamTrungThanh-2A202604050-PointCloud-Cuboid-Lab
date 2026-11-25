# Báo cáo Day 13 — Phạm Trung Thành, 2A202604050

## 1. Kết luận trước

- **Thông tin bài làm:** Mã số học viên `2A202604050`; task cá nhân `D13-2A202604050`; 10 frame được giao gồm `s01f13`, `s01f45`, `s02f11`, `s02f45`, `s03f08`, `s03f23`, `s04f40`, `s04f57`, `s05f02`, `s05f30`. Môi trường làm việc: Windows 11 (`10.0.22000`), trình duyệt Google Chrome, CVAT local phiên bản `2.74.1` chạy qua Docker tại `http://localhost:8080`. Đổi kiểm: chưa có bạn ngồi cạnh để đổi kiểm do làm bài trước buổi học.
- **Tóm tắt kết quả:** Tổng cộng đã gán **104 cuboid** dạng `Shape` trên 10 frame (87 `Car`, 5 `Pedestrian`, 5 `Cyclist`, 4 `Truck`, 3 `Bus`; gồm 84 cuboid trong vùng `≤ 30 m` và 20 cuboid trong vùng `30–50 m`, 0 cuboid ngoài `50 m`). Frame ít cuboid nhất là `s02f45` (**6 cuboid**) vì đây là đoạn đường thoáng có dải phân cách sắt và bồn hoa hai bên, mật độ phương tiện thực tế rất thưa (ngược lại frame đông nhất là `s03f08` với **14 cuboid**). Phiếu tự kiểm `selfcheck` gắn cờ **7/104 cuboid** (6 cảnh báo điểm sát ngoài mép do vật tĩnh kề bên và 1 cảnh báo đáy chìm `0,66 m` do bụi cây dải phân cách). Phần em tin nhất là hướng `yaw` của các xe trên làn chính (đã đối chiếu đủ 7 camera), phân loại class và độ cao đáy `z_bottom` trên mặt đường phẳng; phần còn yếu nhất là ước lượng chiều dài và ranh giới mép sau của các xe bị che khuất một phần (`occluded`) qua hàng cây dải phân cách cùng các xe thưa điểm ở tầm `35–48 m`.

Cách dùng CVAT trong bài lab:
1. **Đổi góc nhìn:** trong khung 3D chính, kéo chuột trái để xoay góc nhìn (xoay thẳng từ trên xuống để quét toàn cảnh `360°`), kéo chuột phải để dời khung, lăn con lăn để phóng to/thu nhỏ và nháy đúp vào vùng trống để trở về góc nhìn mặc định.
2. **Vẽ và chỉnh cuboid:** chọn biểu tượng cuboid ở thanh công cụ trái → chọn class → bấm **Shape** (không bấm Track) rồi nháy đúp vào cụm điểm của vật để đặt hộp; khi chọn hộp, chỉnh tâm, dài, rộng và góc xoay `yaw` ở khung **Top**, chỉnh độ cao và hạ đáy chạm đường ở khung **Side** và **Front** (không xoay ở **Side/Front** để tránh nghiêng hộp).
3. **Bật mũi tên hướng và xem chiếu camera:** bật **Appearance → Cuboid orientation** ở bảng bên phải để hiện vạch đỏ ngắn (`+X` của cuboid) chỉ về đầu xe; đối chiếu 7 ảnh camera (đổi `cam01`–`cam09` bằng biểu tượng bánh răng) để xác định loại vật và đầu/đuôi, sau đó chạy `python scripts/lab.py preview` (hoặc `sync`) để kiểm tra khung cuboid chiếu lên 7 ảnh camera trong thư mục `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f13-3d.png`, `t1-s01f13-top.png`, `t1-s01f45-3d.png`, `t1-s01f45-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f13` | Phía trước (`-y`) ở **cạnh trên**; `+x` (vạch cam) chỉ **sang trái**, `+y` (vạch xanh lá) chỉ **xuống dưới** (phía sau xe) | Phẳng; mặt đường ở `z ≈ -1,75 m` (thấp hơn LiDAR khoảng `1,75 m`) | Khoảng `45–50 m` (xa nhất đủ rõ để gán là xe `#9 Car` ở `35,91 m`; xa hơn `~48 m` phía sau còn cụm rất thưa) | `Car`, `Bus`, `Pedestrian`; vật tĩnh gồm dải phân cách cây bụi, nhà chờ xe buýt, hàng cây, cột đèn | Bên phải có 1 `Pedestrian` gần nhà chờ xe buýt (`9,50 m`, thấy trên `cam08`, `cam09`); phía sau cùng chiều có 3 `Car` (`20,85 m`, `29,55 m`, `35,91 m`, `cam09`); phía sau-trái bên kia dải phân cách có 1 `Car` ngược chiều (`19,13 m`, `cam05`, `cam09`) | Cụm điểm rời rạc của một xe ở rất xa phía sau (`> 45 m`) chỉ còn vài điểm mờ và một số cụm điểm rải rác phía sau hàng cây bên trái dải phân cách (`> 25 m`) |
| `s01f45` | Phía trước (`-y`) ở **cạnh trên**; `+x` (vạch cam) chỉ **sang trái**, `+y` (vạch xanh lá) chỉ **xuống dưới** (phía sau xe) | Phẳng; mặt đường ở `z ≈ -1,76 m` (thấp hơn LiDAR khoảng `1,76 m`) | Khoảng `40–45 m` (xa nhất đủ rõ để gán là xe buýt `#8 Bus` ở `35,79 m` phía trước) | `Car`, `Bus`, `Pedestrian`; dãy xe đạp dựng không người lái trên vỉa hè phải, giá treo biển phía trên, hàng cây hai bên | Bên phải gần vỉa hè có 2 `Pedestrian` (`8,59 m` và `11,39 m`, thấy trên `cam01`, `cam07`); phía sau cùng chiều có 2 `Car` (`10,17 m` và `13,54 m`, thấy trên `cam08`, `cam09`) | Vùng điểm chồng lấn giữa dãy xe đạp dựng sát nhau, gốc cây và người đứng sâu trên vỉa hè phải (`> 20 m`), cùng vài cụm điểm rất thưa phía trước tầm `> 40 m` bị đoàn xe gần che khuất |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết (rút từ `s01f13` và `s01f45`):
- **Chỉ camera cho biết:**
  1. Màu sắc, đèn xe và chi tiết bề mặt giúp phân biệt chính xác đầu và đuôi xe (ví dụ ở `s01f13`, `cam01`/`cam06` cho thấy xe `#1` là sedan xanh đậm thấy đèn hậu đi cùng chiều, xe `#3` là xe buýt trắng-đỏ phía trước).
  2. Ngữ cảnh và trạng thái người–phương tiện để phân loại đúng class (ví dụ ở `s01f45`, `cam07` cho thấy rõ 2 người đi bộ đứng tách biệt cạnh hàng xe đạp dựng không có người lái trên vỉa hè phải, tránh gán nhầm `Cyclist`).
  3. Đặc tính vật liệu hấp thụ tia như sơn tối màu, mặt kính cửa sổ (trên xe `#1` ở `s01f13`) và tán lá rậm trên dải phân cách che phần thân xe ở làn đối diện (`cam05`, `cam06`).
- **Chỉ LiDAR cho biết:**
  1. Tọa độ 3D và khoảng cách tuyệt đối theo mét trong toàn bộ góc quét `360°`, bao gồm cả vùng chuyển tiếp/góc mù giữa các hình quạt camera (ví dụ đo chính xác `s01f13 #1 Car` cách `5,22 m` và `#3 Bus` cách `29,00 m`).
  2. Kích thước hình học thật bằng mét (dài, rộng, cao) không bị méo phối cảnh 2D theo khoảng cách (như `s01f13 #3 Bus` dài `11,550 m`, rộng `2,600 m`, cao `3,050 m` dù ảnh trên `cam01` khá nhỏ).
  3. Cao độ mặt đường tương đối so với cảm biến (`z ≈ -1,75 m`) và hình dạng vùng bóng che hình học phía sau vật cản khi nhìn từ trên xuống.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f11` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Khu vực ngã tư đông phương tiện (`13 cuboid`); bên trái gần có xe con đô thị dáng hộp thân cao đi ngược chiều (`5,43 m`), phía sau-phải có 2 `Cyclist` qua vạch kẻ đường; mặt đường phía nhánh giao cắt sau hơi thấp hơn (`z_bottom` khoảng `-2,16 m` đến `-2,19 m`) |
| `s02f45` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Đoạn đường thoáng, dải phân cách sắt và bồn hoa hai bên nên ít vật nhất (`6 cuboid`); mặt đường dốc nhẹ xuống về phía trước (`z_bottom` từ `-1,34 m` phía sau giảm xuống `-2,38 m` ở `33,82 m` phía trước) |
| `s03f08` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Đường hẹp đông nhất (`14 cuboid`), có tường quây công trình liên tục sát cả hai bên tạo vệt tường thẳng đứng và che hoàn toàn vùng ngoài tường; nhiều xe tải nhẹ/van chở hàng (`Truck`) đi nối đuôi |
| `s03f23` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Ngay sau ngã tư có rào chắn bên phải, xuất hiện 3 xe con đỗ/chạy lệch góc (`yaw_deg` khoảng `1,0°–15,5°`) ở phía sau-phải và 1 người đi bộ trên vỉa hè; dải cây bên trái che một phần các xe ngược chiều |
| `s04f40` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Đường nhiều làn thoáng, rào kim loại bên trái và tường công trình bên phải; có xe SUV trắng rất sát phía sau-phải (`4,96 m`, `4589 điểm`) và chứa cuboid xa nhất trong bài (`#9 Car` ở `48,37 m`) |
| `s04f57` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Đường hơi dốc xuống về phía trước (`z_bottom` khoảng `-2,05 m` đến `-2,10 m` phía trước so với `-1,63 m` đến `-1,68 m` phía sau); có 1 xe ba bánh điện có người lái (`Cyclist`) ở làn ngược chiều bên trái |
| `s05f02` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Có xe tải thùng lớn (`Truck`, cao `3,056 m`) đi ngay phía sau (`12,28 m`); hàng cây bụi rậm và cao trên dải phân cách trái che thân dưới các xe ngược chiều và đẩy ước lượng mặt đất cục bộ gần xe `#8` lên cao |
| `s05f30` | Cạnh trên (`-y` hướng lên; `+x` trái, `+y` dưới) | Mặt đường hơi dốc xuống về phía trước (`z_bottom` khoảng `-1,91 m` đến `-2,12 m` phía trước so với `-1,32 m` đến `-1,70 m` phía sau); dải cây xanh bên trái che một phần 4 xe con ngược chiều (`#8`–`#11`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f13`, `idx = 1` | `Car` | Đúng: vạch đỏ hướng về đầu xe cùng chiều (`yaw_deg = 269,0°`, đối chiếu `cam01`, `cam06`) | Chạm đường (`z_bottom = -1,751 m`, `ground_gap_m = 0,00 m`) | Có: đã căn sát 6 mặt trên Top/Side/Front và kiểm ảnh `preview` | Đúng khoảng Sedan: `4,650 × 1,850 × 1,494 m` (chuẩn `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f13`, `idx = 3` (thẻ UI `#5`) | `Bus` | Đúng: vạch đỏ chỉ về đầu xe buýt đang hơi chuyển làn trái (`yaw_deg = 284,3°`, đối chiếu `cam01`, `cam03`) | Chạm đường (`z_bottom = -1,731 m`, `ground_gap_m = 0,02 m`) | Có: đã căn sát 6 mặt trên Top/Side/Front và kiểm ảnh `preview` | Đúng khoảng Xe buýt thường: `11,550 × 2,600 × 3,050 m` (chuẩn `10,0–12,5 × 2,50–2,60 × 3,00–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f45`, `idx = 11` (thẻ UI `#12`) | `Pedestrian` | Đúng: vạch đỏ hướng theo chiều di chuyển dọc vỉa hè về phía trước (`yaw_deg = 270,0°`, đối chiếu `cam01`, `cam07`) | Chạm đường (`z_bottom = -1,755 m`, `ground_gap_m = 0,01 m`) | Có: đã căn sát 6 mặt trên Top/Side/Front và kiểm ảnh `preview` | Đúng khoảng Người lớn: `0,650 × 0,650 × 1,680 m` (chuẩn `0,4–0,9 × 0,5–0,9 × 1,50–1,95 m`) |

- **Vòng hiệu chuẩn:** Chưa thực hiện vẽ 3 cuboid trên task chung `D13-HC485087` để gửi ghép lớp (đã tạo task chung để quan sát các frame hiện tượng Bước 4 và sẽ bổ sung hiệu chuẩn khi học lý thuyết/thực hành trên lớp). Khi gán 10 frame cá nhân, em tuân thủ chặt quy tắc guideline: chỉ dùng `Shape`, bật `occluded` cho xe bị che khuất một phần, không gán xe hai bánh dựng bên đường không có người lái, và kiểm kích thước bằng bảng 4.3 cùng 7 ảnh camera.
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f13`, phía sau xe `#9 Car` trên làn cùng chiều (cách khoảng `45–55 m`, thấy rất nhỏ trên `cam09`): cụm điểm xe con chỉ còn vài điểm rời rạc (dưới `15–20%` thể tích cuboid, sát giới hạn `50 m`) — `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`.
  - `s01f45`, khu vực vỉa hè bên phải (`x ≈ -6 m` đến `-9 m`, `cam07`): nhiều xe đạp dựng không có người lái (bỏ qua theo guideline) và một vài người đứng sâu sau hàng cây (`> 20 m`) bị che gần hết thân dưới — `LOW_VISIBILITY`.
  - `s03f08`, các xe đi sâu phía trước và phía sau trên hai làn hẹp (`> 30 m`) bị hàng xe tải và xe con phía trước che gần như toàn bộ thân xe, không đủ `15–20%` thể tích để xác định tâm hộp — `LOW_VISIBILITY`.
  - `s05f02` và `s05f30`, một số xe ở làn ngược chiều bên kia dải phân cách cây rậm (`x > 10 m`, cách `> 30 m`) chỉ lộ vài điểm nóc xe phía trên tán cây, khó nhận diện chắc chắn loại xe và kích thước — `LOW_VISIBILITY` / `UNCERTAIN_CLASS`.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Các nguyên nhân nêu dưới đây đều là **giả thuyết** (không khẳng định chắc chắn).

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` / `t4-1-bong-che.png` / **nguồn: task chung `D13-HC485087`** | **Giả thuyết:** xe buýt lớn chạy rất sát bên trái (khoảng `4 m`) đã chắn toàn bộ chùm tia laser chiếu về góc sau-trái của nó nên tia không đi xuyên qua được | Phía sau thân xe buýt xuất hiện một vùng hình nón đen lớn gần như không có điểm nào, trong khi hai bên ngoài góc khuất các vòng quét mặt đường vẫn rất dày điểm | Vùng trống sau vật che là **chưa quan sát được**, không phải đường trống; với vật bị che một phần nhưng còn suy luận được thì bật `occluded` và kéo cuboid bao trọn cả phần suy luận được, không cắt cụt hộp |
| 2. Thưa ở xa | `s01f13` / `t4-2-thua-o-xa.png` / **nguồn: task cá nhân `D13-2A202604050`** | **Giả thuyết:** góc giữa các tia quét dọc và ngang của LiDAR 40 tia nở rộng theo khoảng cách `r`, làm số tia cắt trúng bề mặt vật ở xa giảm mạnh | Cụm điểm khoanh đỏ của ô tô phía sau xe `#9 Car` (xa hơn `45 m`) chỉ còn vài điểm rời rạc trên 1–2 đường quét, mất hoàn toàn đường bao thân xe; xe `#9 Car` ở `35,91 m` cũng chỉ còn `51 điểm` so với `2718 điểm` ở `5,22 m` | Áp dụng quy tắc `15–20%` thể tích: chỉ vẽ khi thấy khoảng `15–20%` thể tích hoặc đủ cơ sở suy luận từ camera + bảng 4.3; với cụm quá thưa trong vòng khoanh đỏ thì bỏ qua và ghi `LOW_VISIBILITY` / `UNCERTAIN_SCOPE` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f13` / `t4-3-phan-xa-thap.png` / **nguồn: task cá nhân `D13-2A202604050`** | **Giả thuyết:** lớp sơn tối màu (xanh đậm) hấp thụ phần lớn năng lượng tia hồng ngoại và bề mặt kính cửa sổ gây phản xạ gương lệch khỏi đầu thu, làm tín hiệu phản xạ yếu hơn ngưỡng ghi nhận | Dù chiếc sedan xanh đậm `s01f13 #1 Car` ở rất gần (`5,22 m`, xem `cam01`, `cam06`), vùng kính bên, kính sau và nóc xe có khoảng trống thưa điểm và cường độ yếu hơn rõ rệt so với phần cản dưới, viền bánh hoặc vỉa hè sáng màu cùng cự ly | Giữ nguyên nhân ở mức giả thuyết; không được bóp nhỏ cuboid theo phần điểm dày mà bỏ lọt vùng kính/nóc bị khuyết — phải đối chiếu camera và kiểm đủ 6 mặt ở Top/Side/Front để bao trọn thân xe (`4,650 × 1,850 × 1,494 m`) |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / **nguồn: task chung `D13-HC485087`** | **Giả thuyết:** tia laser bị đa phản xạ (multipath) qua bề mặt nhẵn như kính/thân xe hoặc mặt đường trước khi quay lại cảm biến, làm thời gian bay đo dài hơn thực tế và đẩy tọa độ tính ra xuống sâu dưới mặt đất | Khi xoay nghiêng khung 3D thấy một cụm khoảng `46 điểm` (khoanh đỏ) ở phía sau-trái, cách khoảng `29 m`, nằm ở cao độ `z ≈ -13,5 m` (thấp hơn mặt đường xung quanh khoảng `11 m`), không tương ứng vật nào trên camera | Đây là sai lệch hình học chắc chắn: theo guideline **không gán thành vật**, không kéo đáy cuboid của vật gần đó xuống bao lấy cụm điểm lạ này, và chỉ nêu nguyên nhân dưới dạng giả thuyết |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu trích nguyên từ `submission/D13-2A202604050-bangA.csv` và cột `points_in`, `ground_gap_m` từ `submission/D13-2A202604050-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f13`, `idx = 1`, `Car` | `-1,004 m` | `1,494 m` | `-1,751 m` | `5,22 m` | `5,32 m` | `269,0°` | Chạm đường (lệch `-0,051 m` so với `-1,7 m`; `ground_gap_m = 0,00 m`) |
| `s01f13`, `idx = 3`, `Bus` | `-0,206 m` | `3,050 m` | `-1,731 m` | `29,00 m` | `29,00 m` | `284,3°` | Chạm đường (lệch `-0,031 m` so với `-1,7 m`; `ground_gap_m = 0,02 m`) |
| `s01f45`, `idx = 11`, `Pedestrian` | `-0,915 m` | `1,680 m` | `-1,755 m` | `11,39 m` | `11,43 m` | `270,0°` | Chạm đường (lệch `-0,055 m` so với `-1,7 m`; `ground_gap_m = 0,01 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s03f08, idx = 1 (Car)` tại `(x = 3,192 m; y = 2,654 m; z = -1,016 m)` có `dist_m = 4,15 m` và `r = 4,27 m` (`2333 điểm`). Chênh lệch `r - dist_m = 0,12 m` do `|z| = 1,016 m` đáng kể khi vật ở rất gần.
  - **Xa nhất:** `s04f40, idx = 9 (Car)` tại `(x = -5,702 m; y = 48,037 m; z = -0,981 m)` có `dist_m = 48,37 m` và `r = 48,38 m` (`48 điểm`). Ở khoảng cách lớn, thành phần `z` rất nhỏ so với khoảng cách ngang nên `r ≈ dist_m` (chênh `0,01 m`).
- **(b) So kích thước với bảng 4.3:**
  - **Người đi bộ / xe hai–ba bánh:** `s01f45, idx = 11 (Pedestrian)` có `l = 0,650 m`, `w = 0,650 m`, `h = 1,680 m`, cả 3 chiều đều nằm trong khoảng Người lớn (`0,4–0,9 m × 0,5–0,9 m × 1,50–1,95 m`). Xét thêm trường hợp ngoài khoảng: `s04f57, idx = 4 (Cyclist)` (xe ba bánh điện có người lái) có `l = 2,380 m`, `w = 1,050 m`, `h = 1,680 m`; so với mục Xe ba bánh có người (`2,5–3,5 m × 1,00–1,50 m × 1,50–2,00 m`), chiều dài ngắn hơn ngưỡng dưới `0,120 m` vì đây là mẫu xe ba bánh điện đô thị cỡ gọn (khớp thực tế `234 điểm` LiDAR và ảnh `cam05`, `cam06`).
  - **Xe con:** `s01f13, idx = 1 (Car)` có `l = 4,650 m`, `w = 1,850 m`, `h = 1,494 m`, nằm trọn trong khoảng Sedan (`3,8–5,0 m × 1,70–1,90 m × 1,40–1,60 m`). Xét trường hợp có chiều ngoài khoảng Sedan: `s02f11, idx = 4 (Car)` có `l = 3,750 m`, `w = 1,720 m`, `h = 1,820 m`, chiều dài ngắn hơn ngưỡng dưới Sedan `0,050 m` và chiều cao vượt ngưỡng trên Sedan `0,220 m` vì đây là dòng xe đô thị mini dáng hộp/kei car có thân ngắn nhưng trần cao (khớp cụm `3258 điểm` ở `5,43 m` và ảnh `cam06`).
- **(c) Hướng `yaw_deg` của xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước: `s01f13, idx = 1 (Car)` có `yaw_deg = 269,0°` (và `s01f45, idx = 3 (Car)` có `yaw_deg = 267,0°`), rất gần mốc `270°`.
  - Xe ngược chiều: `s02f11, idx = 4 (Car)` có `yaw_deg = 90,5°` và `s02f45, idx = 4 (Car)` có `yaw_deg = 89,5°`, rất gần mốc `90°`.

Mật độ theo khoảng cách (4 cuboid `Car` không bị che khuất ở 4 khoảng `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f13`, `idx = 1`, `Car` | `5,22 m` | `2718 điểm` |
| `s01f45`, `idx = 3`, `Car` | `10,24 m` | `1012 điểm` |
| `s01f13`, `idx = 4`, `Car` | `20,85 m` | `316 điểm` |
| `s05f30`, `idx = 6`, `Car` | `36,83 m` | `80 điểm` |

- **Quy luật mật độ và so với slide bài giảng:** Khi khoảng cách tăng gấp `2 lần` (từ `10,24 m` lên `20,85 m`), số điểm giảm còn khoảng `31%` (gần `1/3`, từ `1012 điểm` xuống `316 điểm`); khi khoảng cách tăng gấp `3,6 lần` (từ `10,24 m` lên `36,83 m`), số điểm chỉ còn khoảng `7,9%` (`80 điểm`), gần theo quy luật tỉ lệ nghịch với bình phương khoảng cách (`1/r²`). So với đường cong trung vị xe con trên slide bài giảng (`1968 điểm` ở `0–10 m`, `718 điểm` ở `10–20 m`, `214 điểm` ở `20–30 m`, `72 điểm` ở `30–40 m`, `19 điểm` ở `60–80 m`), 4 mốc trên bám rất sát dạng suy giảm này và nhỉnh hơn trung vị nhẹ vì các xe đều nằm ở đầu dải khoảng cách và không bị che khuất (`occluded = 0`).
- **Tự kiểm sau `preview` / `selfcheck`:**
  - **Hai cuboid đã sửa:**
    1. `s05f02, idx = 2 (Truck)`: sau khi xem ảnh chiếu `cam09` (`work/xem/D13-2A202604050-s05f02.jpg`), thấy thành thùng xe tải hơi vượt nhẹ ra ngoài hộp nên đã nới `width` từ `2,496 m` lên `2,610 m` và chỉnh tâm `x` từ `-0,144 m` sang `-0,087 m` (`2526 điểm`, `ground_gap_m = 0,01 m`).
    2. `s05f02, idx = 8 (Car)`: ban đầu `selfcheck` cảnh báo đáy thấp hơn mặt đường `0,91 m`; sau khi đối chiếu `Side/Front` và `cam01`/`cam06`, đã nâng `z_bottom` từ `-1,620 m` lên `-1,383 m` (điều chỉnh `height` từ `1,520 m` thành `1,200 m` và `x` từ `10,601 m` sang `10,349 m`) để ôm gọn phần thân xe nhìn thấy phía trên tán cây, giảm độ lệch cờ từ `0,91 m` xuống `0,66 m`.
  - **7 cảnh báo giữ nguyên kèm lý do:**
    - Năm cuboid `Pedestrian` (`s01f13 #6`: `216 điểm`, `s01f45 #11`: `51 điểm`, `s01f45 #12`: `86 điểm`, `s02f45 #6`: `37 điểm`, `s03f23 #9`: `27 điểm` "sát ngay ngoài mép"): giữ nguyên vì kiểm trên `Top/Side/Front` và ảnh camera cho thấy các điểm sát mép thuộc vật tĩnh đứng kề bên (vách nhà chờ xe buýt, hàng xe đạp dựng, gốc cây, cột đèn, lan can); kích thước hộp đã chuẩn Người lớn và đáy sát đất (`ground_gap_m = 0,01–0,07 m`), nếu nới rộng sẽ bao nhầm vật tĩnh.
    - `s01f13 #7 Car` (`10 điểm sát ngay ngoài mép`): giữ nguyên vì xe nằm sau dải phân cách cây bụi (`occluded = 1`), 10 điểm sát mép là tán lá bụi cây cạnh thân xe.
    - `s05f02 #8 Car` (`đáy thấp hơn mặt đường 0,66 m`): giữ nguyên sau khi đã chỉnh ở trên vì dải phân cách ngay sát xe có hàng cây bụi dày và cao, khiến phép ước lượng mặt đường cục bộ của `selfcheck` lấy nhầm cao độ tán cây thay vì mặt đường nhựa ở làn ngược chiều.
- **Đổi kiểm:** Chưa thực hiện đổi kiểm với bạn ngồi cạnh do hoàn thành bài trước buổi học và chưa được ghép cặp đổi kiểm. Em đã tự đối chiếu toàn bộ các số trích trong mục 5 (ví dụ `s01f13 #1 Car`: `z_bottom = -1,751 m`, `dist_m = 5,22 m`; `s03f08 #1 Car`: `dist_m = 4,15 m`; `s04f40 #9 Car`: `dist_m = 48,37 m`) khớp hoàn toàn với `submission/D13-2A202604050-bangA.csv` và `submission/D13-2A202604050-tukiem.csv`.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Ý nghĩa của nhãn nhìn từ phía model:**
  - Nhãn 3D đo bằng **mét** thay vì pixel vì model nhận thức 3D và lập kế hoạch chuyển động cho xe tự hành cần biết tọa độ vật lý, khoảng cách thực và kích thước bất biến quanh xe để tính khoảng hở an toàn và thời gian va chạm, trong khi tọa độ pixel trên ảnh 2D bị biến dạng theo khoảng cách, tiêu cự và góc phối cảnh.
  - Vật thưa điểm cần **quy tắc riêng** (ngưỡng `15–20%` thể tích, bật `occluded` và không đoán bừa khi thiếu cơ sở) vì nếu gán tùy tiện các cụm quá ít điểm hoặc tự ý cắt nhỏ hộp theo điểm thấy được, dữ liệu huấn luyện sẽ bị nhiễu nặng về tâm, kích thước và nhãn nền, làm model học sai phân bố hình học vật thể và tăng dự đoán sai ở tầm xa.
- **(b) Cách dùng AI:**
  - **Công cụ:** Google Gemini / Antigravity.
  - **Câu hỏi chính:** Tra cứu quy ước hệ tọa độ VF (`+x` trái, `+y` sau, `-y` trước), cách chạy các lệnh `scripts/lab.py` trong môi trường `.venv` trên Windows, giải thích các cột của Bảng A và ý nghĩa cảnh báo `selfcheck`, cùng kiểm tra đủ mục và đơn vị đo trong báo cáo theo rubric.
  - **Cách dùng kết quả:** Dùng làm hướng dẫn tham khảo và danh sách kiểm tra chéo; toàn bộ cuboid, ảnh chụp và con số trong báo cáo đều được đối chiếu trực tiếp trên CVAT, ảnh chiếu `work/xem/` từ `lab.py sync` và xác minh lại bằng `scripts/check_submission.py`.
- **(c) Điều còn chưa chắc:**
  - Với các xe ở làn ngược chiều bị dải phân cách cây bụi che khuất phần lớn thân dưới (như `s01f13 #7`, `s05f02 #8`, `s05f30 #8–#11`), việc xác định thật chính xác chiều dài thân xe và vị trí đầu/đuôi chỉ từ phần nóc trên point cloud vẫn còn độ không chắc chắn nhất định.
  - Trên các đoạn đường hơi dốc (`s02f45`, `s04f57`, `s05f30`), cao độ mặt đường thay đổi theo khoảng cách nên cần luôn kiểm tra lại mặt đất cục bộ ở khung `Side/Front` thay vì chỉ dựa vào mốc cố định `-1,7 m`.
  - Với người đi bộ đứng sát công trình/vật tĩnh trên vỉa hè và các xe ở tầm xa `35–48 m`, việc tách ranh giới điểm sát mép giữa vật và nền vẫn cần thêm kinh nghiệm thực tế để giữ độ khớp hình học cao nhất.
