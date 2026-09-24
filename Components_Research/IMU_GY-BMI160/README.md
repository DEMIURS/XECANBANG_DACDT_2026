GY-BMI160 là cảm biến quán tính 6 bậc tự do, tích hợp cảm biến gia tốc ba trục và cảm biến con quay hồi chuyển ba trục. 
Trong hệ thống xe hai bánh tự cân bằng, module có nhiệm vụ thu thập dữ liệu về gia tốc và tốc độ góc của thân xe.
Dữ liệu này được truyền đến STM32F103C8T6 thông qua giao tiếp I²C để thực hiện hiệu chỉnh, lọc và ước lượng góc nghiêng của xe.
Góc nghiêng và tốc độ góc là các thông tin phản hồi quan trọng để thuật toán điều khiển xác định mức tác động cần thiết lên hai động cơ nhằm duy trì trạng thái cân bằng.
Nếu STM32 là bộ não, TB6612FNG là tầng công suất, thì GY-BMI160 chính là cảm biến giúp STM32 "biết xe đang nghiêng và đang chuyển động như thế nào".
3-axis accelerometer — cảm biến gia tốc 3 trục.
3-axis gyroscope — cảm biến tốc độ góc 3 trục.

Do đó có tổng cộng: 3+3=6 DOF

hay còn gọi là 6-axis IMU.
Trong xe cân bằng, hai nhóm dữ liệu này có vai trò khác nhau:
  Cảm biến accelerometer: Xác định hướng trọng lực → ước lượng góc nghiêng
  Cảm biến gyroscope: Đo tốc độ quay → biết xe đang nghiêng nhanh/chậm

BMI160 thường có địa chỉ I²C: 0x68 hoặc 0x69 tùy trạng thái của chân địa chỉ trên module.
