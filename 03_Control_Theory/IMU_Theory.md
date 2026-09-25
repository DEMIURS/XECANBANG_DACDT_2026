# Lý Thuyết Cảm Biến Đo Lường Quán Tính (IMU Theory)

Tài liệu này trình bày nguyên lý hoạt động, mô hình toán học và đặc tính đo lường của cảm biến IMU 6 trục (sử dụng module **Bosch BMI160**) áp dụng cho xe tự cân bằng 2 bánh.

---

## 1. Tổng quan cảm biến BMI160
Module GY-BMI160 tích hợp:
* **Gia tốc kế 3 trục (3-axis Accelerometer)**: Đo gia tốc trọng trường và gia tốc chuyển động.
* **Con quay hồi chuyển 3 trục (3-axis Gyroscope)**: Đo vận tốc góc quay quanh 3 trục tọa độ ($X, Y, Z$).

Giao tiếp với vi điều khiển STM32F103 qua chuẩn truyền thông **I2C** (tần số chuẩn 400 kHz Fast Mode).

---

## 2. Nguyên lý tính góc nghiêng từ Gia tốc kế (Accelerometer)

### 2.1. Mô hình toán học
Trọng lực $\vec{g}$ luôn hướng thẳng xuống tâm Trái Đất. Khi xe bị nghiêng một góc $\theta$ quanh trục trục nghiêng (thường chọn trục $Y$ làm góc Pitch):
* Thành phần gia tốc trục $X$: $a_x$
* Thành phần gia tốc trục $Z$: $a_z$

Góc nghiêng đo bởi trọng lực được tính bằng công thức:

$$\theta_{acc} = \arctan2\left(a_x, \sqrt{a_y^2 + a_z^2}\right) \cdot \frac{180^\circ}{\pi}$$

*(Hoặc dạng đơn giản nếu xe chỉ nghiêng 1 trục chính: $\theta_{acc} = \arctan2(a_x, a_z) \cdot \frac{180^\circ}{\pi}$)*

### 2.2. Đặc tính đo lường
* **Ưu điểm**: Không bị trôi (drift) theo thời gian, cho giá trị góc tĩnh (DC) chính xác.
* **Nhược điểm**: Bị ảnh hưởng rất nặng bởi rung động cơ học (vibrations) từ động cơ và quán tính khi xe tăng/giảm tốc độ. Tín hiệu chứa nhiều nhiễu tần số cao (High-frequency noise).

---

## 3. Nguyên lý tính góc nghiêng từ Con quay hồi chuyển (Gyroscope)

### 3.1. Mô hình toán học
Gyroscope đo vận tốc góc $\omega_y$ (đơn vị: $^\circ/s$ hoặc $rad/s$). Góc nghiêng được xác định bằng cách tích phân vận tốc góc theo chu kỳ lấy mẫu $\Delta t$:

$$\theta_{gyro}(t) = \theta(t - \Delta t) + (\omega_y - \text{Bias}) \cdot \Delta t$$

Trong đó:
* $\Delta t$: Chu kỳ lấy mẫu của vòng lặp điều khiển (thường là $5\text{ ms} = 0.005\text{ s}$ tương ứng $200\text{ Hz}$).
* $\text{Bias}$: Độ lệch điểm 0 (Zero-rate offset) của trục Gyroscope khi xe đứng yên.

### 3.2. Đặc tính đo lường
* **Ưu điểm**: Phản ứng cực nhanh với thay đổi góc, không bị ảnh hưởng bởi rung động cơ học tức thời. Tín hiệu mượt mà ở dải tần số cao.
* **Nhược điểm**: Hiện tượng **trôi điểm không (Gyro Drift)** do tích phân sai số offset theo thời gian:
$$\lim_{t \to \infty} \int_0^t \text{error} \, dt = \pm \infty$$

---

## 4. Tóm tắt so sánh & Hướng giải quyết

| Cảm biến | Tần số đáp ứng | Hiện tượng lỗi | Bộ lọc phù hợp |
| :--- | :--- | :--- | :--- |
| **Accelerometer** | Chậm (Low-frequency tốt) | Nhiễu cơ học, rung động động cơ | Lọc thông thấp (Low-pass Filter) |
| **Gyroscope** | Nhanh (High-frequency tốt) | Trôi tích phân (Drift) | Lọc thông cao (High-pass Filter) |

**Kết luận**: Cần dung hợp dữ liệu (Sensor Fusion) bằng **Bộ lọc bù (Complementary Filter)** hoặc **Bộ lọc Kalman** để kết hợp ưu điểm của cả hai loại cảm biến.
