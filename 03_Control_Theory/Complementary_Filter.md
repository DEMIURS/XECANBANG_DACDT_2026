# Bộ Lọc Bù Dung Hợp Dữ Liệu Cảm Biến (Complementary Filter)

Tài liệu này trình bày nguyên lý toán học, phương trình sai phân rời rạc và cách hiện thực giải thuật Bộ lọc bù (Complementary Filter) trên vi điều khiển STM32F103 nhằm ước lượng chính xác góc nghiêng (Pitch) của xe tự cân bằng 2 bánh từ cảm biến IMU GY-BMI160.

---

## 1. Đặt vấn đề & Khái niệm

Để xe tự cân bằng hoạt động ổn định, hệ thống cần biết chính xác góc nghiêng tức thời:
* **Gia tốc kế (Accelerometer)**: Cung cấp góc nghiêng tuyệt đối dựa vào trọng lực Trái Đất, không bị trôi theo thời gian nhưng cực kỳ nhạy cảm với các rung chấn cơ học và lực quán tính khi động cơ đổi chiều quay (nhiễu tần số cao).
* **Con quay hồi chuyển (Gyroscope)**: Phản ứng tức thời với vận tốc góc, không bị ảnh hưởng bởi rung lắc cơ học tần số cao nhưng lại bị trôi điểm không (drift) khi tích phân theo thời gian (nhiễu tần số thấp).

**Bộ lọc bù** giải quyết bài toán trên bằng cách kết hợp:
* Một bộ **Lọc thông thấp (Low-Pass Filter - LPF)** cho dữ liệu từ Gia tốc kế.
* Một bộ **Lọc thông cao (High-Pass Filter - HPF)** cho dữ liệu từ Con quay hồi chuyển.

```text
Gyro Rate (ω) ----> [ High-pass Filter (HPF) ] ----\
                                                     (+) ---> Góc ước lượng (θ)
Accel Angle (θ_acc) -> [ Low-pass Filter (LPF) ] ---/
Điều kiện cốt lõi để tín hiệu không bị méo pha:
$$H_{LPF}(s) + H_{HPF}(s) = 1$$

---

## 2. Mô hình toán học rời rạc hóa

Công thức truy hồi được tính toán sau mỗi chu kỳ lấy mẫu $\Delta t$:

$$\theta_k = \alpha \cdot (\theta_{k-1} + \omega_y \cdot \Delta t) + (1 - \alpha) \cdot \theta_{acc}$$

Trong đó:
* $\theta_k$: Góc nghiêng ước lượng tại chu kỳ lấy mẫu hiện tại (đơn vị: độ - $^\circ$).
* $\theta_{k-1}$: Góc nghiêng ước lượng ở chu kỳ trước đó.
* $\omega_y$: Tốc độ góc đo quanh trục nghiêng từ Gyroscope (đơn vị: $^\circ/s$, đã trừ offset điểm tĩnh).
* $\Delta t$: Chu kỳ lấy mẫu của vòng quét cảm biến (ví dụ: $\Delta t = 0.005\text{ s}$ ứng với tần số ngắt $200\text{ Hz}$).
* $\theta_{acc}$: Góc nghiêng tính tức thời từ Accelerometer:
  $$\theta_{acc} = \arctan2(a_x, a_z) \cdot \frac{180}{\pi}$$
* $\alpha$: Hệ số trọng số của bộ lọc bù ($0 < \alpha < 1$).

### Xác định hệ số $\alpha$:
Hệ số $\alpha$ được tính toán thông qua hằng số thời gian $\tau$ và tần số cắt $f_c$:
$$\alpha = \frac{\tau}{\tau + \Delta t} \quad \text{với} \quad \tau = \frac{1}{2\pi f_c}$$

* Trong ứng dụng xe tự cân bằng thực tế, $\alpha$ thường được chọn trong khoảng **$0.95 \le \alpha \le 0.98$** (ưu tiên $95\% - 98\%$ đáp ứng nhanh của Gyroscope và dùng $2\% - 5\%$ trọng lực từ Accelerometer để ghì góc không bị trôi).

---

## 3. Mã nguồn triển khai trên STM32 (Ngôn ngữ C)

```c
#include "math.h"

#define COMP_ALPHA   0.96f    // Hệ số lọc bù thực nghiệm
#define DT_FILTER    0.005f   // Chu kỳ 5ms (200Hz)
#define RAD_TO_DEG   57.29577951308232f

typedef struct {
    float angle_pitch;
    float gyro_bias_y;
} ComplementaryFilter_t;

static ComplementaryFilter_t filter_instance = {0.0f, 0.0f};

/**
 * @brief Cập nhật bộ lọc bù mỗi chu kỳ 5ms
 * @param ax, ay, az: Dữ liệu gia tốc chuẩn hóa (g)
 * @param gy: Vận tốc góc trục Y (deg/s)
 * @return Góc pitch ước lượng (đơn vị: độ)
 */
float ComplementaryFilter_Update(float ax, float ay, float az, float gy) {
    // 1. Tính góc nghiêng tĩnh từ gia tốc kế
    float accel_pitch = atan2f(ax, sqrtf(ay * ay + az * az)) * RAD_TO_DEG;

    // 2. Trừ bias và tích phân Gyroscope kết hợp lọc bù
    float gyro_rate = gy - filter_instance.gyro_bias_y;
    filter_instance.angle_pitch = COMP_ALPHA * (filter_instance.angle_pitch + gyro_rate * DT_FILTER)
                                + (1.0f - COMP_ALPHA) * accel_pitch;

    return filter_instance.angle_pitch;
}
