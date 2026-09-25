Nhiệm vụ của TB6612FNG là nhận tín hiệu điều khiển mức logic từ STM32 và kết hợp nguồn áp từ các cell pin để đóng/ngắt dòng công suất cấp cho động cơ.
TB6612FNG là dual DC motor driver, tức là có 2 kênh điều khiển động cơ độc lập:
Kênh A → Motor trái
Kênh B → Motor phải
Mỗi kênh có thể:
  quay thuận;
  quay nghịch;
  dừng;
  phanh;
  điều chỉnh tốc độ bằng PWM.

Điểm quan trọng là:
STM32 không cấp trực tiếp dòng điện cho motor.
STM32 chỉ xuất tín hiệu điều khiển:PWM, IN1, IN2
TB6612FNG chịu trách nhiệm đóng/ngắt đường công suất cho motor.
