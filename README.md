# 🪷 Thực Hành Thiền Tuệ Online (Chùa Online 3D Việt Nam)
**Tài liệu Kiến trúc Kỹ thuật & Hướng dẫn Bàn giao cho AI Phiên Sau (AI Handover Guide)**

---

## 1. Tổng Quan Dự Án, Triển Khai (Deployment) & Cách Chạy

- **Mô tả:** Không gian Thiền Viện 3D trực tuyến xây dựng bằng **Three.js (WebGL)** kết hợp mô hình 3D `.glb` và kiến trúc cổ truyền Việt Nam được dựng bằng hình học thủ tục (procedural geometry).
- **Tên miền chính thức (Custom Domain):** **https://thienvienonline.com** *(và tên miền phụ Cloudflare Pages: https://thien-tue-online.pages.dev)*
- **Kho chứa mã nguồn (GitHub):** **https://github.com/nguyenchilinhedu-create/thien-tue-online.git** (nhánh `main`).
- **Quy trình cập nhật lên mạng (Auto-Deploy CI/CD):**
  Mỗi khi AI chỉnh sửa xong trên máy, chỉ cần chạy:
  ```powershell
  git add -A
  git commit -m "Mô tả thay đổi"
  git push origin main
  ```
  Cloudflare Pages đã được kết nối trực tiếp với nhánh `main` của GitHub và sẽ tự động cập nhật bản mới lên `https://thien-tue-online.pages.dev` sau khoảng **20–30 giây** (Lưu ý: nếu thay đổi các file `.glb` trong `models_3d/`, hãy tăng tên cache `MODEL_CACHE_NAME` trong `loadModels()` ví dụ từ `'thien-tue-3d-models-v2'` lên `'thien-tue-3d-models-v3'` để trình duyệt người dùng tự động tải lại mô hình mới).
- **Kiến trúc Single-File:** Toàn bộ HTML, CSS, UI và logic 3D/AI nằm gọn trong **1 file duy nhất**: [`index.html`](./index.html) (~6.830 dòng).
- **Cách chạy Local Server để test:**
  ```powershell
  python -m http.server 8080 --directory "d:\Thien Vipassana\ChuaOnline"
  ```
  Sau đó mở trình duyệt tại: `http://localhost:8080` (nhấn `Ctrl + F5` sau mỗi lần sửa để xóa cache).

---

## 2. Cấu Trúc Thư Mục & Tài Nguyên Quan Trọng

```text
d:\Thien Vipassana\ChuaOnline\
├── index.html                        # File mã nguồn chính (HTML + CSS + Three.js + AI + Audio)
├── Ambient ThienNCL.MP3              # Âm thanh nền thu thực tế khi hành thiền (tự động phát lặp vô tận)
├── models_3d\                        # Thư mục chứa các mô hình 3D (.glb)
│   ├── chanh_dien_chua_viet.glb      # Mô hình Chánh Điện Đại Thiền Đường & Bàn thờ Phật
│   ├── su_phu.glb                    # Mô hình 3D Sư Phụ đắp y Kāsāya ngồi thiền định
│   ├── be_nam_ao_lam.glb             # Mô hình nhân vật Nam (Áo tràng lam)
│   ├── be_nu_ao_hong.glb             # Mô hình nhân vật Nữ (Áo tràng hồng)
│   ├── tuong_phat_hoang.glb          # Đại tượng Phật Hoàng Trần Nhân Tông
│   └── tuong_bac_ho.glb              # Đại tượng Chủ tịch Hồ Chí Minh
├── KE_HOACH_THIET_KE_DI_CHUA_ONLINE_VIET_NAM.md
└── README.md                         # Tài liệu hướng dẫn này dành cho AI các phiên sau
```

---

## 3. Hệ Trục Tọa Độ 3D & Bản Đồ Không Gian (Spatial Map)

Hệ trục tọa độ Three.js trong dự án quy ước:
- **Trục X:** Tây (`-X`, bên Trái khi nhìn từ cổng vào) ↔ Đông (`+X`, bên Phải).
- **Trục Z:** Bắc (`-Z`, hướng Bàn thờ Phật / Hậu chùa / Đại Sơn) ↔ Nam (`+Z`, hướng Cổng Tam Quan / Đại Hồ).
- **Trục Y:** Độ cao mặt sàn (`0.0` ngoài sân; `HALL_FLOOR_Y = 0.45` trên nền Thượng Điện).

### 3.1. Ba Phân Vùng Tường Bao (`getZoneCompartment(x, z)`)
Mọi thuật toán tìm đường bắt buộc phân biệt 3 phân vùng ngăn cách bởi tường:
1. **`'hall'` — Lòng Chánh Điện Đại Thiền Đường:**
   - Phạm vi: `|x| <= 8.05`, `z ∈ [-6.95, 9.05]`, cao độ sàn `HALL_FLOOR_Y = 0.45m`.
   - **Tòa Tọa Cụ & Sư Phụ (`SU_PHU_POS`):** Đặt tại `{ x: 0.0, z: -4.85 }` (thụt lùi 1 ô gạch phía Bắc hàng bồ đoàn đầu tiên `z = -3.2`), mặt hướng Nam (`rotation.y = 0`). Có **bức tường vô hình bán kính `r = 1.10m`** bao quanh, không nhân vật hay GHOST nào bước qua được.
   - **36 Bồ đoàn trong Chánh Điện:** 6 hàng `rz ∈ [-3.2, -1.6, 0.0, 1.6, 3.2, 4.8]` × 6 cột `cx ∈ [-5.0, -3.6, -2.2, 2.2, 3.6, 5.0]`. Giữa `cx = -2.2` và `cx = +2.2` là **Đại lộ trung tâm rộng 4.4m (`x = 0.0`)**.
   - **4 Cột gỗ chính:** Đặt tại `(±6.2, -2.5)` và `(±6.2, 3.5)`.
   - **3 Cửa ra vào Chánh Điện:**
     - Cửa Chính mặt tiền: `n_hall_door_f (0.0, 9.05)` (bậc tam cấp `z: 9.05 -> 10.45`).
     - Cửa Hông Trái: `n_hall_door_l (-8.05, 0.8)` (bậc tam cấp `x: -8.15 -> -9.35`).
     - Cửa Hông Phải: `n_hall_door_r (8.05, 0.8)` (bậc tam cấp `x: 8.15 -> 9.35`).

2. **`'inner'` — Sân Nội Tự (Bao quanh bởi Tường Rào Nội Tự):**
   - Phạm vi: `|x| < 13.5`, `z ∈ [-7.0, 22.5]` (ngoài Chánh Điện).
   - **Lư Hương Chính:** `(0.0, 15.2, r = 0.95)`.
   - **Gác Chuông 2 Tầng:** Tâm `(10.2, 17.0)`, vùng cấm leo lên bệ đá `x ∈ [7.30, 13.10], z ∈ [14.10, 19.90]`.
   - **12 Bồ đoàn Sân Thiền Vườn (`san_chua`):** `cx ∈ [-6.0, -4.2, 4.2, 6.0]`, `rz ∈ [14.5, 16.2, 17.9]`.
   - **5 Cổng Vòm trên Tường Rào Nội Tự:**
     - Cổng Nam Nội Tự: `n_gate_south (0.0, 22.5)` — Hoành phi: *"CHÁNH NIỆM & BUÔNG BỎ"*.
     - Cổng Tây Bắc (sang Đài Phật Hoàng): `n_gate_plaza_l (-13.5, 0.8)` — Hoành phi: *"TRÚC LÂM THIỀN"*.
     - Cổng Đông Bắc (sang Đài Bác Hồ): `n_gate_plaza_r (13.5, 0.8)` — Hoành phi: *"CHÍ CÔNG VÔ TƯ"*.
     - Cổng Tây Nam (Đường Kinh Hành): `n_gate_kinh_l (-13.5, 12.5)` — Hoành phi: *"THIỀN HÀNH ĐẠO"*.
     - Cổng Đông Nam (Đường Kinh Hành): `n_gate_kinh_r (13.5, 12.5)` — Hoành phi: *"CHÁNH NIỆM ĐẠO"*.

3. **`'outer'` — Khuôn Viên Ngoại Vi (Trong Đại Tường Bao `|x| <= 37.5, z ∈ [-15.5, 46.0]` & Bến Hồ):**
   - **Quảng trường Đài Tượng Phật Hoàng Trần Nhân Tông (Bên Trái):** Tâm tượng `(-22.0, -5.2)`, lư hương `(-22.0, -3.0)`, 16 bồ đoàn `cx ∈ [-25, -23, -21, -19] × rz ∈ [-1.2, 0.4, 2.0, 3.6]`.
   - **Quảng trường Đài Tượng Chủ Tịch Hồ Chí Minh (Bên Phải):** Tâm tượng `(22.0, -5.2)`, lư hương `(22.0, -3.0)`, 16 bồ đoàn `cx ∈ [19, 21, 23, 25] × rz ∈ [-1.2, 0.4, 2.0, 3.6]`.
   - **Hồ Sen Tịnh Tâm:** Tâm `(-15.5, 27.0)`, bán kính mặt nước `5.15m`, đường đá vòng cung phía Nam ôm quanh bờ hồ.
   - **Giếng Cổ Đá Ong:** Tâm `(14.5, 27.0)`, bán kính thành giếng `1.45m`, sân đá vuông bao quanh.
   - **Đại Cổng Tam Quan:** Tại `z = 46.0` (Cửa chính: *"THIỀN VIỆN TRÚC LÂM VIỆT NAM"*, Cửa Tả: *"Đạo Pháp – Dân Tộc"*, Cửa Hữu: *"Từ Bi – Trí Tuệ"*).

---

## 4. Bản Đồ Mã Nguồn Trong `index.html` (Codebase Index)

Để tìm và sửa nhanh trong `index.html`, AI phiên sau có thể tra cứu theo các phân đoạn dòng sau:

| Khu vực chức năng | Dòng (xấp xỉ) | Các hàm / biến quan trọng |
| :--- | :--- | :--- |
| **CSS & Giao diện HUD / Modal** | `17 – 384` | `#modal-overlay`, `#hud-temple-wrap`, `#hud-online`, `#hud-actions`, `#ambient-wave-ind` |
| **Cấu hình Camera & Kinh Hành** | `449 – 505` | `smartCam`, `KINH_HANH_PATH`, `kinhHanhEntryWaypoints` |
| **Dữ liệu 80 Bồ Đoàn & 8 GHOST** | `548 – 590` | `BD_POS`, `BD_OCCUPIED`, `GHOSTS`, `ghostObjs`, `SU_PHU_POS` |
| **Tọa Thiền (Glide) & Độ cao sàn** | `615 – 725` | `findNearestFreeBD`, `startGlideTo`, `stopSit`, `updateFloorY` |
| **Hệ thống Ánh sáng Ngày / Đêm** | `730 – 980` | `lightingSystem`, `setupLights`, `applyLightingState`, `setLightingMode` |
| **Tạo Nhân vật, Sư Phụ & Váy Quỳ 3D** | `1020 – 1420` | `makeLabel`, `createSittingSkirtMesh`, `spawnPlayer`, `spawnGhosts`, `spawnSuPhu` |
| **Dựng Cảnh Chùa 3D & Đường Đá** | `1450 – 2600` | `buildFullTempleGrounds`, `addStoneRect`, `buildGrandStatuePlaza`, `addSideArchGate` |
| **Smart Camera 2D & Tránh Tường** | `3380 – 3630` | `solveCameraCorridor2D`, `recordPlayerTrailPoint`, `updateSmartCamera` |
| **Chuỗi Động Tác Lạy Phật & Hành Động**| `3635 – 3885` | `startBowSequence`, `updateBowSequence`, `doKinhHanh`, `doLay`, `doThien`, `doTrinhPhap` |
| **Va Chạm Tường (`canMoveTo`)** | `3890 – 3948` | `canMoveTo(nx, nz)` — Kiểm tra toàn bộ tường, bệ tượng, hồ sen, gác chuông, Sư Phụ |
| **Đồ Thị Dẫn Đường & Lách Tránh** | `3950 – 4230` | `NAV_NODES`, `DOORWAY_NODES`, `NAV_EDGES`, `STATIC_OBSTACLES`, `computeMindfulSteer` |
| **Chọn Chỗ Đứng Trống & Điều Hướng** | `4245 – 4565` | `isOnPavedSurface`, `resolveFreeStandingSpot`, `goToDestination`, `triggerDestinationArrivalCam` |
| **Di Chuyển Người Chơi (`handleMove`)** | `4575 – 4795` | `stepPlayerWithWallSlide`, `handleMove`, `animPlayer` |
| **Đoạn Tường Đặc (`SOLID_WALL_SEGMENTS`)**| `4796 – 4835` | `SOLID_WALL_SEGMENTS` (dùng chung cho Camera, Ẩn/Hiện tên và Chống đi xuyên tường) |
| **AI Đồng Tu Tự Động (`animGhosts`)** | `4840 – 5520` | `getZoneCompartment`, `buildGhostWalkPath`, `stepGhostWithWallSlide`, `assignNextGhostWalkMission`, `animGhosts` |
| **Hệ Thống Chim Chùa Tự Do** | `5525 – 5810` | `createBirdMesh`, `initTempleBirds`, `updateTempleBirds` |
| **Âm Thanh Chuông & Nhạc Nền Thiền** | `6420 – 6620` | `playTempleBellSound`, `initAmbientAudio`, `ensureAmbientPlaying`, `updateOnlineCount` |

---

## 5. Các Hệ Thống Cốt Lõi & Quy Tắc Hoạt Động

### 5.1. Hệ Thống Dẫn Đường & Chống Đi Xuyên Tường (Cho Cả Người Chơi & GHOST)
1. **Dùng chung hàm tìm đường [`buildGhostWalkPath(x1, z1, x2, z2)`](./index.html):**
   - Được sử dụng cho cả:
     - GHOST di chuyển tự động (`assignNextGhostWalkMission`),
     - Người chơi nhấn chọn điểm đến trên menu (`goToDestination`),
     - Người chơi nhấn nút **Tọa Thiền** (`startGlideTo`),
     - Người chơi nhấn nút **Kinh Hành** từ góc tường (`doKinhHanh`).
2. **Quy tắc đi trong Chánh Điện (`getHallRowAisleZ`):**
   - Nhân vật đứng dậy từ bồ đoàn `(x, z)` luôn bước dọc `0.8m` ra khe trống giữa 2 hàng bồ đoàn (`rowZ ∈ [-2.4, -0.8, 0.8, 2.4, 6.6]`), sau đó đi ngang ra **Đại lộ trung tâm `x = 0.0`** (rộng 4.4m).
   - **Tuyệt đối không** cho GHOST đi dọc theo sát vách tường biên `x = ±6.8` (vì sẽ vướng 4 cột gỗ ở `x = ±6.2`).
3. **Quy tắc đi qua Cửa / Cổng Vòm (`DOORWAY_NODES`):**
   - `findNearestNavNodeKey` không bao giờ chọn trực tiếp điểm nằm ngay giữa khe cửa (`DOORWAY_NODES`) và chỉ chọn điểm trong cùng phân vùng (`getZoneCompartment`).
   - Nhờ vậy, mọi lộ trình qua cửa đều bắt buộc đi qua 3 điểm thẳng hàng vuông góc với cửa: **`[Điểm căn trước cửa] → [Tâm cửa] → [Điểm căn sau cửa]`**.
   - Khi nhân vật ở trong bán kính `2.15m` quanh bất kỳ cửa nào (`isNearAnyDoorway`), hàm `computeMindfulSteer` tự động giảm 82% lực lách ngang để nhân vật đi thẳng qua tâm cửa, không bị đẩy lệch vào mép tường.
4. **Khóa va chạm tường vật lý (`stepPlayerWithWallSlide` & `stepGhostWithWallSlide`):**
   - Mọi bước cập nhật tọa độ `(x, z)` đều phải kiểm tra `canMoveTo(nx, nz)` và `!doesSegmentCrossWall(px, pz, nx, nz)`.

### 5.2. Hệ Thống Đồng Tu Tự Động (Autonomous Ghosts AI)
- Có 8 đồng tu (`GHOSTS`) tự động luân chuyển giữa các trạng thái:
  - `'sit'` (Ngồi thiền trên bồ đoàn),
  - `'bow'` (Lạy Phật trên bồ đoàn),
  - `'stand_up'` / `'kneel_down'` (Chuyển tư thế đứng ↔ ngồi mượt mà),
  - `'walk_path'` (Đi bộ theo đường đá sang vị trí mới),
  - `'kinh_hanh'` (Đi thiền hành chánh niệm theo `KINH_HANH_PATH`),
  - `'admire'` (Đứng chiêm bái tượng Phật Hoàng, Bác Hồ, Gác Chuông, Hồ Sen, Giếng Cổ, Tam Quan).
- **Ràng buộc bắt buộc:** Hàm `countMainHallSitters` và `countMainHallSitOrInbound` đảm bảo **trong Chánh Điện luôn có ít nhất 1–2 đồng tu đang ngồi thiền** giữ đạo tràng.
- **Tránh va chạm đối đầu:** Trong `computeMindfulSteer`, khi 2 nhân vật đi ngược chiều đối diện nhau (`|lat| < 0.15`), hướng lách trái/phải được phân định cố định theo chỉ số chẵn/lẻ (`selfIdx % 2`) để 2 người lách về 2 phía khác nhau, không bao giờ dính nhau hay xoay vòng.

### 5.3. Hệ Thống Bộ Đếm Trực Tuyến (`#hud-online` & `updateOnlineCount`)
Góc trên bên phải hiển thị thực tế theo thời gian thực:
1. **`X người đang ngồi thiền` (`#cnt`):** Đếm những ai đang thực sự ngồi yên trong trạng thái `'sit'` trên bồ đoàn (bao gồm Sư Phụ + đồng tu đang `'sit'` + người chơi nếu `mainState === 'sit'`).
2. **`Y người đang thiền hành` (`#cnt-kh`):** Đếm tất cả những ai đang lạy Phật hoặc đang di chuyển/đứng ngoài trạng thái ngồi thiền.

### 5.4. Hệ Thống Âm Thanh Nền (`Ambient ThienNCL.MP3`)
- File `Ambient ThienNCL.MP3` được khởi tạo trong `initAmbientAudio()` với `loop = true`, tự động kích hoạt ngay khi người dùng tương tác và có bộ canh giữ (watchdog) mỗi `2.5 giây` + sự kiện `ended`/`pause`/`timeupdate` để tự động phát lại nếu bị ngắt.
- Khi tiếng chuông chùa (`playTempleBellSound` bằng Web Audio API) vang lên, nhạc nền vẫn chạy song song không bị ngắt.
- Góc dưới bên trái có biểu tượng 4 cột sóng âm nhỏ `#ambient-wave-ind`: khi nhạc nền đang phát sẽ có class `.playing` (sóng vàng nhấp nhô), giúp kiểm tra nhanh trạng thái âm thanh.

---

## 6. Các Hạng Mục Chờ Phát Triển Tiếp (Roadmap Cho Phiên Sau)

1. **Chức năng `🪷 Trình Pháp (Tư vấn trực tiếp)` (`doTrinhPhap()`):**
   - Hiện tại nút `#b-trinhphap` trên thanh công cụ dưới cùng đang hiển thị thông báo: `"🪷 Chức năng đang được bổ sung"`.
   - Khi người dùng yêu cầu triển khai tính năng này ở phiên sau, hãy gắn giao diện/logic tư vấn trực tiếp hoặc hộp thoại Trình Pháp vào hàm `doTrinhPhap()` (khoảng dòng 3853 trong `index.html`).
2. **Chế độ Ánh sáng Ngày / Đêm / Tự động (`#hud-daynight`):**
   - Hiện tại đang giữ ở góc trên bên phải để phục vụ kiểm thử; khi người dùng yêu cầu ẩn đi hoặc chuyển sang tự động hoàn toàn thì chỉnh tại `#hud-daynight`.

---

## 7. Quy Tắc Vàng Khi Chỉnh Sửa (Golden Rules Cho AI)

1. **Không dùng ký tự tiếng Việt / Unicode bên ngoài chuỗi (string) hoặc comment trong thẻ `<script>`:**
   - Mọi tên biến, tên hàm trong JavaScript phải là ASCII chuẩn.
2. **Tuyệt đối không thêm cạnh `NAV_EDGES` cắt qua tường (`SOLID_WALL_SEGMENTS`) hoặc vùng cấm (`!canMoveTo`):**
   - Mọi cạnh trong `NAV_EDGES` phải nằm trên mặt đường đá (`isOnPavedSurface`) và thỏa mãn `isSegmentWalkable(ax, az, bx, bz)`.
3. **Không xóa điểm căn trước cửa (`DOORWAY_NODES`) và không `shift()` bỏ qua waypoint khi chống kẹt:**
   - Nếu nhân vật bị vướng, phải gọi lại `buildGhostWalkPath` từ tọa độ hiện tại, tuyệt đối không `waypoints.shift()` nhảy cóc qua tường.
4. **Luôn giữ vùng bảo vệ quanh Sư Phụ (`SU_PHU_POS = { x: 0.0, z: -4.85 }`, `r = 1.10m`):**
   - Không cho bất kỳ nhân vật hay GHOST nào đi xuyên qua tọa cụ của Sư Phụ.
