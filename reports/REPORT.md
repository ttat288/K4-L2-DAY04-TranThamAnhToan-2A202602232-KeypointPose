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
| OKS trung bình | 0.944 | 0.944 |
| OKS@0.50 | 0.966 | 1.000 |
| OKS@0.75 | 0.966 | 1.000 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 1 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

Ghi chú: rework của tôi có 2 bước. Trước rework còn thiếu hẳn 1 người (train_13) nên
không tính được OKS cho người đó (0.000), và có 1 lỗi nhầm người ở `train_04`. Bước 1
tôi thêm người bị thiếu vào - lúc này hết lỗi thiếu người nhưng người mới thêm lại bị
đảo trái/phải (OKS riêng người đó chỉ 0.781-0.819 tuỳ lần chỉnh). Bước 2 tôi đổi lại
đúng toàn bộ 8 cặp trái/phải cho người đó (mắt, tai, vai, khuỷu tay, cổ tay, hông, gối,
mắt cá - không đổi lẻ tẻ vài điểm vì dễ tạo ra một skeleton nửa đúng nửa sai còn tệ hơn
ban đầu). Sau bước 2: OKS trung bình quay lại 0.944, OKS@0.50/0.75 đạt tối đa 1.000, hết
sạch cả 3 loại lỗi ưu tiên cao (`dao_trai_phai`, `nham_nguoi`, thiếu/thừa người).

**Tôi đã sửa gì giữa hai lần chạy** (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):

- `train_04`, người #1: sửa lỗi nhầm người ở `left_wrist` - điểm đang rơi sang cơ thể
  người bên cạnh, kéo về đúng người.
- `train_13`: gán bổ sung 1 người bị thiếu hoàn toàn so với gold (người nhỏ, đứng xa,
  mờ ở góc trái ảnh) - thêm đủ 17 điểm cho người này.
- `train_13`, người mới thêm ở trên: đổi lại toàn bộ 8 cặp trái/phải (mắt, tai, vai,
  khuỷu tay, cổ tay, hông, gối, mắt cá) sau khi phát hiện bị đảo - xem chi tiết ngay
  dưới đây.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Ảnh đó dễ hay khó? Nếu là ảnh dễ,
bạn nghĩ vì sao mình vẫn sai?

Có 1 lỗi đảo trái/phải, ở `train_13`, người mới thêm (rất nhỏ, đứng xa, mờ ở góc trái
ảnh, khung bao chỉ rộng ~36px). Đây là **ảnh khó**, không phải ảnh dễ - khó đến mức phóng
to hết cỡ vẫn không đủ rõ để khẳng định hướng cơ thể chỉ bằng mắt. `evaluate_pose_
annotations.py` báo hoán đổi toàn bộ cặp trái/phải làm OKS tăng hẳn (0.819 -> 0.943),
nên tôi đổi lại đúng cả 8 cặp cùng lúc (không đổi riêng lẻ từng cặp, vì đã thử đổi mỗi
vai trước đó và tạo ra một skeleton nửa đổi nửa chưa, còn bị flag nặng hơn). Sau khi đổi
đủ cả 8 cặp, lỗi biến mất hoàn toàn (0 lỗi `dao_trai_phai` trong `outputs/eval_vs_gold.json`).
Chi tiết quá trình ghi ở Ca 4, `GUIDELINE_MINI.md`.

## 3. Kiểm chéo

Bỏ qua bước kiểm chéo trong lần làm này (quyết định của người gán, không có bạn cùng
nhóm để so bảng đếm).

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

> Mỗi câu cần trỏ tới ảnh/chỉ số cụ thể. Một con số thấp không tự chứng minh nhãn sai;
> kiểm lại bằng bằng chứng thị giác và kết quả gold.

1. `pose_mAP50-95` tăng nhẹ +0.0055 (từ 0.6853 lên 0.6908), không giảm. Với chỉ 20 ảnh
   fine-tune thì đây là một cải thiện rất nhỏ, gần như trong biên độ nhiễu - không đủ để
   nói model học được điều gì mới đáng kể so với COCO gốc, nhưng ít nhất không làm hỏng
   khả năng đoán pose đã có (`pose_recall` giữ nguyên 0.8462, không có dấu hiệu quên).

2. `box_mAP50-95` giảm nhẹ (-0.0078) trong khi `pose_mAP50-95` tăng nhẹ (+0.0055) - hai
   chỉ số lệch nhau rất ít (dưới 0.01), nên chưa đủ bằng chứng để kết luận model tìm
   *người* dễ hơn hay tìm *khớp* dễ hơn sau fine-tune 20 ảnh. Có thể coi cả hai gần như
   không đổi.

3. Ảnh `test_02`: model đặt một box "person 0.31" (tự tin thấp) lên đúng vị trí một **con
   chim** đậu trên cột ăng-ten, kèm cả một bộ khung xương lên con chim đó. Đây không khớp
   hẳn 4 loại lỗi của slide 43 vì không có người thật ở đó để so `lệch nhẹ/đảo trái-phải/
   nhầm người/trượt hẳn` - gần nhất là **trượt hẳn**: toàn bộ khớp đặt sai hoàn toàn, model
   nhận nhầm vật không phải người thành người.

4. Ảnh `train_15`, người bên trái (đang cúi người dựa vào xe máy ở cây xăng), OKS model
   vs nhãn của tôi chỉ **0.64** - thấp nhất trong 20 ảnh. Xem lại `outputs/vis_train/
   train_15.jpg`: khung xương tôi gán không bắt chéo ở thân, trái/phải đúng theo hướng cơ
   thể, các khớp đặt hợp lý ở tư thế cúi người. Vì vậy tôi cho rằng **model sai chứ không
   phải nhãn tôi sai** - tư thế cúi người và phần chân bị xe máy che khuất nhiều khiến
   model (chỉ fine-tune trên 20 ảnh) khó đoán đúng vị trí gối/mắt cá.

5. Không hoàn toàn. Ảnh có OKS thấp nhất tuyệt đối là `train_15` (0.64), nhưng ảnh này
   trước đó không nằm trong danh sách tôi thấy khó gán hay bị cảnh báo. Ngược lại,
   `train_13` - ảnh tôi từng đánh giá là khó nhất khi gán (phải thêm người bị thiếu, và
   còn 1 ca nghi đảo trái/phải chưa chắc chắn, xem Ca 4 ở `GUIDELINE_MINI.md`) - cũng nằm
   trong nhóm OKS thấp nhất giữa model và nhãn tôi (0.675 và 0.682 cho 2 trong 3 người).
   Nên có sự trùng khớp **một phần**: ảnh khó với người gán cũng có xu hướng khó với
   model, nhưng không phải lúc nào ảnh khó nhất với người cũng là ảnh model tệ nhất.

**Ghi chú thêm:** bảng so OKS cũng cho thấy 2 ảnh model đếm **sai số người** so với nhãn
của tôi: `train_10` (model thấy 2 người, tôi gán 1) và `train_03` (model thấy 4, tôi gán
2) - đây là các trường hợp cần xem lại bằng mắt nhưng không nhất thiết là nhãn tôi sai,
vì model 20-ảnh dễ bị dương tính giả (nhận nhầm vật/bóng thành người) hơn là bỏ sót người
thật.
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
