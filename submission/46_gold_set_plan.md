# Đề xuất gold set theo camera — tình huống giả lập

Đầu bài: 50.000 frame từ bốn camera SVM, ngân sách chọn 200 frame để review/gold. Đây là tình huống trên slide, không phải 50.000 frame có trong repo. Phân bổ đúng 200 ở 45_sampling_plan.csv cho bốn camera, mỗi camera có normal và hard slice. “Gold set” ở đây là kế hoạch tạo reference sau kiểm chứng, không phải teaching reference ADASIND hoặc nhãn bạn vừa vẽ. Nếu cần, dùng notebooks/day11-svm360-colab.ipynb để thử tổng phân bổ; notebook không làm thay phần lý do.

| camera_id | Hard case cần chọn | Vì sao dễ sai | Annotation space / calibration cần giữ | Cách review trước khi gọi là gold |
| :--- | :--- | :--- | :--- | :--- |
| **front** | Ngược nắng chói sáng, mật độ phương tiện đông đúc ở khoảng cách xa và rìa hai bên đầu xe; vạch kẻ đường bị che khuất một phần. | Độ phân giải giảm mạnh ở vùng xa tâm kính mắt cá; lóa sáng làm mất biên vật thể và làm đứt đoạn polyline vạch kẻ. | Giữ nguyên không gian ảnh gốc (raw fisheye image space); loại trừ vùng cản trước (`ego_body`) và viền đen ngoài `lens circle`; lưu kèm thông số intrinsic/extrinsic của cam trước. | 2 annotator gán nhãn độc lập, đối chiếu độ lệch IoU/khoảng cách ở rìa cong; Lead QA phân xử các vật thể bị che khuất >50% trước khi khóa nhãn. |
| **rear** | Bãi đỗ xe thiếu sáng hoặc tầng hầm, bóng râm gầm xe đỗ sát phía sau, lóa đèn phanh/đèn hậu và vạch sơn chia ô đỗ bị mờ/mòn. | Góc đặt cam thấp sát mặt đường gây biến dạng phối cảnh lớn; bóng tối dưới gầm xe dễ bị nhầm với vùng trống (`free_space`) hoặc làm lệch chân hộp giới hạn (bounding box). | Gán nhãn trên ảnh raw fisheye; cố định mặt nạ (mask) che phần cản sau/biển số thuộc `ego_body`; giữ nguyên bộ tham số góc chúc (pitch angle) của cam sau. | Kiểm tra chéo (double-blind review) ranh giới dừng của `free_space` sát chướng ngại vật và mép cản sau; phóng đại 200% vùng bóng râm để xác nhận điểm chạm đất. |
| **left** | Phương tiện (xe máy, ô tô) di chuyển hoặc đỗ sát sườn trái; vật thể nằm ngay vùng rìa cực đại của ống kính (vùng giao thoa với cam trước/sau). | Biến dạng cong hình học (radial distortion) mạnh nhất ở hai mép trái/phải làm bóp méo hình dáng xe và uốn cong vạch đỗ vuông góc. | Giữ không gian raw fisheye đồng bộ với cam trước/sau; cố định ranh giới sườn xe/gương trái (`ego_body`) và giữ nguyên thông số calibration góc lệch sườn trái. | Đối chiếu đồng thuận giữa 2 người gán nhãn trên các đối tượng bị kéo giãn ở mép kính; kiểm tra tính nhất quán với frame cùng thời điểm ở cam trước/sau. |
| **right** | Vật cản hỗn hợp sát lề đường bên phải (vỉa hè cao, cột trụ, người đi bộ, xe hai bánh áp sát) và bóng râm thân xe đè lên vạch đỗ. | Khó phân tách ranh giới giữa mặt đường trống (`free_space`) với gờ vỉa hè thấp hoặc vật cản nhỏ bị biến dạng cong ở góc nhìn nghiêng. | Gán nhãn trên ảnh raw fisheye; tách biệt rõ vùng thân xe/gương phải (`ego_body`); bảo toàn thông số hiệu chuẩn góc lắp đặt cam gương phải. | 2 người kiểm tra độc lập, tập trung nghiệm thu đường biên giữa `free_space` và vỉa hè/chướng ngại vật; chỉ duyệt vào Gold Set khi đạt đồng thuận IoU >= 0.85. |

- **Khi nào cần refresh gold set (đổi camera, calibration hoặc rule):** Cần làm mới (refresh) hoặc gán nhãn lại bộ Gold Set ngay khi: (1) Thay đổi phần cứng cảm biến hoặc ống kính (đổi tiêu cự, góc mở FOV, độ phân giải); (2) Thay đổi vị trí lắp đặt hoặc hiệu chuẩn lại camera (re-calibration làm dịch chuyển vùng `ego_body`, tâm quang học hoặc độ méo rìa ảnh); (3) Cập nhật quy tắc gán nhãn (ontology/annotation guideline) như thêm lớp nhãn mới, thay đổi ngưỡng che khuất (occlusion), hoặc đổi quy định cắt biên tại vòng kính (`lens circle`).
- **Một ca seam/cross-camera cần policy và evidence trước khi ghép hai box:** Khi một phương tiện dài (như xe buýt, xe tải hoặc ô tô đang vượt) nằm vắt ngang qua vùng chồng lấn (seam/overlap zone) giữa camera trước (`front`) và camera sườn (`left`/`right`), không được tự ý suy đoán phần khuất hoặc ghép gộp cảm tính. Cần có **Policy (Quy tắc)** rõ ràng: ở chế độ gán nhãn từng camera độc lập, chỉ vẽ phần vật thể thực sự nhìn thấy bên trong vòng kính (`lens circle`) của từng camera kèm thuộc tính bị cắt biên (`truncated`), chỉ liên kết ID (`track_id` / cross-camera ID) khi có quy định gộp đa góc nhìn; và cần **Evidence (Bằng chứng)** gồm: dấu thời gian (timestamp) đồng bộ chính xác giữa 2 frame, đặc điểm nhận dạng trùng khớp ở vùng giao thoa, và kết quả chiếu hình học 3D/BEV xác nhận hai phần thuộc cùng một vật thể vật lý.
- **Vì sao peer agreement hoặc quality report trên ảnh một camera chưa chứng minh gold set đúng cho cả bốn camera:** Mỗi camera trong hệ thống SVM360 có vị trí lắp đặt, góc chúc (pitch/roll), phần thân xe lọt vào ảnh (`ego_body`) và đặc thù cảnh quan hoàn toàn khác nhau (ví dụ: cam trước/sau nhìn dọc theo chiều chuyển động với tầm nhìn xa, trong khi cam trái/phải gắn dưới gương nhìn ngang sát mặt đường với độ biến dạng gần cực lớn). Do đó, việc đạt chỉ số đồng thuận (peer agreement) cao trên 1 camera chỉ chứng minh người gán nhãn nắm vững quy tắc ở góc nhìn đó, chứ không phát hiện được các lỗi đặc thù ở 3 góc còn lại cũng như sự thiếu nhất quán khi một vật thể đi qua vùng giáp ranh (seam) giữa các camera.



# Đề xuất gold set theo camera — tình huống giả lập

**Đầu bài:** 50.000 frame từ bốn camera SVM, ngân sách chọn 200 frame để review/gold. Đây là tình huống trên slide,
**không phải** 50.000 frame có trong repo. Phân bổ đúng 200 ở `45_sampling_plan.csv` cho bốn camera, mỗi camera có
normal và hard slice. “Gold set” ở đây là **kế hoạch tạo** reference sau kiểm chứng, không phải teaching reference
ADASIND hoặc nhãn bạn vừa vẽ. Nếu cần, dùng `notebooks/day11-svm360-colab.ipynb` để thử tổng phân bổ; notebook
không làm thay phần lý do.

| camera_id | Hard case cần chọn | Vì sao dễ sai | Annotation space / calibration cần giữ | Cách review trước khi gọi là gold |
|---|---|---|---|---|
| front | TODO | TODO | TODO | TODO |
| rear | TODO | TODO | TODO | TODO |
| left | TODO | TODO | TODO | TODO |
| right | TODO | TODO | TODO | TODO |

- Khi nào cần refresh gold set (đổi camera, calibration hoặc rule): TODO
- Một ca seam/cross-camera cần policy và evidence trước khi ghép hai box: TODO
- Vì sao peer agreement hoặc quality report trên ảnh một camera chưa chứng minh gold set đúng cho cả bốn camera: TODO


