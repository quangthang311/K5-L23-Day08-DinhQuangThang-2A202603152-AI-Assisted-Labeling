# Label contract — Phố châu Á — BỔ SUNG v1.1 — Đinh Quang Thắng - 2A202603152

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ thuộc `bicycle` hay `motorbike`? | Chưa đủ mẫu riêng trong gold để chốt khác v1.0; dùng bề ngoài động cơ/khung xe: xe đạp có khung đạp và truyền lực đạp thì `bicycle`, xe máy hai bánh có động cơ và không có bàn đạp thì `motorbike`. Nếu không nhìn rõ nền hoặc định danh, chọn lớp tin nhất và tick `can_review`, không chọn ngẫu nhiên. | `img_002.jpg` | `CAL-05` |
| 2 | Hàng nhô ra của xe đạp/xe ba gác có thuộc box xe không; mask của phương tiện và người/hàng hoá tách thế nào? | Box xe gồm phần thân xe, mái che, bánh xe và hàng gắn liền nhìn thấy; người lái/hành khách/hàng nhô rời không gộp vào box/mask xe. Hàng gắn cố định thuộc xe; người tách thành `person`. | `img_013.jpg` | `CAL-01` |
| 3 | Box máy chồng nhau hoặc ranh giới không chắc thì giữ hay gộp? | Giữ từng vật nhìn thấy riêng; không gộp nhiều box thành một đối tượng. Khi không chắc do che khuất/chồng box, chọn lớp tin nhất và tick `can_review` để calibration, không giữ box lỏng hoặc box trùng. | `img_029.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Tuk-tuk đỏ gần camera; người lái/hành khách che; SAM bám vật có ranh giới phức tạp | YOLO26 gán tuk-tuk là `car`; SAM 2.1 Base phủ gần hết thân xe và lẫn vùng hành khách (`r04`) | Gán chiếc xe là `three_wheeler`; người/hành khách gán riêng bằng box `person`; mask ôm phần xe/mái/bánh, không bao người/hàng rời, rà lại thủ công | Bổ sung mục 4.2 (polygon/mask phương tiện) và 4.3 (phân lớp); case `CAL-01` |
| `img_021.jpg` | Bẫy wrong_class lọt; xe đúng vị trí nhưng sai tên lớp | Máy gán xe ô tô là `truck` thay vì `car` | Giữ box nếu vị trí và viền đúng nhưng sửa lớp thành `car`; không đổi lớp chỉ dựa vào kích thước | Bổ sung checklist “soi nhãn và 4 cạnh” vào mục 4.1; case `CAL-02` |
| `img_029.jpg` | Bẫy loose lọt; box nới lỏng và chứa nền dư | Box máy che không khít vật | Vẽ lại/cắt box theo phần nhìn thấy; không giữ box nếu viền hở hoặc IoU không đạt ngưỡng accepted 0.9 | Bổ sung tiêu chí ranh giới box và rà bẫy loose vào mục 4.1; case `CAL-03` |
| `img_009.jpg` | Bẫy missing lọt; vật rõ bị máy bỏ sót | Không có box máy tại vật bị sót | Quét toàn ảnh trước, sau đó mới đối chiếu box để thêm vật thiếu; vật rõ ràng ngoài vùng ignore không được bỏ | Bổ sung bước 1 checklist “quét toàn ảnh trước khi nhìn nhãn máy” vào mục 4.1; case `CAL-04` |
| `img_002.jpg` | Box `three_wheeler` có conf thấp trên xe trắng; ca không chắc lớp | YOLOE-26s tạo box độ tin cậy 0.37 trên xe trắng bên phải | Chỉ gán `three_wheeler` khi ba bánh/khung ba bánh nhìn thấy đáng tin; không đếm được bánh thì tick `can_review`, không accepted theo conf | Bổ sung quy tắc use-of-can_review và ngưỡng tin cậy định tính vào mục 4.3; case `CAL-05` |

Ghi chú: `submission/labels/batch_reviewed.json` hiện không có annotation nào bật `can_review=true`. Các ca `can_review` ở đây là quy tắc sẽ áp dụng; không được tick `can_review` cho mọi box khó vẽ.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt ba câu hỏi mở; thêm quy tắc mask/box, kiểm lớp, quét missing và dùng `can_review` chỉ cho ca không chắc quy tắc | `CAL-01` đến `CAL-05`; lấy từ lỗi máy, ba bẫy lọt và box mơ hồ |
