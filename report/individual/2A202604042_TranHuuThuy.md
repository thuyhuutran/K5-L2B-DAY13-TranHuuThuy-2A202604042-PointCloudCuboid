# Báo cáo Day 13 — Trần Hữu Thủy, 2A202604042

## 1. Kết luận trước

Task `D13-2A202604042` trên Windows, CVAT 2.81.0. Trình duyệt và MSSV bạn ngồi cạnh: cần xác nhận.

Bài có **90 cuboid trên 10 frame**. `s02f40` có ít nhất (7 hộp). Ở frame này, các xe nhìn rõ nằm rải rác và nhiều vật ở xa mép đường nên khó đọc. Phiếu tự kiểm gắn cờ **8/90 hộp**. Tôi tự tin hơn với đáy và hướng của các xe gần, thấy rõ trên cả LiDAR lẫn camera. Chiều dài của xe bị cây che và kích thước vật ở xa khó xác định hơn.

- Khung 3D lớn: kéo chuột trái để xoay, kéo chuột phải để dời, lăn chuột để đổi mức zoom. Xoay nhìn thẳng xuống để đọc toàn cảnh.
- Chọn nhãn và **Shape**, nháy đúp để đặt hộp. Dùng **Top** để chỉnh tâm, kích thước ngang và yaw, rồi dùng **Side/Front** để chỉnh đáy và chiều cao.
- Bật **Appearance → Cuboid orientation**, căn vạch đỏ ngắn về đầu xe. Lệnh `python scripts/lab.py preview` chiếu các hộp lên ảnh camera để đối chiếu.

## 2. Đọc frame (T1)

Quy ước khi xoay ảnh nhìn từ trên xuống: trước xe (−y) ở phía trên, +x sang trái, +y xuống dưới. Đối chiếu hướng đánh dấu theo góc xoay thực tế trên từng ảnh.

| Frame | Mặt đường | Vật xa nhất trong các hộp đã ghi | Cảnh quanh xe | Cụm chưa đọc chắc |
|---|---|---|---|---|
| `s01f01` | Tương đối phẳng cục bộ, thấp hơn LiDAR khoảng 1,8 m gần xe | Bus #7, 32,16 m | Sedan sát bên trái, xe và buýt phía phải, hai người đi bộ ở vạch sang đường, nhiều xe phía sau | Vệt điểm sâu dưới mặt đường ở phía trước bên phải |
| `s01f50` | Tương đối phẳng cục bộ, thấp hơn LiDAR khoảng 1,8 m gần xe | Car #5, 21,04 m | Các sedan xếp ở phía trước và phía sau, hai người đi xe hai bánh bên phải | Phần thân của xe bị che trong dãy xe phía sau |

Ảnh: `t1-s01f01-3d.png`, `t1-s01f01-top.png`, `t1-s01f50-3d.png`, `t1-s01f50-top.png`. Hai ảnh `s01f50` cần bổ sung.

Camera cho thấy màu xanh của xe ở giao lộ, màu đèn tín hiệu và hình dáng xe hai bánh có người ngồi. LiDAR cho phép đọc khoảng cách theo mét, ước lượng độ cao mặt đường so với cảm biến và thấy vị trí vật quanh xe trên cùng hệ tọa độ 3D.

| Frame lướt | Hướng trước khi căn theo quy ước trên | Quan sát |
|---|---|---|
| `s02f22` | Mép trên | Hàng rào giữa đường che phần thấp của xe bên kia |
| `s02f40` | Mép trên | Xe phía xa có cụm điểm thưa |
| `s03f07` | Mép trên | Nhiều xe gần nhau, dễ lẫn điểm giữa các xe |
| `s03f57` | Mép trên | Buýt phía sau, nằm lệch làn |
| `s04f17` | Mép trên | Hàng rào vàng chắn một phần thân xe |
| `s04f52` | Mép trên | Hàng rào dọc giữa đường tách hai luồng xe |
| `s05f14` | Mép trên | Dải cây che phần dưới nhiều xe |
| `s05f29` | Mép trên | Van sáng màu bị cây che, khó xác định chiều dài |

## 3. Ba cuboid mẫu (T2)

| Ảnh | Hộp đối chiếu | Hướng và đáy | Kích thước theo bảng 4.3 |
|---|---|---|---|
| `t2-a-car.png` | `s01f01` #1, Car, xe gần nhất của frame (4,20 m) | Yaw 267,4° gần hướng tiến, đáy cao hơn mặt đường ước tính 0,05 m | 4,566 × 1,633 × 1,462 m. Rộng dưới khoảng sedan 1,70–1,90 m |
| `t2-b-bus-truck.png` | `s01f01` #7, Bus | Yaw 0,0° theo trục ngang cảnh, đáy cao hơn mặt đường ước tính 0,07 m | 11,621 × 2,875 × 3,130 m. Dài và cao trong khoảng xe buýt thường, rộng cao hơn khoảng tham khảo |
| `t2-c-ped-cyclist.png` | `s01f50` #8, Cyclist, đối chiếu lại với ảnh đã chụp | Yaw 271,6° gần hướng tiến, đáy thấp hơn mặt đường ước tính 0,02 m | 1,766 × 0,688 × 1,912 m. Nếu đối chiếu theo xe đạp có người, cao hơn mốc 1,90 m khoảng 0,01 m |

Đối chiếu lại ba ảnh với hộp đã chọn, hướng mũi tên và các ô Top/Side/Front trước khi nộp. Ảnh cắt riêng hộp khó xác nhận chính xác frame và idx. Kết quả vòng hiệu chuẩn và danh sách vật bỏ qua: cần bổ sung theo ghi chép buổi lab.

## 4. Bốn hiện tượng chất lượng (T4)

| Hiện tượng | Frame và ảnh | Giả thuyết và dấu hiệu LiDAR | Ảnh hưởng khi gán nhãn |
|---|---|---|---|
| Bóng che | `s03f07`, `t4-1-bong-che.png` (cần xác nhận frame ảnh) | Vật gần có thể chắn tia, làm vùng phía sau thưa hơn. | Khi quan sát được một phần vật, bật occluded và bao phần có thể suy ra. |
| Thưa ở xa | `s05f29` #6, `t4-2-thua-o-xa.png` (cần xác nhận frame ảnh) | Xe ở 48,59 m có 51 điểm trong hộp. Các tia cách nhau xa hơn trên bề mặt vật ở tầm xa. | Vẽ nếu thấy đủ khoảng 15–20% thể tích hoặc suy luận được. Nếu quá ít căn cứ, ghi LOW_VISIBILITY. |
| Phản xạ thấp hoặc ít điểm bất thường | `s03f07` #6, `t4-3-phan-xa-thap.png` | Sedan đỏ có 839 điểm ở 9,87 m, so với sedan bạc #5 có 1.519 điểm ở 8,46 m. Góc nhìn chính diện hoặc đặc tính bề mặt có thể góp phần làm giảm số điểm. | Đối chiếu cam09 và các góc LiDAR để đặt biên hộp. |
| Điểm lạ | `s01f01`, `t4-4-diem-la.png` | LiDAR có sáu điểm thành vệt ngắn ở khoảng 27 m phía trước bên phải, z khoảng −12,8 m. Có thể là phép đo sai hoặc phản xạ gián tiếp. | Bỏ qua vệt điểm khi gán vật. |

Nguồn số liệu: task cá nhân. Nếu một ảnh chụp từ task khác, ghi lại đúng frame và nguồn.

## 5. Số từ cuboid của tôi (T5)

Khoảng cách ngang `dist_m` đo trên mặt phẳng xy. Khoảng cách 3D r tính thêm cao độ tâm z.

| Cuboid | z (m) | h (m) | z − h/2 (m) | dist_m (m) | r (m) | Yaw | Đáy |
|---|---:|---:|---:|---:|---:|---:|---|
| `s01f01` #1 Car | −1,031 | 1,462 | −1,762 | 4,20 | 4,33 | 267,4° | Gần mặt đường |
| `s01f01` #7 Bus | −0,020 | 3,130 | −1,585 | 32,16 | 32,16 | 0,0° | Gần mặt đường |
| `s01f01` #8 Pedestrian | −0,921 | 1,696 | −1,769 | 16,09 | 16,12 | 78,9° | Gần mặt đường |

**Gần nhất:** `s03f07` #7, Car, khoảng cách ngang 3,37 m và r 3,53 m. **Xa nhất:** `s05f29` #6, Car, khoảng cách ngang 48,59 m và r 48,61 m.

Người đi bộ `s01f01` #8 có kích thước 0,873 × 0,552 × 1,696 m, trong khoảng người lớn ở bảng 4.3. Sedan `s01f01` #1 hơi hẹp theo bảng, có thể do biên thân xe ở phía khuất khó xác định. Car #1 phía trước cùng chiều có yaw 267,4°, gần 270°. Car #4 phía bên kia giao lộ có yaw 89,0°, gần 90°.

| Cuboid Car | Khoảng cách ngang (m) | Điểm trong hộp |
|---|---:|---:|
| `s01f01` #1 | 4,20 | 2.906 |
| `s01f01` #3 | 17,22 | 753 |
| `s02f40` #3 | 35,81 | 80 |
| `s05f29` #6 | 48,59 | 51 |

Trong mẫu này, từ 4,20 m lên 17,22 m, khoảng cách tăng khoảng 4,1 lần và số điểm còn khoảng 26%. Mức giảm phù hợp với xu hướng bài giảng, từ khoảng 1.968 điểm ở 0–10 m xuống 718 ở 10–20 m và 72 ở 30–40 m. Số điểm từng hộp còn phụ thuộc góc nhìn và che khuất.

Sau tự kiểm, `s01f01` #7 được tăng chiều cao Bus từ 2,24 lên 3,13 m để bao sát nóc. `s04f52` #6 được kéo dài hộp về phía đầu xe từ 2,58 lên 4,18 m. Phiếu mới còn 8 cảnh báo:

- Các cờ chiều rộng `s01f01` #4, `s03f07` #3, `s04f17` #8 và `s05f14` #6 lệch khoảng 1–5 cm so với ngưỡng, hoặc hông xe bị che nên khó xác định thêm.
- `s01f01` #8 có người đi sát bên. 53 điểm ngoài mép có thể thuộc người cạnh hộp.
- `s03f07` #4 và #8 là xe chở hàng nhỏ. Phần đầu xe khó quan sát, nên chiều dài vẫn cần đọc thận trọng.
- `s05f29` #9 có chiều dài 2,48 m. Cây che phần dưới và một phần thân, nên khó xác định trọn chiều dài từ điểm nhìn thấy.

Kiểm chéo với bạn cùng lớp: cần bổ sung MSSV, hai số đối chiếu và kết quả thực tế.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn theo mét giúp mô hình học vị trí và kích thước vật trong không gian thực qua nhiều góc nhìn. Với vật thưa điểm, quy tắc về mức độ quan sát giúp các hộp suy ra có độ nhất quán và tránh gán các cụm quá mơ hồ.

- Công cụ: Codex.
- Câu hỏi chính: chỉ vị trí cuboid bị cờ, tìm ví dụ T4 và tổng hợp số liệu báo cáo.
- Cách dùng: đối chiếu ảnh và LiDAR, hỗ trợ chỉnh một số hộp qua CVAT và xuất lại CSV, rồi dùng số liệu mới để lập báo cáo.

Điều còn chưa chắc là biên xe sau dải cây và nguyên nhân cụ thể của các cụm thưa hoặc điểm nằm sâu dưới mặt đường. Tình trạng lỗi phát hiện sau hạn: cần xác nhận theo thời điểm nộp.

