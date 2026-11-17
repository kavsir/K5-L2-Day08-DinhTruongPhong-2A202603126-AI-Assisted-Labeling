# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 620; đối tượng sau duyệt: 552; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 479 | 479/620 = 77.3% | 620 box máy |
| edited (sửa) | 20 | 20/620 = 3.2% | 620 box máy |
| deleted (xoá) | 102 | 102/620 = 16.5% | 620 box máy |
| added (vẽ thêm) | 53 | 53/552 = 9.6% | 552 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 42.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 253 | 3 | 14 | 16 |
| bicycle | 5 | 0 | 2 | 3 |
| motorbike | 160 | 11 | 61 | 9 |
| car | 46 | 0 | 3 | 18 |
| bus | 2 | 0 | 1 | 3 |
| truck | 3 | 0 | 2 | 3 |
| three_wheeler | 10 | 6 | 19 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 52 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 38 | 36 | 14 | 51.4% (KTC 95%: 40–62%) | 73.1% (KTC 95%: 60–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 0/3
  - missing: 2/3
  - wrong_class: 2/3
  - lọt: img_022.jpg (loose), img_006.jpg (missing), img_017.jpg (loose), img_004.jpg (wrong_class), img_014.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
