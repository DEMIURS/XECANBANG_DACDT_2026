# 🧭 Lý thuyết cảm biến quán tính (IMU)

GY-BMI160 trên xe gồm gia tốc kế và con quay hồi chuyển ba trục, tạo thành IMU 6 trục (6-DoF). IMU đo chuyển động của cảm biến; phần mềm phải hiệu chuẩn, đổi đơn vị và ghép các trục theo hướng lắp cảm biến trước khi dùng dữ liệu trong vòng điều khiển.

---

## 1. Gia tốc kế

Gia tốc kế MEMS đo **gia tốc riêng**. Theo quy ước véc-tơ trọng trường $\vec g$ hướng xuống, tín hiệu đo có thể viết $\vec f = \vec a - \vec g$, trong đó $\vec a$ là gia tốc chuyển động của cảm biến. Khi đứng yên, độ lớn véc-tơ đo xấp xỉ $g$; dấu từng trục phụ thuộc hướng đặt cảm biến và quy ước tọa độ.

Khi xe đứng yên hoặc chuyển động chậm, có thể ước lượng góc nghiêng từ trọng lực:

* Roll: $\phi = \operatorname{atan2}(A_y,A_z)\cdot 180/\pi$.
* Pitch: $\theta = \operatorname{atan2}(-A_x,\sqrt{A_y^2+A_z^2})\cdot 180/\pi$.

Các công thức trên giả định trục cảm biến trùng với hệ tọa độ nêu trong tài liệu. Nếu BMI160 được xoay khi lắp lên xe, cần hoán đổi trục hoặc đổi dấu theo sơ đồ lắp thực tế. Gia tốc kế không xác định được góc yaw tuyệt đối vì trọng trường không cung cấp hướng tham chiếu quanh trục thẳng đứng.

**Ưu điểm:** cung cấp tham chiếu góc dài hạn khi chỉ có trọng lực tác dụng. **Hạn chế:** rung và gia tốc tịnh tiến làm véc-tơ đo lệch khỏi trọng lực, khiến góc tức thời nhiễu.

## 2. Con quay hồi chuyển

Con quay hồi chuyển MEMS đo tốc độ góc quanh ba trục, thường được biểu diễn bằng `deg/s` hoặc `rad/s`. Để đổi tốc độ góc thành góc quanh một trục, có thể tích phân số:

$$\theta_k = \theta_{k-1} + \omega_k\Delta t$$

Đây là xấp xỉ phù hợp cho chuyển động một trục hoặc góc nhỏ quanh tư thế thẳng đứng. Với chuyển động ba chiều lớn, tốc độ Euler không bằng trực tiếp tốc độ quay thân trên từng trục; cần phép biến đổi tư thế đầy đủ.

**Ưu điểm:** phản ứng nhanh với thay đổi góc ngắn hạn. **Hạn chế:** bias và nhiễu của gyro tích lũy thành drift khi tích phân. Trừ bias nhưng không loại bỏ hết nhiễu ngẫu nhiên.

## 3. Hiệu chuẩn gyro đứng yên

Khi khởi động, giữ xe đứng yên và lấy trung bình nhiều mẫu để ước lượng bias. Mẫu trong ví dụ là số đếm thô; nếu driver đã đổi sang `deg/s`, bias và dữ liệu dùng sau đó phải cùng đơn vị.

```c
#define CALIBRATION_SAMPLES 500

float gyro_x_bias = 0.0f;
float gyro_y_bias = 0.0f;
float gyro_z_bias = 0.0f;

void IMU_Calibrate(void) {
    float sum_x = 0.0f;
    float sum_y = 0.0f;
    float sum_z = 0.0f;

    // Giữ xe bất động; đọc lần lượt đủ 500 mẫu mới.
    for (int i = 0; i < CALIBRATION_SAMPLES; ++i) {
        sum_x += Read_Raw_Gyro_X();
        sum_y += Read_Raw_Gyro_Y();
        sum_z += Read_Raw_Gyro_Z();
        Delay_ms(2);
    }

    gyro_x_bias = sum_x / (float)CALIBRATION_SAMPLES;
    gyro_y_bias = sum_y / (float)CALIBRATION_SAMPLES;
    gyro_z_bias = sum_z / (float)CALIBRATION_SAMPLES;
}
```

Sau hiệu chuẩn, tính `gyro_corrected = gyro_raw - gyro_bias`, rồi đổi sang đơn vị góc theo dải đo gyro đã cấu hình trong BMI160. Chỉ hiệu chuẩn khi cảm biến đã khởi động ổn định và xe không bị chạm hoặc rung; nếu xe di chuyển trong lúc lấy mẫu, bias ước lượng sẽ sai. Hiệu chuẩn bias gyro không hiệu chuẩn hệ số tỷ lệ, lệch trục hay offset gia tốc kế.

BMI160 hỗ trợ giao tiếp I²C và SPI. Địa chỉ I²C 7-bit thường là `0x68` hoặc `0x69`, tùy mức chân SDO/SA0; một số thư viện yêu cầu truyền địa chỉ đã dịch trái, vì vậy cần kiểm tra quy ước của API đang dùng.

## 4. Ghép dữ liệu

Bộ lọc bù dùng gyro cho đáp ứng nhanh và gia tốc kế để hiệu chỉnh drift dài hạn. Cách tính và mã C một trục được trình bày trong [Bộ lọc bù](Complementary_Filter.md).

## 5. Tài liệu tham khảo

* [Bosch Sensortec BMI160 Datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmi160-ds000.pdf)
* [Tài liệu tham khảo và datasheet của dự án](../../References/Links.md)
