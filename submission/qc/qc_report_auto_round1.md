# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 156 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 18/156 = 11.5% | 156 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 6/8 = 75.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 190/503 = 37.8% / 139/503 = 27.6% / 163/503 = 32.4% | 503 box máy |
| Máy: added | 63/392 = 16.1% | 392 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 16 | img_003.jpg, img_018.jpg, img_021.jpg |
| thua | 2 | img_006.jpg, img_008.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 6 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **93%**. Ngưỡng 2% không thể được **chứng minh** 

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 75.0% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 16/18 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
