# Mini guideline - mã cặp: 21902232  |  người gán: TRẦN THẨM ANH TOÀN (MSSV 2A202602232)  |  ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn (phải điền)

| Tình huống | Luật nhóm bạn chọn | Vì sao |
| --- | --- | --- |
| Hông của người mặc quần áo dài | Vẫn đặt điểm, coi là `v=1` (che), ước lượng theo eo/quần | Hông gần như không bao giờ thấy được, đây là điểm giải phẫu ước lượng chứ không phải điểm nhìn thấy thật |
| Tai bị tóc hoặc mũ bảo hiểm che một phần | Vẫn đặt điểm ở đúng vị trí tai, gắn `v=1` | Tai còn trong khung, chỉ là bị vật khác che, không phải mất hẳn |
| Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên) | Chân/mắt cá không nằm trong ảnh thì để `v=0`, không đặt chấm | Đây là trường hợp thật của `train_04`: cổ tay phải, hông, gối, mắt cá của một người nằm ngoài khung ảnh thật, không phải bị vật che, nên đúng là Outside |
| Cổ tay nằm sau tay lái / sau thân mình | Vẫn đặt điểm ở vị trí ước lượng, gắn `v=1` | Khớp vẫn còn trong ảnh, chỉ bị xe/thân người khác che khuất — đây là lỗi đã gặp ở `train_11` và `train_13`, ban đầu gán nhầm thành `v=0`, đã sửa lại thành `v=1` |
| Hai người chồng lên nhau | Gán xong hẳn một người rồi mới sang người kia, không làm xen kẽ | Làm xen kẽ dễ gắn nhầm điểm của người này sang người kia |
| Người nhỏ đến mức nào thì không gán nữa | Không gặp trường hợp này trong 20 ảnh core | Bộ ảnh đã được chọn để mọi người đều đủ lớn để gán |

Với mỗi luật, chèn **một ảnh mẫu** (screenshot từ CVAT) thay vì chỉ viết một câu.
Slide 12 nói rõ: khớp không có bề mặt nhìn thấy được thì phải có ảnh mẫu, không phải
một câu văn chung chung.

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_02`, người thứ `1`, khớp `left_shoulder/right_shoulder, left_hip/right_hip`

- Mơ hồ ở chỗ nào: script `check_pose_labels.py` báo nghi đảo trái/phải vì vai và hông
  nằm ngược chiều so với hai mắt.
- Bạn quyết thế nào: giữ nguyên, không đổi trái/phải.
- Vì sao: người trong ảnh đang đứng nghiêng người bên xe đạp, không quay thẳng mặt vào
  camera, nên vị trí mắt không phản ánh đúng hướng vai/hông. Đã tự kiểm lại theo cách
  "tưởng tượng đứng vào vị trí người đó" và xác nhận trái/phải đang đúng.
- Nếu người khác quyết ngược lại thì model học sai cái gì: model sẽ học nhầm quy tắc
  trái/phải theo hướng mặt thay vì theo hướng cơ thể, dẫn tới đoán sai ở mọi ảnh người
  đứng nghiêng hoặc quay lưng.

### Ca 2 - ảnh `train_04`, người thứ `2`, khớp `right_wrist, left_hip, right_hip, left_knee, right_knee, left_ankle, right_ankle`

- Mơ hồ ở chỗ nào: các khớp này không nằm trong khung ảnh, script cảnh báo vì tưởng
  người đó nằm gọn giữa ảnh nên nghi đang dùng nhầm Outside thay vì Occluded.
- Bạn quyết thế nào: giữ `v=0` (Outside), không đặt chấm.
- Vì sao: kiểm lại ảnh gốc thấy các khớp đó thật sự nằm ngoài mép ảnh, không phải bị
  vật khác che, nên Outside là đúng.
- Nếu người khác quyết ngược lại thì model học sai cái gì: nếu gán nhầm thành `v=1` và
  đặt chấm ở một vị trí đoán mò ngoài khung, model sẽ học một toạ độ ảo không có thật,
  làm hỏng việc học các khớp thân dưới.

### Ca 3 - ảnh `train_11` và `train_13`, người thứ `1`, khớp `hông, gối, mắt cá`

- Mơ hồ ở chỗ nào: ban đầu các khớp này bị gán `v=0` (Outside) vì bị bàn/người khác che
  khuất, dễ nhầm là "không thấy nên không tồn tại".
- Bạn quyết thế nào: sửa lại thành `v=1` (Occluded), đặt chấm ở vị trí ước lượng.
- Vì sao: người đó vẫn nằm trọn trong khung ảnh, chỉ là vật khác đứng trước che mất
  phần thân dưới - đây là bị che, không phải ra khỏi khung.
- Nếu người khác quyết ngược lại thì model học sai cái gì: model sẽ học rằng khớp biến
  mất hoàn toàn khi bị che, thay vì học đoán vị trí ước lượng khi bị che một phần -
  làm giảm khả năng suy luận khi gặp người bị vật cản trong ảnh thật.

### Ca 4 - ảnh `train_13`, người mới thêm vào (trước đó bị thiếu hoàn toàn), toàn bộ cặp trái/phải

- Mơ hồ ở chỗ nào: đây là một người rất nhỏ, đứng xa và mờ ở góc trái ảnh (khung bao chỉ
  rộng ~36px). Khi chấm với gold, `evaluate_pose_annotations.py` báo nghi đảo trái/phải.
  Phóng to ảnh hết cỡ vẫn không đủ rõ để khẳng định bằng mắt hướng người đó đang quay về
  đâu, nên ban đầu chỉ sửa được một phần (đổi vai nhưng quên đổi hông) - kết quả là một
  skeleton nửa đổi nửa chưa, vẫn bị báo lỗi và OKS gần như không đổi.
- Bạn quyết thế nào: đổi lại **toàn bộ 8 cặp trái/phải cùng lúc** (mắt, tai, vai, khuỷu
  tay, cổ tay, hông, gối, mắt cá) thay vì đổi lẻ tẻ từng cặp một.
- Vì sao: bài học rút ra từ lần sửa nửa vời trước đó là **không được đổi rời rạc từng
  cặp** - đảo trái/phải là một quyết định áp dụng cho toàn bộ cơ thể, đổi một phần tạo ra
  một skeleton còn sai hơn bản gốc. Sau khi đổi đủ cả 8 cặp, `evaluate_pose_annotations.py`
  xác nhận hết lỗi (OKS riêng người này từ 0.819 lên 0.943, không còn bị gắn cờ
  `dao_trai_phai`).
- Nếu người khác quyết ngược lại (chỉ đổi một vài cặp thay vì cả 8): model sẽ học một
  skeleton không tồn tại trong thực tế (nửa người quay hướng này, nửa kia quay hướng
  khác) - còn tệ hơn cả việc giữ nguyên lỗi ban đầu.

## 4. Sau khi so visibility report với bạn cùng nhóm

Bỏ qua bước này theo quyết định của người gán - không có bạn cùng nhóm để so bảng đếm
trong lần làm này.
