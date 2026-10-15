# 🧭 Lý Thuyết Cảm Biến Quán Tính (IMU Theory)

Tài liệu chi tiết về nguyên lý hoạt động, mô hình toán học và các đặc tính vật lý của cảm biến đo lường quán tính (Inertial Measurement Unit - IMU), trọng tâm là cảm biến 6 bậc tự do (6-DOF: Accelerometer + Gyroscope như MPU6050/ICM-20600).

---

## 1. Cảm Biến Gia Tốc (Accelerometer)

### 1.1. Nguyên lý đo
* Cảm biến gia tốc MEMS đo lực quán tính tác dụng lên một khối lượng chuẩn (proof mass) bị giữ bởi các vi lò xo điện dung.
* Đại lượng đo được là **Gia tốc riêng (Specific Force)**, bao gồm tổng của gia tốc chuyển động tuyến tính và phản lực trọng trường:
  $$\vec{a}_{meas} = \vec{a}_{motion} - \vec{g}$$
* Ở trạng thái đứng yên hoặc chuyển động đều ($\vec{a}_{motion} = 0$), cảm biến chỉ đo duy nhất thành phần trọng trường Trái Đất:
  $$\vec{a}_{meas} = -\vec{g}$$

### 1.2. Công thức tính góc nghiêng (Tilt Angles)
Từ các thành phần gia tốc trục $A_x, A_y, A_z$, ta tính được các góc nghiêng (Pitch $\theta$ và Roll $\phi$) theo hệ tọa độ Euler:

* **Góc Roll ($\phi$ - quay quanh trục X):**
  $$\phi = \arctan2\left(A_y, A_z\right) \cdot \frac{180}{\pi}$$

* **Góc Pitch ($\theta$ - quay quanh trục Y):**
  $$\theta = \arctan2\left(-A_x, \sqrt{A_y^2 + A_z^2}\right) \cdot \frac{180}{\pi}$$

> ⚠️ **Lưu ý:** Accelerometer **không thể đo được góc Yaw (quay quanh trục Z)** vì trục quay trùng với phương của véc-tơ trọng trường.

### 1.3. Ưu & Nhược điểm
* **Ưu điểm:** Đo góc tĩnh tuyệt đối chính xác; sai số không bị tích lũy theo thời gian (Zero Drift).
* **Nhược điểm:** Cực kỳ nhạy cảm với rung động cơ khí tần số cao và gia tốc chuyển động tức thời của xe/robot.

---

## 2. Con Quay Hồi Chuyển (Gyroscope)

### 2.1. Nguyên lý đo
* Hoạt động dựa trên hiệu ứng **Coriolis**: Khi một phần tử vi cơ chuyển động dao động và có chuyển động quay của khung, một lực Coriolis vuông góc sẽ sinh ra, làm biến dạng tụ điện đo.
* Đại lượng đo trực tiếp là **Tốc độ góc (Angular Velocity)** $\omega$ trên từng trục: $\omega_x, \omega_y, \omega_z$ (đơn vị: $\text{deg/s}$ hoặc $\text{rad/s}$).

### 2.2. Tính toán góc nghiêng qua Tích phân
Góc nghiêng được tính bằng cách tích phân số tốc độ góc theo thời gian:

$$\theta(k) = \theta(k-1) + \omega(k) \cdot \Delta t$$

### 2.3. Ưu & Nhược điểm
* **Ưu điểm:** Tốc độ đáp ứng cực cao, mượt mà, không bị ảnh hưởng bởi rung chấn cơ học ngắn hạn.
* **Nhược điểm:** 
  - **Gyro Bias / Offset:** Khi đứng yên, Gyro vẫn xuất hiện giá trị đo khác 0 ($\omega_{offset} \neq 0$).
  - **Drift (Trôi góc):** Việc tích phân một giá trị offset cố định qua thời gian làm sai số góc tăng tuyến tính theo thời gian:
    $$e_{\theta}(t) = \int_{0}^{t} \omega_{offset} \, d\tau = \omega_{offset} \cdot t$$

---

## 3. Hiệu Chuẩn Tĩnh (Zero-rate Calibration)

Trước khi đưa vào thuật toán điều khiển, bắt buộc phải loại bỏ Gyro Offset bằng quy trình lấy mẫu tĩnh khi vừa cấp nguồn:

```c
#define CALIBRATION_SAMPLES 500

float gyro_x_offset = 0.0f;
float gyro_y_offset = 0.0f;
float gyro_z_offset = 0.0f;

void IMU_Calibrate(void) {
    float sum_x = 0, sum_y = 0, sum_z = 0;
    
    // Đảm bảo robot được giữ cố định hoàn toàn trong giai đoạn này
    for (int i = 0; i < CALIBRATION_SAMPLES; i++) {
        sum_x += Read_Raw_Gyro_X();
        sum_y += Read_Raw_Gyro_Y();
        sum_z += Read_Raw_Gyro_Z();
        Delay_ms(2);
    }
    
    gyro_x_offset = sum_x / CALIBRATION_SAMPLES;
    gyro_y_offset = sum_y / CALIBRATION_SAMPLES;
    gyro_z_offset = sum_z / CALIBRATION_SAMPLES;
}