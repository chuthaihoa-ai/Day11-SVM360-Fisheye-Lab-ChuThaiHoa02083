
## Sensor context

- **Rig:** Hệ thống camera mắt cá góc siêu rộng (SVM360 - Surround View Monitoring) gắn trên phương tiện di chuyển trong môi trường giao thông đường phố (ADASIND) ở các hướng quanh xe (`front`, `rear`, `left`, `right`), góc nhìn hướng ra mặt đường và hơi chúc xuống để quan sát không gian lân cận quanh thân xe.
- **ego_body:** Nhìn thấy ở phần rìa sát mép vòng kính (thường ở mép dưới hoặc mép bên của vùng ảnh sáng), bao gồm một phần thân xe gắn camera như viền cản xe (bumper), mép nắp capo, sườn xe hoặc cụm gương/tay lái bị uốn cong theo hiệu ứng thấu kính mắt cá.
- **Vòng kính (lens circle):** Nằm ở vùng trung tâm khung hình dọc (kích thước 1080x1920, kéo dài theo trục dọc từ khoảng y ≈ 136 đến y ≈ 1677), hai bên chạm sát mép trái/phải và để lại 2 vùng viền đen lớn (`lens_border`) ở phía trên và phía dưới khung hình; vùng ảnh bên trong vòng kính chiếm khoảng 65% – 75% tổng diện tích khung hình.

# Sensor context

- TODO — Rig: mô tả ngắn xe/camera gắn ở đâu theo hiểu biết của bạn từ ảnh (ADASIND không kèm tài liệu rig chi
  tiết, ghi theo quan sát).
- TODO — `ego_body` nhìn thấy ở đâu trong frame (góc capo, gương, tay lái...).
- TODO — Vòng kính (lens circle) nằm ở vị trí nào trong ảnh, chiếm khoảng bao nhiêu phần khung hình.
