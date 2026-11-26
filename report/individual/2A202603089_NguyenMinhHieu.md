# Báo cáo Day 13 — Nguyễn Minh Hiếu_2A202603089

## 1. Kết luận trước
- **Mã số (MSSV):** 2A202603089
- **Task:** `D13-2A202603089`
- **10 frame:** `s01f08`, `s01f56`, `s02f07`, `s02f37`, `s03f16`, `s03f33`, `s04f38`, `s04f55`, `s05f19`, `s05f43`
- **Môi trường:** CachyOS Linux, Zen Browser, CVAT 2.83.1
- **MSSV bạn ngồi cạnh:** 2A202603099

Tổng số: 62 cuboid (`s01f08`=10, `s01f56`=6, `s02f07`=6, `s02f37`=9, `s03f16`=7, `s03f33`=3, `s04f38`=7, `s04f55`=4, `s05f19`=5, `s05f43`=5).
Frame ít nhất: `s03f33` (3 cuboid) (frame thứ 6).
Tự kiểm: 1/62 cuboid còn cảnh báo (`s05f43`, `idx=5`, `Car`: `width=2,43 m` và `height=2,37 m` vượt ngưỡng bảng 4.3 do là dòng xe bán tải/SUV cỡ lớn; đã sửa lỗi chính tả `selftcheck` → `selfcheck`, `pedestian` → `pedestrian`).

Đổi góc nhìn: Kéo chuột trái để xoay (xoay thẳng xuống để nhìn toàn cảnh), chuột phải để dời, lăn chuột để zoom, nháy đúp về góc mặc định.
Vẽ và chỉnh cuboid: Chọn class → Shape → nháy đúp vào cụm điểm để đặt; chỉnh tâm/kích thước/yaw ở Top và chỉnh độ cao, đáy chạm đường ở Side/Front (không xoay).
Hướng & preview: Bật Appearance → Cuboid orientation để xoay vạch đỏ (+X) chỉ đầu xe ở khung Top; bấm Save rồi chạy `python scripts/lab.py preview` để xem cuboid trên 7 ảnh camera (`work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f08-3d.png`, `t1-s01f08-3d-top.png`, `t1-s01f56-3d.png`, `t1-s01f56-3d-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f08 | cạnh trên; +x chỉ sang phải, +y chỉ lên trên | phẳng; thấp hơn LiDAR ~1,7 m | 47,94 m (xe tải) | Car, Truck, Cyclist | xe con phía sau và hai bên làn, vũng nước phản xạ trên mặt đường | một vài cụm điểm thưa ở rìa xa > 45 m chưa đủ hình dạng để phân loại |
| s01f56 | cạnh phải; +x chỉ sang trái/lên trên tùy góc xoay, +y chỉ hướng tiến | phẳng; thấp hơn LiDAR ~1,75 m | 82,23 m (building) | Car, công trình (building) | hàng xe con nối đuôi phía sau và bên cạnh | cụm điểm nhỏ sát tường rào xa ngoài vùng làn đường |

Ba điều **chỉ camera** cho biết (lấy từ `s01f08` và `s01f56`):
1. Màu sắc thân xe, đèn xi-nhan/đèn phanh đang sáng và vạch kẻ phân làn trên mặt đường.
2. Phân biệt rõ mặt đường ướt/vũng nước phản chiếu với hố lõm hay vật cản vật lý.
3. Nhận diện chính xác class của các cụm ở xa hoặc bị che một phần (ví dụ phân biệt người đi xe đạp với người đi bộ hay biển báo).

Ba điều **chỉ LiDAR** cho biết (lấy từ `s01f08` và `s01f56`):
1. Tọa độ không gian 3D chính xác $(x, y, z)$, khoảng cách thực tế tới xe thu thập (`dist_m`, ví dụ `35,30 m` ở `s01f08` idx 4) và độ cao mặt đường (`z_bottom ≈ -1,7 m`).
2. Kích thước hình học thật tính bằng mét (`length`, `width`, `height`) không bị biến dạng bởi góc nhìn phối cảnh.
3. Quan sát toàn cảnh 360° xung quanh xe từ góc nhìn Top và phát hiện vật thể ở vùng tối/ngược sáng mà camera khó nhìn rõ.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f07 | cạnh trên | có 2 xe buýt kích thước lớn (`Bus`, dài `10,79 m` và `12,37 m`) ở làn bên phải |
| s02f37 | cạnh trên | xuất hiện cụm người đi bộ (`Pedestrian`, idx 8, 9) và xe đạp (`Cyclist`, idx 7) đi sát nhau bên lề trái |
| s03f16 | cạnh trên | nhiều xe con nối đuôi thẳng hàng dọc theo trục tiến/lùi của xe ego |
| s03f33 | cạnh trên | frame thưa đối tượng nhất trong toàn bộ 10 frame (chỉ có 3 cuboid: 1 `Car`, 1 `Pedestrian`, 1 `Cyclist`) |
| s04f38 | cạnh trên | có 1 xe tải trung (`Truck`, idx 5, dài `6,76 m`) và một số xe ở xa hơi lún nhẹ xuống mặt đường (`z_bottom ≈ -2,01 m`) |
| s04f55 | cạnh trên | các xe phân bố gọn ở cự ly gần và trung bình (`5,76 m` – `15,48 m`), đáy cuboid bám rất sát mặt đường (`ground_gap_m` từ `-0,01 m` đến `0,07 m`) |
| s05f19 | cạnh trên | có xe ở hai chiều lưu thông ngược nhau (`yaw_deg ≈ 272°` và `yaw_deg ≈ 92°`) |
| s05f43 | cạnh trên | xuất hiện xe con thân rộng/cao (`idx=5`, `width=2,43 m`, `height=2,37 m`) vượt nhẹ ngưỡng chuẩn của `Car` |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f08`, `idx=1` | `Car` | Có (`268,1°`, chỉ đúng đầu xe) | Có (`z_bottom = -1,571 m`, `ground_gap_m = 0,20 m`) | Có | Có (`4,48 × 2,25 × 1,36 m`) |
| (b) xe buýt hoặc xe tải | `s02f07`, `idx=3` | `Bus` | Có (`83,1°`, chỉ đúng đầu xe buýt) | Có (`z_bottom = -1,772 m`, `ground_gap_m = 0,17 m`) | Có | Có (`10,79 × 3,15 × 3,23 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s02f37`, `idx=7` | `Cyclist` | Có (`89,0°`, chỉ đúng hướng di chuyển) | Có (`z_bottom = -1,810 m`, `ground_gap_m = -0,01 m`) | Có | Có (`1,39 × 0,78 × 1,71 m`) |

Vòng hiệu chuẩn: Cuboid của mình ban đầu có xu hướng đặt hơi rộng về chiều ngang (`width`) ở các xe bị khuất một bên sườn và đặt đáy hơi cao (`ground_gap_m` khoảng `0,20 m` – `0,25 m`) do lấy theo điểm thấp nhất của thân xe thay vì mặt đường thực tế. Cả lớp thống nhất cách hiểu guideline: luôn hạ đáy cuboid chạm tới mặt phẳng đường xung quanh (`z_bottom ≈ -1,7 m`), giữ kích thước xe ở xa theo kích thước chuẩn của loại xe đó thay vì co nhỏ theo số điểm nhìn thấy, và đối chiếu lại khung chiếu trên cả 7 ảnh camera sau khi chỉnh.

Các vật bỏ qua vì không đủ cơ sở:
- `s01f08`, rìa xa phía trước (`dist_m > 45 m`): cụm điểm chỉ còn vài tia laser rời rạc, nằm ngoài tầm nhìn rõ của camera → bỏ qua (`LOW_VISIBILITY`).
- `s01f56`, bên lề phải phía sau dải phân cách (`dist_m > 40 m`): cụm điểm bị che khuất một phần bởi xe phía trước, không xác định được là xe con hay vật thể tĩnh bên đường → bỏ qua (`UNCERTAIN_CLASS`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f08` / `t4-1-bong-che.png` / task `D13-2A202603089` | Tia laser từ LiDAR trên nóc xe ego bị thân xe gần chắn ngang, tạo vùng khuất phía sau vật cản | Xuất hiện khoảng trống đen hình nón không có điểm phía sau xe gần; xe phía sau chỉ hiện một phần đầu hoặc sườn | Không thu nhỏ cuboid theo phần điểm nhìn thấy; ước lượng đủ chiều dài/rộng xe dựa vào ảnh camera và kích thước chuẩn |
| 2. Thưa ở xa | `s02f37` (`idx=6`, `35,06 m`) / `t4-2-thua-o-xa.png` / task `D13-2A202603089` | Các chùm tia laser phân kỳ theo góc quét nên mật độ tia trên một đơn vị diện tích giảm mạnh ở cự ly > 30 m | Xe ở `35,06 m` chỉ còn `126` điểm tạo thành các vạch ngang thưa thớt, mất chi tiết góc cạnh | Bám vào các điểm biên còn lại (cản trước/sau, nóc xe), kết hợp ảnh camera để giữ kích thước và góc `yaw` chuẩn |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f08` (`idx=10`, `27,09 m`) / `t4-3-phan-xa-thap.png` / task `D13-2A202603089` | Bề mặt vật thể tối màu, kính xe hoặc góc tới tia laser quá nghiêng làm tín hiệu phản xạ yếu dưới ngưỡng thu | Dù ở cự ly `27,09 m` (gần hơn `idx=4` ở `35,30 m`) nhưng chỉ có `87` điểm, thân xe bị rỗng từng mảng | Kiểm tra chéo trên ảnh camera để xác nhận ranh giới thực của thân xe, tránh bỏ sót vật thể |
| 4. Điểm lạ | `s01f08` / `t4-4-diem-la.png` / task `D13-2A202603089` | Tia laser phản xạ đa đường (multipath) qua vũng nước trên mặt đường hoặc kính xe tạo điểm nhiễu lệch vị trí | Xuất hiện các điểm lơ lửng dưới mặt đường (`z < -2,0 m`) hoặc cụm điểm nhiễu rời rạc không khớp hình dạng vật thể | Không gán nhãn cho cụm điểm phản xạ ảo dưới mặt đường; không kéo đáy cuboid xuống theo các điểm nhiễu này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603089-bangA.csv` và `submission/D13-2A202603089-tukiem.csv` (mặt đường chuẩn khoảng `-1,7 m`, lệch quá `0,3 m` là lơ lửng hoặc chìm).

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s03f16`, `idx=1`, `Truck` | -0,638 | 1,825 | -1,551 | 2,71 | 2,79 | 272,3 | chạm đường (`ground_gap_m = 0,20 m`) |
| `s01f08`, `idx=1`, `Car` | -0,891 | 1,361 | -1,572 | 4,00 | 4,10 | 268,1 | chạm đường (`ground_gap_m = 0,20 m`) |
| `s02f07`, `idx=3`, `Bus` | -0,156 | 3,232 | -1,772 | 14,18 | 14,18 | 83,1 | chạm đường (`ground_gap_m = 0,17 m`) |
| `s02f37`, `idx=8`, `Pedestrian` | -0,683 | 1,682 | -1,524 | 9,53 | 9,55 | 268,7 | chạm đường (`ground_gap_m = 0,24 m`) |
| `s01f08`, `idx=10`, `Car` | -1,288 | 2,118 | -2,347 | 27,09 | 27,12 | 91,5 | chìm nhẹ (`z_bottom` thấp hơn `-1,7 m` khoảng `0,65 m`, nhưng `ground_gap_m = 0,03 m` do dốc cục bộ) |
| `s01f08`, `idx=4`, `Car` | -0,666 | 1,727 | -1,530 | 35,30 | 35,31 | 274,1 | chạm đường (`ground_gap_m = 0,18 m`) |

**(a) Cuboid gần nhất và xa nhất:**
- **Gần nhất:** `s03f16`, `idx=1` (`Truck`) với `dist_m = 2,71 m` và $r = \sqrt{2,688^2 + (-0,380)^2 + (-0,638)^2} = 2,79\text{ m}$.
- **Xa nhất:** `s01f08`, `idx=4` (`Car`) với `dist_m = 35,30 m` và $r = \sqrt{0,630^2 + 35,295^2 + (-0,666)^2} = 35,31\text{ m}$.

**(b) Kích thước $l, w, h$ so với bảng 4.3:**
- **Người đi bộ (`s02f37`, `idx=8`, `Pedestrian`):** $l = 0,620\text{ m}$, $w = 0,665\text{ m}$, $h = 1,682\text{ m}$ — tất cả các chiều đều nằm gọn trong khoảng chuẩn của `Pedestrian` và không có cảnh báo trong phiếu tự kiểm.
- **Xe con (`s05f43`, `idx=5`, `Car`):** $l = 4,917\text{ m}$, $w = 2,426\text{ m}$, $h = 2,365\text{ m}$ — chiều rộng (`2,43 m` ngoài khoảng `1,4–2,3 m`) và chiều cao (`2,37 m` ngoài khoảng `1,2–2,3 m`) vượt nhẹ ngưỡng chuẩn của `Car` do đây là xe bán tải/SUV thân lớn, điểm quét bao gồm cả phần gương chiếu hậu hai bên và giá nóc.

**(c) Góc hướng `yaw_deg`:**
- **Xe cùng chiều phía trước (`s01f08`, `idx=2`, `Car`):** `yaw_deg = 270,9°`, rất gần `270°` (hướng di chuyển dọc theo trục làn đường).
- **Xe ngược chiều (`s01f08`, `idx=7`, `Car`):** `yaw_deg = 89,5°`, rất gần `90°` (ngược chiều `180°` so với hướng `270°` của xe cùng chiều).

Mật độ điểm theo khoảng cách (4 cuboid `Car` ở các `dist_m` tăng dần):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f08`, `idx=1`, `Car` | 4,00 | 3130 |
| `s01f08`, `idx=6`, `Car` | 12,18 | 1142 |
| `s01f08`, `idx=2`, `Car` | 21,46 | 297 |
| `s01f08`, `idx=4`, `Car` | 35,30 | 98 |

Quy luật: Khi khoảng cách xa gấp ~3 lần (từ `4,00 m` lên `12,18 m`), số điểm giảm từ `3130` xuống còn `1142` (còn khoảng 36%, gần với quy luật suy giảm theo bình phương khoảng cách khi xét cả góc che khuất); xa gấp ~5,4 lần (`21,46 m`) thì số điểm chỉ còn `297` (khoảng 9,5%). Số liệu thực tế này khớp với đường cong tham chiếu trên slide bài giảng (`1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`), nhỉnh hơn nhẹ do xe ở làn gần không bị che khuất.

Tự kiểm:
- **Hai cuboid đã sửa sau `preview` / `selfcheck`:**
  1. `s01f08`, `idx=6` (`Car`): hạ cao độ $z$ để đáy cuboid chạm sát mặt đường (giảm `ground_gap_m` xuống `0,10 m`) và chỉnh lại góc `yaw_deg` về `87,9°` cho khớp khung chiếu trên ảnh camera.
  2. `s02f37`, `idx=1` (`Car`): dịch tâm $(x, y)$ ở khung Top để khung chiếu cuboid trên camera ôm khít đầu và sườn xe, đưa `ground_gap_m` về `0,00 m`.
- **Cảnh báo giữ nguyên (1/62 cuboid):** `s05f43`, `idx=5` (`Car`) báo `width=2,43 m` (ngoài khoảng `1,4–2,3 m`) và `height=2,37 m` (ngoài khoảng `1,2–2,3 m`). Giữ nguyên vì đối chiếu trên camera đây là xe bán tải/SUV cỡ lớn, kích thước cuboid đã ôm sát đúng các điểm LiDAR thực tế trên thân xe và gương chiếu hậu; nếu ép co vào ngưỡng `2,3 m` sẽ làm cắt mất phần sườn xe.

Đổi kiểm:
- **MSSV bạn kiểm chéo:** `2A202603099`
- **Hai con số đối chiếu trên bảng A:** tổng số cuboid của `s01f08` (`10` cuboid) và khoảng cách `dist_m` của cuboid gần nhất `s03f16` `idx=1` (`2,71 m`) — kết quả khớp hoàn toàn giữa file `bangA.csv` và giao diện CVAT/ảnh preview, không phát sinh sai lệch cần sửa thêm.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
**(a) Ý nghĩa từ phía mô hình học máy:**
Nhãn 3D phải đo bằng đơn vị mét $(x, y, z, l, w, h, \text{yaw})$ trong hệ tọa độ thực thay vì pixel trên ảnh 2D vì hệ thống tự lái cần biết chính xác khoảng cách vật lý, vận tốc tương đối và kích thước thật của vật cản để lập quỹ đạo tránh va chạm mà không bị sai số do biến dạng phối cảnh. Các cụm vật thể thưa điểm ở xa cần quy tắc gán riêng (ước lượng đủ kích thước chuẩn dựa trên ngữ cảnh và camera thay vì bó chặt theo vài điểm ít ỏi) để mô hình học được hình học toàn vẹn của đối tượng, tránh việc mạng nơ-ron dự đoán ra các hộp nhỏ biến dạng khi gặp dữ liệu LiDAR thưa ở cự ly xa.

**(b) Dùng AI:**
- **Công cụ:** Gemini
- **Câu hỏi chính:** Cách tăng độ sáng/kích thước hiển thị point cloud trong CVAT trên Zen Browser và tổng hợp, đối chiếu số liệu từ `bangA.csv` và `tukiem.csv` vào mẫu báo cáo.
- **Cách dùng kết quả:** Áp dụng userscript tăng kích thước điểm shader trong CVAT để nhìn rõ cụm điểm tối/xa khi gán nhãn, và kiểm tra lại toàn bộ các phép tính từ file CSV trước khi đưa vào báo cáo.

**(c) Điều còn chưa chắc và lỗi phát hiện sau hạn:**
Với các xe ở cự ly xa trên 25 m bị khuất một phần thân sau hoặc nằm trên đoạn đường có độ dốc nhẹ, việc xác định chính xác tuyệt đối cao độ đáy cuboid và chiều dài phần đuôi bị khuất vẫn còn độ sai số nhỏ giữa góc nhìn Top và hình chiếu trên camera. Sau khi chạy tự kiểm, nhận thấy một vài cuboid ở xa có `z_bottom` hơi thấp hơn mức `-1,7 m` chuẩn dù `ground_gap_m` vẫn nằm trong ngưỡng cho phép do mặt đường cục bộ hơi trũng xuống.