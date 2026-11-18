# Báo cáo QC tự động 

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 175 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/175 = 0.0% | 175 đối tượng trong mẫu |
| **Ảnh có lỗi**  | 0/8 = 0.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 190/503 = 37.8% / 139/503 = 27.6% / 163/503 = 32.4% | 503 box máy |
| Máy: added | 63/392 = 16.1% | 392 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Giới hạn thống kê
- 0 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **32%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy
## Gợi ý quyết định
**RELEASE**

- ảnh lỗi 0.0% <= 2%, không lỗi hệ thống, bắt bẫy / nhất quán đạt.

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
