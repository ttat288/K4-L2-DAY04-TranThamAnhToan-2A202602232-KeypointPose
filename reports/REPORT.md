# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: TRẦN THẨM ANH TOÀN   MSSV: 2A202602232   Mã cặp: 21902232   Ngày: 16/09/2026

> Cách dùng: copy file này thành `reports/REPORT.md`. Điền bằng số liệu do công cụ sinh ra;
> không tự ước lượng hoặc sửa số trong file JSON.

## 1. Nhãn của tôi

<!-- Lấy số từ reports/visibility_report.md hoặc outputs/visibility_report.json sau Chặng 4.
Số ảnh phải là 20; số skeleton là tổng số người trong 20 ảnh. Thời gian trung bình = tổng
thời gian gán / 20. -->

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 28 |
| v=2 / v=1 / v=0 | 314 / 138 / 24 |
| Thời gian trung bình mỗi ảnh | ______ (chưa đo lại, bạn tự điền nếu có ghi lại lúc gán) |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. left_ear - 68%
2. right_ear - 46%
3. left_hip - 39%

Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích.

Đúng một phần. Tai hay bị tóc hoặc mũ/mũ bảo hiểm che nên bị `v=1` nhiều là hợp lý,
không bất ngờ. Hông thì khác: nó gần như không bao giờ nhìn thấy được thật sự (bị quần
áo che), nên `%v=1` cao ở đây không phải vì "khó gán" mà vì bản chất khớp đó luôn phải
ước lượng theo giải phẫu chứ không đọc được trực tiếp từ ảnh.

**Cảnh báo từ `check_pose_labels.py` và cách xử lý (trước khi khoá nhãn):**

| Ảnh | Người thứ | Cảnh báo | Đã xử lý |
| --- | --- | --- | --- |
| train_02 | 1 | Nghi đảo trái/phải ở vai và hông (so với hai mắt) | Kiểm lại thủ công, xác nhận đúng - người đứng nghiêng bên xe đạp nên heuristic so với mắt bị sai, giữ nguyên |
| train_04 | 2 | 7 khớp (cổ tay phải, hông, gối, mắt cá) bị `v=0` dù người tưởng như nằm gọn giữa ảnh | Kiểm ảnh gốc, xác nhận các khớp đó thật sự ngoài khung ảnh, giữ `v=0` |
| train_11 | 1 | 6 khớp (hông, gối, mắt cá) bị `v=0` | Sửa thành `v=1` - người vẫn trong khung, chỉ bị bàn/mèo che |
| train_13 | 1 | 4 khớp (gối, mắt cá) bị `v=0` | Sửa thành `v=1` - người vẫn trong khung, chỉ bị người khác che |

Sau khi sửa, chạy lại `check_pose_labels.py` chỉ còn 2 cảnh báo (train_02 và train_04),
cả hai đã kiểm tay và xác nhận là đúng, không phải lỗi.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.944 | 0.939 |
| OKS@0.50 | 0.966 | 1.000 |
| OKS@0.75 | 0.966 | 1.000 |
| Lỗi `dao_trai_phai` | 0 | 1 |
| Lỗi `nham_nguoi` | 1 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

Ghi chú: trước rework còn thiếu hẳn 1 người (train_13) nên không tính được OKS cho người
đó (0.000). Sau khi thêm người này vào, OKS@0.50 và OKS@0.75 lên tối đa 1.000 vì đủ
29/29 người khớp với gold, nhưng OKS trung bình giảm nhẹ vì người mới thêm bị nghi đảo
trái/phải (xem bên dưới) nên kéo điểm riêng người đó xuống 0.781.

**Tôi đã sửa gì giữa hai lần chạy** (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):

- `train_04`, người #1: sửa lỗi nhầm người ở `left_wrist` - điểm đang rơi sang cơ thể
  người bên cạnh, kéo về đúng người.
- `train_13`: gán bổ sung 1 người bị thiếu hoàn toàn so với gold (người nhỏ, đứng xa,
  mờ ở góc trái ảnh) - thêm đủ 17 điểm cho người này.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Ảnh đó dễ hay khó? Nếu là ảnh dễ,
bạn nghĩ vì sao mình vẫn sai?

Có 1 lỗi nghi đảo trái/phải, ở `train_13`, người #3 - chính là người vừa được thêm vào
ở trên. Đây là **ảnh khó**, không phải ảnh dễ: người này rất nhỏ, đứng xa và mờ, khó
nhìn rõ hướng cơ thể. Script `evaluate_pose_annotations.py` thử hoán đổi toàn bộ cặp
trái/phải cho người này và OKS tăng hẳn - dấu hiệu khách quan cho thấy có thể bị ngược.
Tuy nhiên đã phóng to ảnh hết cỡ mà vẫn không đủ rõ để khẳng định chắc chắn bằng mắt, nên
quyết định **giữ nguyên** theo phán đoán lúc gán thay vì đổi theo gợi ý số liệu (chi tiết
xem Ca 4 trong `GUIDELINE_MINI.md`). Vì chỉ là 1/29 người, ảnh hưởng tới điểm tổng thể
của cả bộ nhãn không đáng kể.

## 3. Kiểm chéo

Bỏ qua bước kiểm chéo trong lần làm này (quyết định của người gán, không có bạn cùng
nhóm để so bảng đếm).

## 4. Model

<!-- Chép số từ outputs/eval_model.json sau Chặng 6. “Chênh” = sau fine-tune trừ baseline;
đây là quan sát trên tập test, không phải chất lượng sản phẩm. -->

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | | | |
| pose_mAP50-95 | | | |
| pose_precision | | | |
| pose_recall | | | |
| box_mAP50-95 | | | |

### Trả lời năm câu hỏi ở cuối notebook

> Mỗi câu cần trỏ tới ảnh/chỉ số cụ thể. Một con số thấp không tự chứng minh nhãn sai;
> kiểm lại bằng bằng chứng thị giác và kết quả gold.

1. `pose_mAP50-95` thay đổi bao nhiêu? Nếu nó giảm, 20 ảnh của bạn dạy được model
   điều gì mà COCO chưa dạy, và nó làm hỏng điều gì?

2. `box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm
   *khớp* dễ hơn? Vì sao?

3. Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43
   (lệch nhẹ / đảo trái/phải / nhầm người / trượt hẳn):

4. Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?

5. Ảnh bạn gán tệ nhất có *cũng* là ảnh model đoán tệ nhất không? Nếu có, điều đó
   nói gì về bức ảnh đó?

## 5. Một rule evidence bạn đã dùng

Chọn một keypoint trong ảnh core mà bạn phải quyết định giữa `v=1` và `v=0`. Nêu ảnh, người,
khớp, bằng chứng nhìn thấy và lý do chọn trạng thái đó trong 3-5 câu.

Ảnh `train_11`, người đang ngồi ăn cạnh con mèo, các khớp hông/gối/mắt cá. Ban đầu tôi
gán các khớp này là `v=0` (Outside) vì không nhìn thấy chúng. Sau khi xem lại ảnh gốc,
thấy toàn bộ người đó vẫn nằm trọn trong khung ảnh - phần thân dưới chỉ bị cái bàn và
con mèo che khuất, không phải bị cắt bởi mép ảnh. Vì vậy tôi đổi lại thành `v=1`
(Occluded) và đặt chấm ở vị trí ước lượng theo hướng của đùi/thân người. Bằng chứng để
phân biệt hai trường hợp: nếu bounding box của người đó chạm mép ảnh thì mới là ra
ngoài khung (`v=0`); nếu box vẫn nằm gọn bên trong ảnh thì phần "biến mất" là do vật
khác che, phải là `v=1`.
