# Visibility report

- Thư mục nhãn: `dataset\labels\train`
- 20 ảnh, 29 skeleton, trung bình 16.17 khớp có v > 0 mỗi người
- Tổng: v=2 323 | v=1 146 | v=0 24

| # | Khớp | v=2 | v=1 | v=0 | %v=1 |
| ---: | --- | ---: | ---: | ---: | ---: |
| 0 | nose | 22 | 7 | 0 | 24% |
| 1 | left_eye | 18 | 11 | 0 | 38% |
| 2 | right_eye | 20 | 9 | 0 | 31% |
| 3 | left_ear | 10 | 19 | 0 | 66% |
| 4 | right_ear | 15 | 14 | 0 | 48% |
| 5 | left_shoulder | 25 | 4 | 0 | 14% |
| 6 | right_shoulder | 25 | 4 | 0 | 14% |
| 7 | left_elbow | 25 | 4 | 0 | 14% |
| 8 | right_elbow | 25 | 4 | 0 | 14% |
| 9 | left_wrist | 20 | 9 | 0 | 31% |
| 10 | right_wrist | 20 | 8 | 1 | 28% |
| 11 | left_hip | 17 | 11 | 1 | 38% |
| 12 | right_hip | 21 | 7 | 1 | 24% |
| 13 | left_knee | 18 | 8 | 3 | 28% |
| 14 | right_knee | 16 | 11 | 2 | 38% |
| 15 | left_ankle | 14 | 7 | 8 | 24% |
| 16 | right_ankle | 12 | 9 | 8 | 31% |

## Đọc bảng này thế nào

1. Khớp nào có **%v=1 cao**: khớp hay bị che. Cổ tay và hông thường là hai vị trí cần xem lại guideline trước khi kết luận.
2. Khớp nào có **v=0 cao bất thường**: mọi người đang dùng Outside ở chỗ đáng lẽ là Occluded. Đó là lỗi số 3 của slide 46, và nó xoá thẳng khớp đó khỏi bảng điểm OKS.
3. Khi so hai người: **lệch lớn = bất đồng về guideline**, không phải về bức ảnh. Sửa guideline trước, sửa nhãn sau.
