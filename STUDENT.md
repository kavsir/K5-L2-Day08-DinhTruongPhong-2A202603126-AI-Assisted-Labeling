# Thông tin học viên

- Họ và tên: Đinh Trường Phong
- MSHV: 2A202603126
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: kavsir
- Repo bài nộp (URL): https://github.com/kavsir/K5-L2-Day08-DinhTruongPhong-2A202603126-AI-Assisted-Labeling
- Commit chốt bài (hash): 58aa4b4
- Máy: Windows 11, GPU NVIDIA GeForce RTX 4070 12GB (CUDA 12.8)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 479/620 (77.3%) / 20/620 (3.2%) / 102/620 (16.5%) (mẫu số: 620 box máy); added: 53/552 (9.6%) (mẫu số: 552 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 51.4% (38/74) / 73.1% (38/52 đối tượng gold)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 6.25 phút/ảnh (25 phút / 4 ảnh); duyệt có pre-label 2.71 phút/ảnh (65 phút / 24 ảnh)
- Lỗi QC: 40.0% theo ảnh (2/5), 2.0% theo đối tượng (2/101)
- **Quyết định:** Rework — lý do một dòng: Tỷ lệ bắt bẫy đạt 58.3% < 80% do lọt bẫy viền lỏng (loose) và tỷ lệ ảnh lỗi QC đạt 40.0% > 2%, cần rà soát lại để khắc phục thiên kiến tự động hóa.
