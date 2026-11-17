# Báo cáo Day 08 — 2A202603126 - Đinh Trường Phong

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Theo lớp: `motorbike=0.1`, các lớp khác dùng mặc định `0.3` | `r02`, `r03`: YOLO26 phát hiện rất nhạy person và motorbike. Tại threshold sweep, `motorbike=0.1` đạt recall 100% trên gold. Các lớp person, car, bicycle không đạt recall 90% ở conf <= 0.5 hoặc số mẫu gold < 5 nên giữ ngưỡng chuẩn 0.3. |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | 0.15 | `r04`: YOLO26 hoàn toàn không biết lớp xe ba bánh. YOLOE với text prompt phát hiện tốt tuk-tuk/xích lô, tại threshold sweep đạt recall 100%, precision 78% trên 7 vật gold. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 ViT-Base (`sam2.1_b.pt`)**. Máy có GPU NVIDIA RTX 4070 12GB VRAM mạnh mẽ, SAM 2.1 base mang lại sự cân bằng hoàn hảo giữa chất lượng đường biên phân đoạn sắc nét, tốc độ xử lý nhanh (~2.2s/ảnh), vượt trội hoàn toàn so với `mobile_sam.pt` của SAM 1 và không vướng rào cản tài khoản gated của SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận nhầm tuk-tuk, xích lô thành `car` hoặc `truck` của YOLO26 không thể sửa bằng cách thay đổi ngưỡng, vì danh mục huấn luyện của YOLO26 (COCO 80 lớp) không hề có nhãn `three_wheeler`. Cách sửa: Kết hợp mô hình open-vocabulary YOLOE thông qua `merge_prelabels.py` (ưu tiên nhãn của YOLOE đè lên box của YOLO26) và reviewer trực tiếp sửa đổi nhãn trên CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Ở `img_013.jpg`, YOLO26 phát hiện chiếc xe tuk-tuk với conf > 0.85 nhưng gán nhãn là `car` (điểm tin cậy rất cao nhưng sai bản chất lớp).
  - *Điểm thấp mà đúng:* Ở `img_032.jpg`, YOLOE phát hiện xe ba gác ở góc khuất với conf chỉ 0.27 (dưới ngưỡng 0.3 thông thường) nhưng thực tế đúng là một chiếc `three_wheeler`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Vì tập dữ liệu quét ngưỡng và đánh giá trùng nhau (in-sample) với chỉ 4 ảnh (52 đối tượng), con số precision/recall của máy trên gold mang tính ước lượng thô và lạc quan. Hơn nữa, các lớp hiếm (bus, truck có 0 vật trong gold; bicycle chỉ có 4 vật) không đủ kích thước mẫu thống kê để tối ưu ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 479 | 479/620 = 77.3% | 620 box máy |
| edited | 20 | 20/620 = 3.2% | 620 box máy |
| deleted | 102 | 102/620 = 16.5% | 620 box máy |
| added | 53 | 53/552 = 9.6% | 552 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Máy thể hiện tệ nhất ở lớp `three_wheeler` (bị xóa 19/35 box đề xuất, sửa 6 box, chỉ accepted 10 box) và lớp `motorbike` (bị xóa 61 box do bắt nhầm các đốm nhỏ hậu cảnh). Tại `img_013.jpg` và `img_002.jpg`, detector thường xuyên bị nhầm lẫn giữa thân xe ba bánh và xe ô tô/người lái.
- So với gold: precision / recall của **máy**: Precision đạt **51.4%** (38 TP / 74 box dự đoán của máy, KTC 95%: 40–62%); Recall đạt **73.1%** (38 TP / 52 đối tượng gold, KTC 95%: 60–83%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **7/12 = 58.3%** (KTC 95%: 32–81%); loại nào lọt: Lọt 3/3 bẫy viền lỏng (`loose` tại `img_022.jpg`, `img_017.jpg`, `img_014.jpg`), 1 bẫy bỏ sót (`missing` tại `img_006.jpg`), và 1 bẫy sai lớp (`wrong_class` tại `img_004.jpg`). Bắt được 3/3 bẫy vẽ thừa (`extra`).
- Vì sao (bạn đã bỏ bước nào của checklist)? Tôi đã bỏ sót bước siết chặt 4 cạnh viền box (tighten loose boxes) và bước quét mắt toàn cảnh trước khi bật nhãn máy. Khi thấy một vật thể đã có sẵn bounding box bao quanh, mắt người có xu hướng mặc định là "đã được gán" mà không zoom vào kiểm tra xem cạnh viền có ôm sát hay bị thừa quá nhiều khoảng trống.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỷ lệ accepted đạt tới 77.3% trong khi tỷ lệ bắt bẫy chỉ đạt 58.3% (đặc biệt lọt 100% bẫy loose) phản ánh rõ nét **Thiên kiến tự động hóa (Automation Bias)**: Reviewer có tâm lý tin tưởng và ỷ lại vào các đề xuất sẵn có của AI, chỉ tập trung xóa các box rác lộ liễu chứ chưa thực sự khắt khe về độ chính xác hình học của các box máy gợi ý.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **6.25 phút/ảnh** (25 phút / 4 ảnh) vs duyệt có pre-label **2.71 phút/ảnh** (65 phút / 24 ảnh). Thiên lệch: Reviewer gán 4 ảnh gold trước nên đã quen mắt với đặc trưng hình ảnh đường phố châu Á, do đó tốc độ duyệt ở lô 24 ảnh có phần nhanh hơn so với người bắt đầu hoàn toàn mới.
- Kết luận: **CÓ ĐÁNG DÙNG**, giúp tiết kiệm hơn 56% thời gian lao động, đặc biệt hiệu quả cao cho các đối tượng phổ thông rõ nét (`person`, `car`, `motorbike`). Tuy nhiên, không được phó mặc cho máy ở các lớp xe đặc thù địa phương (`three_wheeler`) hoặc các vùng xe cộ đông đúc bị che khuất.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc Ẩn nhãn máy trước (Blinded Initial Scan):* Luôn tắt hiển thị layer annotation máy trong 5–10 giây đầu tiên để mắt tự quét và đếm nhẩm các đối tượng chính trong ảnh trước khi bật nhãn máy lên so sánh.
  2. *Quy tắc Kiểm tra viền 4 cạnh (Edge Snapping Audit):* Phóng to (zoom) tối thiểu 200% đối với ít nhất 30% số box máy đề xuất để kiểm tra xem 4 cạnh viền có chạm sát mép ngoài cùng của vật thể hay không, chống hiện tượng lọt bẫy viền lỏng.
  3. *Quy tắc Kiểm tra cặp đôi Người - Xe (Person-Vehicle Coupling):* Với mỗi phương tiện hai bánh (`motorbike`, `bicycle`) đang lưu thông, bắt buộc kiểm tra xem đã có đủ cặp 2 box độc lập (1 box xe và 1 box `person` cho người điều khiển) hay chưa.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Gồm trọn vẹn thân xe tuk-tuk và khung mui; tách rời người đi bộ sát cạnh nhờ click âm | Lúc đầu chưa có click âm, mask bị loang dính sang người đứng cạnh xe | `submission/masks/tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Gồm thân xe xích lô (cyclo) và bánh trước; tách biệt người đạp phía sau | Các nan hoa và khoảng trống nhỏ ở bánh xe bị gộp kín vào mask | `submission/masks/cyclo_img004.jpg` |
| `img_002.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 1 box gợi ý + 2 click (1 pos, 1 neg) | Gồm thân xe ba gác máy và mui; không bao gồm hàng cồng kềnh nhô cao | Sử dụng box gợi ý giúp SAM cố định vùng quan tâm, tránh loang ra mặt đường | `submission/masks/three_wheeler_img002.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc cho phép bỏ qua lỗ thủng < 5% diện tích giúp ích rất nhiều cho SAM khi phân đoạn căm xe/nan hoa của xích lô. Tuy nhiên, guideline còn thiếu quy định phân tách chi tiết giữa người lái và thân xe ba bánh dạng mui hở khi cả hai đan xen vào nhau.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: CVAT Docker cục bộ có tích hợp sẵn mô hình SAM ViT-H qua Nuclio serverless (`cvat.pth.facebookresearch.sam.vit_h:latest-gpu`). Thao tác click trực tiếp trên web UI tạo polygon tức thì, rất trực quan cho người dùng; tuy nhiên so với script offline `sam2_points.py`, Interactor trên web tiêu tốn nhiều tài nguyên hơn và polygon xuất ra có quá nhiều đỉnh (> 100 điểm) gây nặng file dữ liệu.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** (`CAL-01` đến `CAL-04` trong `calibration_decisions.csv`); quan trọng nhất: `CAL-03` (sửa triệt để việc detector YOLO26 gán nhầm tuk-tuk và xe ba gác thành `car` hoặc `truck`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung quy định phân loại xe điện dựa vào bàn đạp trợ lực (CAL-01); xác định ranh giới cao < 20 px để gom cụm bãi đỗ xe máy vào `ignore_region` (CAL-02); hướng dẫn nhận diện và sửa lỗi máy nhầm xe ba bánh (CAL-03); bắt buộc gán đủ cặp box người lái xe máy/xe đạp (CAL-04) và siết chặt quy trình kéo viền ôm sát vật thể.
- Nguyên nhân gốc: Đến từ cả ba khía cạnh: (1) Guideline v1.0 để mở nhiều câu hỏi thực tế chưa được định nghĩa; (2) Mô hình AI bị giới hạn danh mục nhãn cố định; và (3) Người duyệt bị ảnh hưởng bởi thiên kiến tự động hóa dẫn đến bỏ sót lỗi viền lỏng.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ca khó + 5 ngẫu nhiên, seed 42), gồm 101 đối tượng được kiểm tra lại độc lập; người kiểm: học viên (**hạn chế**: người kiểm tra QC cũng chính là người review lô, có thể tồn tại điểm mù đánh giá chủ quan).
- Lỗi theo **đối tượng**: **2/101 = 2.0%** · theo **ảnh**: **2/5 = 40.0%** (ngưỡng Release yêu cầu ≤ 2%).
- Top lỗi: Bỏ sót đối tượng (`bo_sot` = 1 lỗi tại `img_006.jpg` do sót vật ở sát mép ảnh) và sai vị trí kích thước (`sai_vi_tri_kich_thuoc` = 1 lỗi tại `img_014.jpg` do viền box xe lỏng lẻo chưa kéo ôm sát).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 2 ảnh lỗi trên 5 ảnh mẫu, cận trên 95% khoảng tin cậy Wilson của tỷ lệ ảnh lỗi thật lên tới **77%**. Kích thước mẫu 5 ảnh là quá nhỏ để có thể chứng minh một cách có ý nghĩa thống kê rằng tỷ lệ lỗi của toàn bộ lô nhỏ hơn 2%.
- Script kiểm được / không kiểm được: Script kiểm tra tự động được định dạng COCO, sự tồn tại của nhãn, kiểm tra toạ độ vượt mép ảnh và tính toán IoU; script KHÔNG kiểm tra được ngữ nghĩa đời thực (liệu chiếc xe ba gác có bị gọi là ô tô hay không) nếu không có nhãn ground truth đối sánh.

**Quyết định: REWORK**, ngày 2026-11-17. Lý do (đi theo cây Hold → Rework → Release):
1. Guideline đã được làm rõ và hoàn thiện ở bản v1.1, không còn câu hỏi mở gây tắc nghẽn -> Không chọn HOLD.
2. Tỷ lệ bắt bẫy chỉ đạt **58.3% (7/12) < ngưỡng tối thiểu 80%**, đặc biệt lọt 100% bẫy viền lỏng (`loose`), đồng thời tỷ lệ ảnh có lỗi trong mẫu QC đạt **40.0% > ngưỡng 2%**. Điều này cho thấy lô nhãn chưa đạt tiêu chuẩn chất lượng hình học và có dấu hiệu thiên kiến tự động hóa -> **Quyết định REWORK**.

Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại toàn bộ 24 ảnh của lô batch, tập trung zoom in siết chặt 4 cạnh của các box bị lỏng/rộng quanh xe cộ, quét kỹ các mép ảnh để bổ sung các đối tượng bị bỏ sót, và áp dụng nghiêm ngặt các quy tắc đã được hiệu chuẩn trong guideline v1.1.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng trợ lý AI để hỗ trợ giải thích các khái niệm trong bài lab (IoU, Automation Bias, Calibration), gợi ý lệnh terminal tương tác với CVAT Docker và chạy script thống kê; toàn bộ việc gán tay 4 ảnh gold, review 24 ảnh trên CVAT và đánh giá số liệu đều do học viên tự thực hiện và đối chiếu trực tiếp.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Hoàn toàn KHÔNG bị nhiễm**. 4 ảnh gold được gán tay thủ công độc lập trên CVAT từ đầu, trước khi chạy bất kỳ mô hình AI nào lên các ảnh này.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce RTX 4070 12GB VRAM, CUDA 12.8, PyTorch 2.11; CVAT v2.74.1 chạy qua Docker Desktop. Quá trình chạy mô hình diễn ra rất nhanh và ổn định.
- Dùng cache `data/cache/` thay vì tự chạy? **Không dùng cache**. Tự chạy 100% các lệnh sinh pre-label cục bộ trên GPU máy.
- Lỗi / bug script đã gặp (nếu có): Chú ý định dạng truyền tham số ảnh trong PowerShell cần dùng khoảng trắng thay vì dấu phẩy để script nhận diện đúng danh sách file.
- Một điều bạn làm khác lần sau: Sẽ dành thời gian kiểm tra kỹ 4 cạnh viền của từng box thay vì quá chú ý vào việc xóa box thừa, nhằm khắc phục triệt để thiên kiến tự động hóa dạng chấp nhận viền lỏng.
