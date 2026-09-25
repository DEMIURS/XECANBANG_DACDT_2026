STM32F103C8T6 Blue Pill là bộ điều khiển trung tâm (main controller). Nó tiếp nhận dữ liệu từ BMI160 và encoder, xử lý thuật toán điều khiển, sau đó tạo tín hiệu điều khiển TB6612FNG để điều khiển hai động cơ.
Các nhiệm vụ chính của STM32F103C8T6 Blue pill:
  Đọc cảm biến BMI160
  Đọc encoder của hai động cơ
  Chạy thuật toán điều khiển cân bằng
  Xuất xung điều khiển hai động cơ
  Xử lý giao tiếp HC-05
