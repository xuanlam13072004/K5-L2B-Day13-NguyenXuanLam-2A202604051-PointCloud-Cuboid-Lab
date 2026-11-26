# Báo cáo Day 13 — Nguyễn Xuân Lam, 2A202604051

## 1. Kết luận trước
- Mã số: 2A202604051 · Task CVAT: `D13-2A202604051`.
- 10 frame: s01f02, s01f51, s02f25, s02f53, s03f33, s03f52, s04f01, s04f55, s05f08, s05f33.
- Môi trường: Windows (PowerShell), trình duyệt Chrome, CVAT server 2.81.0 (cvat-sdk 2.83.0). Ngồi cạnh: 2A202604059.

Tôi vẽ **70 cuboid**: s01f02 có 12 hộp, s01f51 có 6, s04f01 có 12, s05f08 có 10; sáu frame s02f25, s02f53, s03f33, s03f52, s04f55, s05f33 mỗi frame 5 hộp (ít nhất) vì trong các frame này có ít vật ở gần và có cụm điểm rõ. Lần `sync` đầu `selfcheck` cờ 10/70 cuboid; sau khi xem lại và sửa, lần `sync` cuối còn **2/70** cuboid có cảnh báo, cả hai tôi giữ nguyên có lý do (mục 5). Phần tôi tin nhất: hướng (yaw), vì tôi đã đối chiếu đầu xe trên ảnh camera và kiểm lại trên ảnh `preview`. Phần còn yếu nhất: vật xa, thưa điểm, vì ít điểm nên khó căn hộp sát vật.

Cách dùng CVAT:
- Đổi góc nhìn: kéo chuột trái để xoay, kéo chuột phải để dời, lăn để phóng to; xoay khung 3D tới khi nhìn thẳng xuống để xem toàn cảnh từ trên xuống.
- Vẽ cuboid: nút cuboid → chọn label → **Shape** → nháy đúp lên cụm điểm; chỉnh tâm, dài, rộng, yaw ở Top; chỉnh đáy và chiều cao ở Side/Front (không xoay ở Side/Front).
- Hướng và kiểm trên ảnh: bật Appearance → Cuboid orientation để căn vạch đỏ ngắn về đầu xe; CVAT của lớp không chiếu cuboid lên ảnh camera nên tôi dùng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Ảnh: `t1-s01f02-3d.png`, `t1-s01f02-top.png`, `t1-s01f51-3d.png`, `t1-s01f51-top.png` (ảnh top có đánh dấu phía trước −y, +x, +y).

Hệ trục VF: gốc ở tâm LiDAR, x sang trái, y ra sau, z lên; phía trước xe là −y. Tôi xác định phía trước bằng cách đối chiếu vật trong ảnh cam01 với cụm điểm trên khung 3D.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f02 | Phía trước (−y) ở cạnh trên; +x chỉ sang trái; +y chỉ xuống dưới | Phẳng; thấp hơn LiDAR ~1,7 m | ~30–40 m | Xe con, xe buýt/xe tải, xe máy/xe đạp, người đi bộ | Xe con phía sau | Không có |
| s01f51 | Phía trước (−y) ở cạnh trên; +x chỉ sang trái; +y chỉ xuống dưới | Phẳng; thấp hơn LiDAR ~1,7 m | ~20–30 m | Xe con, xe buýt/xe tải, xe máy/xe đạp, người đi bộ | Xe con phía sau | Không có |

Chỉ camera cho biết: (1) màu xe, màu áo; (2) đèn tín hiệu, biển báo, vạch kẻ đường; (3) người trên xe hai bánh đang lái hay đang dắt xe.
Chỉ LiDAR cho biết: (1) khoảng cách tới vật theo mét; (2) kích thước thật của vật (dài, rộng, cao theo mét); (3) độ cao mặt đường (z ≈ −1,7 m) và vật nằm ở góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f25 | Cạnh trên | |
| s02f53 | Cạnh trên | |
| s03f33 | Cạnh trên | |
| s03f52 | Cạnh trên | Xe làn đối diện bị dải phân cách có hàng cây che phần dưới |
| s04f01 | Cạnh trên | |
| s04f55 | Cạnh trên | |
| s05f08 | Cạnh trên | |
| s05f33 | Cạnh trên | |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f51, idx 4 | Car | Có | Có | Có | Có |
| (b) xe buýt hoặc xe tải | s05f08, idx 9 | Truck | Có | Có | Có | Có (chiều rộng tính cả gương chiếu hậu) |
| (c) người đi bộ hoặc xe hai bánh | s04f01, idx 7 | Pedestrian | Có (theo hướng đi) | Có | Có | Có |

Vật bỏ qua vì không đủ cơ sở: s02f25, một xe phía trước bên trái, bị hàng rào phân cách che và hơi xa nên chỉ còn ít điểm; loại nghi vấn LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân là **giả thuyết**, chưa kiểm chứng.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f52 / `t4-1-bong-che.png` / task của tôi `D13-2A202604051` | Giả thuyết: hàng cây và dải phân cách chặn tia laser nên tia không tới được phía sau | Phía sau dải phân cách có hàng cây chỉ còn vài điểm; phần dưới chiếc xe ở làn đối diện gần như không có điểm | Bật occluded cho vật bị che; cuboid bao cả phần suy luận được. Vùng trống sau vật che là chưa quan sát được, không phải đường trống |
| 2. Thưa ở xa | s04f55 / `t4-2-thua-o-xa.png` / task của tôi | Giả thuyết: các tia laser toả ra xa nhau dần theo khoảng cách nên càng xa càng ít tia trúng vật | Một xe cách khoảng 40 m chỉ còn rất ít điểm | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc suy luận được; không đủ cơ sở thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s04f01 / `t4-3-phan-xa-thap.png` / task của tôi | Giả thuyết: bề mặt màu đen hấp thụ bớt tia laser nên ít tia phản xạ về LiDAR | Xe màu đen có ít điểm hơn xe màu trắng | Ghi là giả thuyết, không khẳng định nguyên nhân; vẫn vẽ theo điểm thấy được và ảnh camera nếu đủ cơ sở |
| 4. Điểm lạ | s02f25 / `t4-4-diem-la.png` / task của tôi | Giả thuyết: có thể là lá cây (cành cây vươn ra trên cao), chưa kiểm chứng | Cụm khoảng 5–6 điểm lơ lửng trên không, không thuộc vật nào | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Số lấy từ `submission/D13-2A202604051-bangA.csv` và `submission/D13-2A202604051-tukiem.csv` sau lần `sync` cuối.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s04f01, 1, Car | −0.791 m | 1.836 m | −1.709 m | 4.70 m | 4.77 m | 269.4° | Chạm đường |
| s01f02, 3, Pedestrian | −0.780 m | 1.639 m | −1.599 m | 13.95 m | 13.97 m | 89.2° | Chạm đường |
| s03f33, 3, Bus | −0.008 m | 3.515 m | −1.765 m | 50.07 m | 50.07 m | 268.8° | Chạm đường |

Cuboid gần nhất: s04f01, idx 1, Car — `dist_m` 4.70 m, r 4.77 m. Xa nhất: s03f33, idx 3, Bus — `dist_m` 50.07 m, r 50.07 m. Người đi bộ s01f02, idx 3 có kích thước l/w/h = 0.872/0.632/1.639 m, nằm trong khoảng người lớn ở bảng 4.3; xe con s01f02, idx 1 có kích thước 4.344/1.842/1.434 m, nằm trong khoảng sedan/hatchback. Yaw xe con s01f02, idx 1 là 266.5° (gần 270°); xe con s02f25, idx 2 là 89.9° (gần 90°).

Với hai xe tải, hộp bao cả gương chiếu hậu theo quy tắc "bao cả phần chìa ra" của guideline, nên chiều rộng hộp lớn hơn chiều rộng thân xe trong bảng 4.3 (2,40–2,60 m).

Mật độ theo khoảng cách:

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f02, 1, Car | 6.04 m | 2213 |
| s01f02, 4, Car | 15.22 m | 567 |
| s02f25, 3, Car | 29.81 m | 208 |
| s03f33, 5, Car | 33.83 m | 114 |


Theo bốn xe con này, khoảng cách tăng từ 6.04 m lên 33.83 m (gấp khoảng 5.6 lần) thì số điểm giảm từ 2213 xuống 114 (còn khoảng 5%); các mức 2213, 567, 208, 114 điểm nhìn chung giảm theo khoảng cách và gần với đường cong slide ở 0–10 m, 10–20 m, 20–30 m, nhưng mức 30–40 m cao hơn mốc 72 điểm.

**Tự kiểm.** Lần `sync` đầu có 10/70 cuboid bị cờ. Tôi xem lại từng hộp trên CVAT và `work/xem/`, rồi sửa:
- s03f33 #3 Bus và s03f52 #5 Bus: rộng 3,31 m và 3,38 m, quá rộng so với xe buýt (2,50–2,60 m). Tôi căn lại hướng ở khung Top cho cạnh dài song song thân xe, rồi thu hai cạnh bên vào sát cụm điểm. Sau khi sửa, cảnh báo chìm của s03f52 #5 cũng hết.
- s05f08 #9 Truck và s05f33 #4 Truck: rộng 3,30 m và 3,39 m. Tôi căn lại hướng cho thẳng thân xe, giữ phần gương chiếu hậu trong hộp theo guideline nhưng thu cạnh bên sát mép điểm gương.
- s04f01 #6 Car (dài 6,06 m): thu ngắn hai đầu cho sát điểm, giữ class Car.
- s04f55 #1, s04f55 #5 và s05f08 #1 Car (rộng 2,31 m, 2,33 m, 2,30 m): thu hẹp hai cạnh bên cho sát cụm điểm, giữ class Car.

Lần `sync` cuối còn 2/70 cảnh báo, tôi giữ nguyên:
- s03f52 #4 Car, đáy thấp hơn mặt đường 0,72 m: đây là xe ở làn đối diện, phần dưới bị dải phân cách có hàng cây che. Tôi suy luận đáy xe và kéo hộp xuống dưới mép hàng cây, đã bật occluded. Giả thuyết: mốc mặt đường của `selfcheck` (ước từ các điểm quanh hộp trong khoảng 4 m) lấy phải điểm của dải phân cách, không phải mặt đường làn đối diện. Trong bảng A, hộp này có `z_bottom` = −1,817 m, chỉ thấp hơn mức mặt đường tham chiếu −1,7 m khoảng 0,12 m (dưới ngưỡng 0,3 m), nên tôi cho rằng đáy hộp gần đúng mặt đường làn đối diện.
- s01f02 #3 Pedestrian, có 58 điểm sát ngay ngoài mép: tôi mở lại và thấy các điểm đó thuộc một người khác đi sát bên cạnh người đi bộ này, không phải của chính người này, nên không nới hộp để tránh bao dư.

Đổi kiểm: chưa thực hiện đổi kiểm với bạn bên cạnh.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Không trả lời phần này.

(b) Dùng AI:
- Công cụ: Claude (Anthropic).
- Câu hỏi chính: tóm tắt yêu cầu bài lab; gỡ lỗi cài đặt (đường dẫn `data` trỏ nhầm vào file zip, chưa kích hoạt `.venv` nên thiếu `cvat-sdk`); hệ trục VF (x trái, y sau, phía trước −y) và khác biệt giữa trục cảnh với vạch +X của cuboid; cách xem cuboid trên ảnh bằng `preview`; cách hiểu cảnh báo `selfcheck` và quy tắc bao gương chiếu hậu; cấu trúc báo cáo.
- Cách dùng kết quả: tôi tự xem lại từng hộp trên CVAT rồi tự quyết định sửa hay giữ, tự vẽ cuboid, tự chụp ảnh, tự đọc bảng A và điền số. Claude soạn khung báo cáo và các dòng ghi lại những gì tôi đã trao đổi; không xem ảnh, dữ liệu VF hay bảng A, và không điền số.

(c) Điều còn chưa chắc: đáy của xe s03f52 #4 bị dải phân cách che phần dưới nên là suy luận, chưa kiểm chứng được. Hộp xa nhất s03f33 #3 có `dist_m` = 50,07 m, nằm ngay ngoài ranh 50 m nên có thể không được chấm.
