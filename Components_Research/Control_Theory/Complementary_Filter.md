# 📡 Bộ Lọc Bù (Complementary Filter)

Ước lượng góc nghiêng chính xác bằng cách kết hợp ưu điểm của **Con quay hồi chuyển (Gyroscope)** và **Cảm biến gia tốc (Accelerometer)**.

---

## 1. Vấn đề của cảm biến quán tính (IMU)

* **Accelerometer (Cảm biến gia tốc):**
  - **Ưu điểm:** Đo góc tĩnh tốt dựa trên trọng trường Trái Đất, không bị trôi dạt theo thời gian dài.
  - **Nhược điểm:** Nhạy cảm với rung động cơ khí và gia tốc tịnh tiến ngắn hạn (nhiễu tần số cao).
* **Gyroscope (Con quay hồi chuyển):**
  - **Ưu điểm:** Đo tốc độ góc cực nhanh, mượt, không bị ảnh hưởng bởi rung chấn ngoại lực.
  - **Nhược điểm:** Tích lũy sai số trôi góc (Drift) theo thời gian khi tích phân vận tốc góc (nhiễu tần số thấp).

---

## 2. Nguyên lý bộ lọc bù

Bộ lọc bù đưa tín hiệu Accel qua **Bộ lọc thông thấp (Low-Pass Filter)** và tín hiệu tích phân Gyro qua **Bộ lọc thông cao (High-Pass Filter)**, sau đó cộng gộp lại:

$$\theta_{\text{filtered}}(k) = \alpha \cdot \left[ \theta_{\text{filtered}}(k-1) + \omega_{\text{gyro}} \cdot dt \right] + (1 - \alpha) \cdot \theta_{\text{acc}}$$

Trong đó:
* $\theta_{\text{acc}}$: Góc tính từ gia tốc kế, $\theta_{\text{acc}} = \text{atan2}(A_y, A_z) \cdot \frac{180}{\pi}$.
* $\omega_{\text{gyro}}$: Tốc độ góc đọc từ Gyro (đơn vị: `deg/s`).
* $dt$: Khoảng thời gian giữa 2 lần lấy mẫu (`seconds`).
* $\alpha$: Hệ số trọng số ($0 < \alpha < 1$). Thường chọn $\alpha \approx 0.95 \div 0.98$.

---

## 3. Mã nguồn C/C++ triển khai

```c
#include <math.h>

#define RAD_TO_DEG 57.295779513082320876f

typedef struct {
    float alpha;         // Hệ số lọc bù (0.95 - 0.98)
    float current_angle; // Góc sau khi lọc (deg)
} ComplementaryFilter;

void CompFilter_Init(ComplementaryFilter *filter, float alpha, float initial_angle) {
    filter->alpha = alpha;
    filter->current_angle = initial_angle;
}

float CompFilter_Update(ComplementaryFilter *filter, float ax, float ay, float az, float gx, float dt) {
    // 1. Tính góc pitch từ gia tốc kế
    float accel_angle = atan2f(ay, az) * RAD_TO_DEG;

    // 2. Tích hợp tốc độ góc Gyro và bù trừ với Accel
    filter->current_angle = filter->alpha * (filter->current_angle + gx * dt) 
                          + (1.0f - filter->alpha) * accel_angle;

    return filter->current_angle;
}