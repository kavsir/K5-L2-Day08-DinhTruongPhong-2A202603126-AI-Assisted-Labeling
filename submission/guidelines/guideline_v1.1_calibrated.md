# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603126 - Đinh Trường Phong

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xe có bàn đạp trợ lực tính là `bicycle`; xe chạy điện hoàn toàn không có bàn đạp tính là `motorbike`. | `img_001.jpg` | CAL-01 |
| 2 | Khối xe máy đỗ dày đặc ở khoảng cách trung bình và xa: từng xe hay `ignore_region`? | Xe máy ở xa có chiều cao box < 20 px gom vào 1 box `ignore_region`; xe ở gần (> 20 px) còn thấy rõ >= 20% thân xe thì phải gán từng box `motorbike` riêng. | `img_005.jpg` | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Tuk-tuk, xích lô, xe ba gác bị máy nhận nhầm do COCO đóng | YOLO26 gán thành `car` hoặc `truck` | Đổi sang `three_wheeler`, bao gồm cả mui xe và thùng gắn liền xe | Mục 2: Luôn kiểm tra xe ba bánh khi máy gán `car`/`truck` (CAL-03) |
| `img_007.jpg` | Người điều khiển xe máy/xe đạp bị máy bỏ sót | Máy chỉ vẽ 1 box xe, thiếu box người | Bắt buộc vẽ thêm box `person` cho người lái xe | Mục 4.1.4: Quy tắc kiểm tra đủ cặp `person` + `motorbike`/`bicycle` (CAL-04) |
| `img_014.jpg` | Bẫy viền lỏng quanh xe ba bánh lọt qua mắt người duyệt | Máy vẽ box rộng hơn thân xe ~30% | Phải kéo ôm sát 4 cạnh box chạm mép ngoài cùng vật thể | Mục 4.1.1: Siết chặt quy trình rà soát viền lỏng chống thiên kiến tự động hóa |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Ban đầu |
| v1.1 | Chốt quy tắc xe điện (CAL-01); ranh giới `ignore_region` cho bãi đỗ xe máy (CAL-02); hướng dẫn sửa lỗi máy nhầm tuk-tuk thành car/truck (CAL-03); siết chặt bắt buộc gán đủ box người lái (CAL-04) và siết checklist viền lỏng. | Dựa trên kết quả review 24 ảnh, lỗi lọt bẫy viền lỏng và sai số của detector YOLO26 / YOLOE. |
