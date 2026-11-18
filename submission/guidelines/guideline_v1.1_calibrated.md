# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604055 - Phạm Quang Huy

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Xe cao ≥ 20 px và còn thấy ≥ 20% thì gán từng xe, kể cả khi box chồng nhau. `ignore_region` chỉ dùng khi không tách được từng xe. | img_006 (#55, #60, #70) | CAL-03 |
| 3 | Hàng nhô ra / hàng mang theo | Giữ quy tắc 4.1.5 cho xe (hàng nhô quá ~20% chiều dài xe không tính). Thêm: hàng người mang hoặc vác (bao tải, gánh) **không** tính vào box `person`. | img_022 | CAL-05 |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe? | Kính **thuộc** mask xe; nội thất nhìn qua kính không làm khoét mask. Người lái vẫn là vật riêng, phải tách khỏi mask xe. | img_004, img_017, img_013 | CAL-04 |
| 7 | Mức conf tối thiểu để chấp nhận box máy không sửa? | **Không đặt.** Mọi box máy đều phải nhìn ảnh; `conf` chỉ để xếp thứ tự duyệt. | img_002, img_011 | CAL-02 |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì | Quyết định của bạn | Quy tắc mới |
|---|---|---|---|---|
| img_011, img_013, img_007, img_032 | Tuk-tuk, xích lô | YOLO26m gọi `truck` 0.77 (img_011), `car` 0.42 (img_013), `motorbike` (img_007), `bicycle` 0.70 (img_032) | Luôn `three_wheeler`; YOLOE-26s được gộp trước để thắng khi chồng box | Mục 2: xe ba bánh có mui hoặc có người đạp phía sau vẫn là `three_wheeler`, dù máy gọi car/truck/bicycle với conf cao (CAL-01) |
| img_022 | Người đội nón vác bao tải | Box máy bao chung người và bao hàng (bẫy loose) | Box `person` chỉ ôm thân người | Mục 2 (`person`): hàng mang theo không thuộc box người (CAL-05) |
| img_003, img_008 | Xe đẩy / xe hàng rong chở rau, chuối | YOLOE-26s gọi `three_wheeler`; không rõ có đúng lớp | Đếm bánh theo 4.3: 3 bánh → `three_wheeler`; không thấy đủ bánh → giữ `three_wheeler` và tick `can_review` | Mục 2 (`three_wheeler`): thêm xe hàng rong 3 bánh; mục 4.3 (CAL-06) |
| img_018, img_021, img_035 | Người / xe nhỏ trong cảnh đông, người ngồi vỉa hè | YOLO26m không có box; người duyệt cũng bỏ sót (16/18 lỗi QC là `bo_sot`) | Sau khi duyệt box máy, quét thêm một lượt theo lưới tìm vật cao ≥ 20 px chưa có box; cụm ≥ 5 vật cao < 20 px → `ignore_region` | Mục 4.1.6 + checklist duyệt: thêm bước tìm vật sót (CAL-07) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 3, 5, 7 của mục 6; thêm quy tắc xe ba bánh máy gọi sai lớp, hàng mang theo không thuộc `person`, xe hàng rong 3 bánh, bước quét tìm vật sót sau khi duyệt | CAL-01 → CAL-07 |
