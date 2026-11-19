# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 429; đối tượng sau duyệt: 389; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 322 | 322/429 = 75.1% | 429 box máy |
| edited (sửa) | 23 | 23/429 = 5.4% | 429 box máy |
| deleted (xoá) | 61 | 61/429 = 14.2% | 429 box máy |
| added (vẽ thêm) | 44 | 44/389 = 11.3% | 389 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 36.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 189 | 13 | 34 | 6 |
| bicycle | 6 | 0 | 3 | 1 |
| motorbike | 76 | 5 | 4 | 18 |
| car | 44 | 0 | 4 | 4 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 2 |
| three_wheeler | 3 | 5 | 11 | 11 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 39 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 30 | 26 | 9 | 53.6% (KTC 95%: 41–66%) | 76.9% (KTC 95%: 62–87%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 2/3
  - missing: 2/3
  - wrong_class: 2/3
  - lọt: img_021.jpg (wrong_class), img_029.jpg (loose), img_009.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
