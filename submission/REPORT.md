# Báo cáo Day 08 — 2A202603152 - Đinh Quang Thắng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m (pilot: `r01`, `r02`) | 0.5 | Trên 13 vật gold, TP=12, FP=6, FN=1: precision 66.7%, recall 92.3%; đạt mục tiêu recall 90% (`threshold_sweep.csv`). |
| bicycle | YOLO26m (`r01`, `r02`) | 0.3 (mặc định) | Chỉ có 1 vật gold, chưa đủ mẫu để chọn ngưỡng riêng (`threshold_sweep.md`). |
| motorbike | YOLO26m (`r01`, `r02`) | 0.3 (mặc định) | Tối đa 8/9 = 88.9% recall ở ngưỡng không quá 0.5, chưa đạt 90% (`threshold_sweep.csv`). |
| car | YOLO26m (`r01`, `r02`) | 0.3 (mặc định) | Tối đa 7/8 = 87.5% recall ở ngưỡng không quá 0.5, chưa đạt 90% (`threshold_sweep.csv`). |
| bus | YOLO26m (`r01`, `r02`) | 0.3 (mặc định) | Chỉ có 1 vật gold, chưa đủ mẫu để chọn ngưỡng riêng (`threshold_sweep.md`). |
| truck | YOLO26m (`r01`, `r02`) | 0.3 (mặc định) | Không có vật truck trong gold, không thể hiệu chỉnh ngưỡng riêng (`threshold_sweep.md`). |
| three_wheeler | YOLOE-S, prompt `three_wheeler` (pilot: `r03`) | 0.15 | Trên 7 vật gold, TP=7, FP=2, FN=0: precision 77.8%, recall 100% (`threshold_sweep.csv`). |

- Chuỗi ngưỡng để dùng khi merge: `person=0.5,three_wheeler=0.15`; các lớp khác giữ `--conf 0.3`.
- Lệnh `merge_prelabels.py`: chưa thấy log hoặc đầu ra merge trong repo để xác nhận lệnh đã chạy. Lệnh theo recipe là `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- **SAM đã chạy:** SAM 2.1 với `sam2.1_b.pt` (`r04`); tạo 16 mask trên 2 ảnh, mất 13.81 giây/ảnh trên CPU. Ở `img_004.jpg` mask bám được người đàn ông bên phải; tốc độ chậm nên chỉ dùng hỗ trợ ca khó.
- Lỗi **không sửa được bằng đổi ngưỡng:** ở `img_013.jpg`, tuk-tuk đỏ bị gán `car` và mask phủ gần hết thân xe/lẫn vùng hành khách (`r04`). Cần sửa lớp và rà lại ranh giới mask bằng tay.
- Box cần kiểm tra: ở `img_002.jpg`, YOLOE cho box `three_wheeler` điểm 0.37 trên vùng xe trắng bên phải; xác minh đó có phải xe ba bánh hay false positive trước khi giữ (`r03`).
- **Giới hạn mẫu:** quét ngưỡng dùng đúng 4 ảnh gold; ngưỡng được chọn và số precision/recall được tính trên cùng tập này nên số đo trên gold có tính lạc quan, không đại diện chắc chắn cho ảnh mới. Các lớp chỉ có 0–1 vật gold càng không đủ dữ liệu để kết luận; xem `threshold_sweep.csv`.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 322 | 322/429 = 75.1% | 429 box máy |
| edited (sửa) | 23 | 23/429 = 5.4% | 429 box máy |
| deleted (xoá) | 61 | 61/429 = 14.2% | 429 box máy |
| added (vẽ thêm) | 44 | 44/389 = 11.3% | 389 đối tượng sau duyệt |

- **three_wheeler** là lớp máy yếu nhất theo profile duyệt: chỉ 3/19 box máy được giữ nguyên; 5 box phải sửa, 11 box bị xoá, nên 16/19 box (84.2%) không được giữ nguyên. Ngoài ra còn thêm 11 vật bằng tay. Có thể do nhầm lớp/viền với xe ô tô hoặc tuk-tuk như trường hợp `img_013.jpg` đã ghi ở mục 1.
- Person chiếm nhiều lỗi theo quy mô: 34 box bị xoá trên 236 box máy của lớp, nhưng phần lớn 189/236 box vẫn được giữ nguyên. Motorbike có 18/44 vật được thêm vào (thêm 40.9%), cho thấy rủi ro bỏ sót tương đối cao; còn 76/85 box motorbike được giữ nguyên, 5 được sửa và 4 bị xoá.
- Trong 61 box bị xoá và 44 box được thêm, có 8 trường hợp đổi lớp (box đúng chỗ nhưng sai lớp), không phải lỗi bỏ sót. Vì vậy số vật máy **sót thật** ước tính là 44 − 8 = 36.
- So với gold: máy đạt **precision 53.6%** (30 TP, 26 FP trên 56 dự đoán) và **recall 76.9%** (30 TP, 9 FN trên 39 vật gold). KTC 95% lần lượt khoảng 41–66% và 62–87%; mẫu chỉ 39 vật trong 4 ảnh gold nên kết quả có độ lạc quan do tính in-sample.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 = 75.0%** (ngưỡng mục tiêu 80%; KTC 95% khoảng 47–91%). Kết quả chưa đạt ngưỡng dù chỉ chênh 1/12; khoảng tin cậy rộng nên không nên dựa đơn thuần vào phần trăm.
- Ba bẫy bị lọt: `img_021.jpg` (wrong_class), `img_029.jpg` (loose) và `img_009.jpg` (missing). Theo từng loại: extra 3/3 bắt được, loose 2/3, missing 2/3, wrong_class 2/3. Đây cho thấy bước duyệt cần quét ranh giới/thừa thiếu và kiểm tra lại lớp trước khi accepted, chứ không chỉ kiểm tra box có “trông hợp lý”.
- Accepted cao 322/429 (75.1%) trong khi bắt bẫy thấp hơn mục tiêu cho thấy người duyệt có xu hướng tin vào pre-label và chưa quét đủ độc lập theo checklist. Ngược lại, 61 box bị xoá và 44 vật được thêm cho thấy vẫn có tín hiệu kiểm soát ở mức tổng thể, nhưng thiên vị được thể hiện rõ nhất ở các lỗi bỏ sót, sai ranh giới và sai lớp.
- Vì danh sách 25 đối tượng quanh bẫy đã được loại khỏi thống kê review, profile trên mục 2 phản ánh lỗi bình thường chứ không bị trộn với lỗi nhân tạo được cố ý tạo ra.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): file hiện không có bản ghi, nên không thể tính gán tay **x phút/ảnh** hay duyệt **y phút/ảnh**. Không thể chứng minh lợi ích tốc độ bằng số từ dữ liệu hiện có.
- Thiên lệch: gold được gán trước, sau đó mới dùng gold để chọn ngưỡng và duyệt bẫy. Vì vậy có nguy cơ người duyệt đã quen cảnh và làm cho kết quả tự đánh giá quá lạc quan; kết quả gold không nên xem là độ chính xác độc lập.
- Kết luận: máy **chỉ đáng dùng có kiểm soát** để tạo pre-label hỗ trợ, không nên dùng tự động release toàn bộ. Precision 53.6% và recall 76.9%, cùng 61/429 (14.2%) box bị xoá và 44 vật được thêm, cho thấy vẫn cần con người; `person` tương đối khả dụng hơn, còn `three_wheeler` là phần cần xem xét hoặc gán tay, còn motorbike và các lớp thiếu mẫu gold cần rà soát thêm.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quét ảnh trước khi xem box:** đối chiếu toàn bộ vật trên ảnh với checklist/ba loại bẫy, ghi log từng quyết định; mục tiêu bắt lại ≥80%/12 bẫy và phải không để tràn các 12 trường hợp.
  2. **Không chuyển tất cả vào accepted chỉ vì box trông hợp lý:** kiểm tra vị trí, kích thước, lớp và ranh giới; đặc biệt với `three_wheeler` và các box sai lớp, lấy mẫu các box bị deleted/edited và số lần hồi quy <5% cho mỗi lớp.
  3. **Dùng mẫu độc lập để xác nhận gold và hiệu suất:** không dùng lại chính 4 ảnh gold có chọn ngưỡng để báo recall cuối; giữ 30 TP/26 FP/9 FN làm kết quả thô nhưng yêu cầu thêm ảnh ngoài tập này cho precision/recall nếu precision <80% hoặc recall <90%.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` — tuk-tuk đỏ (vật 1) | SAM 2.1 Base (`sam2.1_b.pt`) | 2 điểm dương (`400,480`, `600,110`) + 1 điểm âm (`720,300`) | Không tách sạch người lái/hành khách: mask che phần thân xe và vẫn bao khu vực người lái/hành khách; hàng hoá xung quanh không được tách thành vật riêng. | SAM 2.1 giữ phần thân/mái/cửa/ bánh tương đối rộng, nhưng điểm âm chỉ khoét một phần lưng áo người lái; đầu/chân người vẫn dính mask. | `submission/masks/tuktuk_img013_sam2.jpg`; bản trùng nội dung: `submission/masks/tuktuk_img013.jpg` |
| `img_013.jpg` — tuk-tuk đỏ (so sánh SAM 1 gọn) | MobileSAM/SAM 1 (`mobile_sam.pt`) | 2 điểm dương (`400,480`, `600,110`) + 1 điểm âm (`720,300`) | Mask vẫn bao người lái/hành khách; không tạo ranh giới riêng cho người hay hàng hoá. | Bản SAM 1 che rộng vùng ghế/người lái và có phần mask dính nút rèm/nền; tách người lái kém hơn việc chỉ cần một điểm âm. | `submission/masks/tuktuk_img013_sam1.jpg` |
| `img_004.jpg` — xe ba bánh đỏ (vật 2) | SAM 2.1 Base (`sam2.1_b.pt`) | 2 điểm dương (`620,480`, `700,560`) + 1 điểm âm (`465,450`) | Mask chính ôm xe ba bánh; điểm âm ở người bên trái giúp tránh người lái, nhưng ảnh vẫn cần kiểm tay vì người/bóng nền có thể tiếp giáp mask. | Bám tốt phần thân xe và bánh; không che rõ người đàn ông bên phải ngoài mask, nhưng không nên dùng chưa rà thủ công ranh giới giữa người và phương tiện. | `submission/masks/tuktuk_img004_sam2.jpg` |
| `img_017.jpg` — tuk-tuk đỏ (vật 3) | SAM 2.1 Base (`sam2.1_b.pt`) | 2 điểm dương (`560,420`, `680,500`) | Mask bao phần ba bánh, cabin và bánh xe; không có điểm âm để loại người đi bộ/hàng hoá ở cạnh người ngoài mask. | Hai điểm dương tạo mask khít phần xe, nhưng gần hai phụ kiện và nền ở cạnh; người đi bộ bên trái không nằm trong vùng mask chính. | `submission/masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: 5 file mask bao phủ đủ ba ca đối tượng `img_013.jpg`, `img_004.jpg`, `img_017.jpg`; quy tắc “mask phương tiện chỉ ôm khung xe, mái và bánh xe; người lái/hàng hoá được tách riêng bằng điểm âm” được kiểm chứng trong cả 3 ảnh. `img_013_sam2/sam1` cho thấy một điểm âm chưa đủ để tách người lái khỏi cabin; cần thêm điểm âm và rà tay hoặc dùng box. SAM 2.1 Base bám xe tốt hơn ở bề rộng ranh giới, còn MobileSAM/SAM 1 ở cùng tọa độ vẫn có mask dính vùng người/nền.

## 6. Calibration (Part 6)

- Số quyết định: 5 dòng (`CAL-01` đến `CAL-05` trong `submission/qc/calibration_decisions.csv`), gồm 1 `loi_may`, 3 `bay_lot` và 1 `cau_hoi_mo`.
- Quyết định quan trọng nhất: `CAL-01` ở `img_013.jpg`: tuk-tuk bị YOLO26 gán sai `car`, nên gán lại `three_wheeler` và tách người/hành khách khỏi mask xe.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt cách phân biệt xe đạp điện/xe máy điện chưa đủ gold, quy tắc box/mask cho hàng và người, quy tắc không gộp box chồng nhau, kiểm lớp ở `wrong_class`, ôm sát box ở `loose`, quét toàn ảnh trước khi duyệt ở `missing`, và dùng `can_review` chỉ khi chưa chắc quy tắc.
- Nguyên nhân gốc: hỗn hợp máy và guideline; `img_013.jpg` và các box sai lớp thuộc lỗi máy/quy tắc ánh xạ lớp, trong khi xe không rõ lớp ở `img_002.jpg` và ba cột ranh giới/phân loại thuộc câu hỏi mở hoặc guideline cần cụ thể hơn.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 0/71 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%). Các dòng QC đều là `khong`, không ghi nhận lỗi trong 71 đối tượng đã kiểm.
- Top lỗi: không ghi nhận loại lỗi nào trong mẫu QC, do đó chưa có ảnh ví dụ lỗi cho bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính hoặc sai mask. Tuy nhiên, QC không phủ 3 bẫy đã lọt `img_021.jpg`, `img_029.jpg`, `img_009.jpg`.
- **Giới hạn mẫu nhỏ** (cận upper 95% trong `qc_report_auto.md`): 0 ảnh lỗi/5 ảnh mẫu có cận trên Wilson khoảng 43%; vì vậy mẫu 5 ảnh không thể chứng minh rằng tỉ lệ ảnh lỗi thật ≤ 2%.
- Script không thể tự kiểm âm thầm theo ý nghĩa: bộ công cụ chỉ đối chiếu phiếu được người điền, đếm loại lỗi và kiểm trùng mẫu; script không phát hiện ôm sát, cùng bỏ sót với máy hay vi phạm checklist nếu người kiểm không tự rà trên overlay. Vì vậy kết quả QC phụ thuộc phiếu bằng mắt `work/qc/review/*.jpg`, không thay thế duyệt độc lập.

**Quyết định: Rework**, ngày 2026-11-19. Lý do theo cây quyết định: không dừng ở Hold vì không ghi nhận guideline chưa rõ hoặc dữ liệu hỏng; không Release do bắt bẫy chỉ đạt 9/12 = 75% < ngưỡng 80%, mẫu QC không chứng minh được ngưỡng 2% và người duyệt vừa là người kiểm. Theo quy tắc script, bắt bẫy dưới 80% làm lô duyệt chưa đáng tin.
Nếu Rework: rà lại checklist cả lô, ưu tiên ba bẫy lọt `img_021.jpg` (wrong_class), `img_029.jpg` (loose), `img_009.jpg` (missing), sửa lỗi, cập nhật `qc_fixed.json` nếu nhãn thay đổi, chạy lại `build_release.py`, tạo lại mẫu bằng `qc_sample.py --force --n 5 --seed 42`, điền phiếu QC mới và chỉ xem xét Release khi bắt bẫy ≥ 10/12 và kết quả QC tự chấp nhận.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): đã dùng AI để so sánh các giá trị kết quả từ các file tránh thiếu sót và sai.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? `gold_manual.json` được tạo riêng theo quy trình 4 ảnh và không nằm trong mẫu sau duyệt.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows trên máy ASUS, NVIDIA GeForce RTX 3050 Laptop GPU (4 GB, CUDA) và AMD Radeon(TM) Graphics; CVAT Docker v2.74.1 tại `http://localhost:8080`. SAM chạy trên CPU mất 13.81 giây/ảnh; chưa ghi nhận vướng mắc cài đặt bổ sung.
- Dùng cache `data/cache/` thay vì tự chạy? Có dùng cache/model output có sẵn trong `data/cache/` và các file work khi cache đủ để tham chiếu kết quả chạy.
- Lỗi / bug script đã gặp (nếu có): ban đầu `work/qc/review/` chưa có file overlay, sau đó đã chạy `python scripts/qc_report.py` khi cần và `python scripts/qc_sample.py --n 5 --seed 42` để sinh lại overlay và phiếu mẫu. Không ghi nhận bug làm sai dữ liệu.
- Một điều bạn làm khác lần sau: tạo log thời gian `gold_manual` và `batch_review`, bám checklist bốn bước khi duyệt, và tạo đủ ba mask/vật trước khi calibration để tránh kết luận dựa trên một ảnh mask.
