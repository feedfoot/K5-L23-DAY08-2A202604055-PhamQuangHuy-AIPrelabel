# Thông tin học viên

- Họ và tên: Phạm Quang Huy
- MSHV: 2A202604055
- Lớp: L2
- Tài khoản GitHub: feedfoot
- Repo bài nộp (URL): https://github.com/feedfoot/K5-L23-DAY08-2A202604055-PhamQuangHuy-AIPrelabel
- Commit chốt bài (hash): 0a3eaa8
- Máy: Windows 11 Pro, không có GPU (chạy CPU, PyTorch 2.15.0+cpu trong `.venv`)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt

- Máy: accepted / edited / deleted: 190 / 139 / 163 = 37.8% / 27.6% / 32.4% (mẫu số: 503 box máy); added: 63 = 16.1% (mẫu số: 392 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 44/70 = 62.9% / 44/70 = 62.9% (70 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 15/4 = 3.75 phút/ảnh; duyệt có pre-label 194/24 ≈ 8.1 phút/ảnh (ước lượng từ mốc thời gian, gồm cả lúc nghỉ và hỏi AI)
- Lỗi QC: vòng 1 75.0% theo ảnh (6/8), 11.5% theo đối tượng (18/156) → Rework; vòng 2 sau sửa 0.0% theo ảnh (0/8), 0.0% theo đối tượng (0/175)
- **Quyết định:** Release — lý do một dòng: QC vòng 2 sau Rework 0/8 ảnh lỗi (≤ 2%); giới hạn: mẫu 24 ảnh, cận trên Wilson 95% là 32%.
