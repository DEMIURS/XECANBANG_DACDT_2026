# ⚡ Điều Khiển Động Cơ DC (Motor Control)

Xử lý tín hiệu điều khiển đầu ra (PWM) và tín hiệu hồi tiếp vận tốc/vị trí (Quadrature Encoder).

---

## 1. Điều khiển phần cứng qua Mạch cầu H (H-Bridge)

Mạch cầu H cho phép đảo chiều điện áp đặt lên cuộn dây động cơ:

| Hướng chuyển động | IN1 / PWM1 | IN2 / PWM2 | Chế độ |
| :--- | :---: | :---: | :--- |
| **Quay thuận** | `PWM` | `0` (GND) | Forward Drive |
| **Quay nghịch** | `0` (GND) | `PWM` | Reverse Drive |
| **Phanh động cơ** | `1` (VCC) | `1` (VCC) | Fast Decay / Active Brake |
| **Thả trôi (Coast)** | `0` | `0` | Inertial Coasting |

---

## 2. Đo tốc độ qua Quadrature Encoder

* Cảm biến Encoder quang/từ phát ra 2 kênh xung lệch pha $90^\circ$ ($A$ và $B$).
* Bằng cách phát hiện cạnh lên/xuống của cả 2 kênh (chế độ **$4\times$ Decoding**), độ phân giải góc tăng lên gấp 4 lần.

Công thức tính tốc độ trục động cơ:

$$\omega = \frac{\Delta \text{Ticks}}{\text{PPR} \times \text{GearRatio} \times \Delta t} \times 60 \quad (\text{RPM})$$

---

## 3. Cấu trúc Cascade Loop (Vòng điều khiển lồng)

Trong hệ thống xe tự cân bằng hoặc bám quỹ đạo:

```mermaid
flowchart LR
    SP[Vị trí mong muốn] --> SpeedPID[Speed PID]
    SpeedPID -->|Góc nghiêng mong muốn| AnglePID[Angle PID]
    AnglePID -->|Tín hiệu PWM| Plant[Động cơ + Xe]
    
    Plant -->|Phản hồi tốc độ| Enc[Encoder]
    Plant -->|Phản hồi góc nghiêng| IMU[IMU Filter]
    
    Enc --> SpeedPID
    IMU --> AnglePID

---

## 4. Chống chết vùng điện áp (Deadzone Compensation)

Động cơ DC luôn có một mức điện áp tối thiểu $V_{\text{dead}}$ mà dưới mức đó lực ma sát tĩnh lớn hơn mô-men sinh ra, khiến trục không quay:

```c
int16_t Apply_Deadzone(int16_t pwm_calculated, int16_t deadzone_threshold, int16_t max_pwm) {
    if (pwm_calculated > 0) {
        pwm_calculated += deadzone_threshold;
    } else if (pwm_calculated < 0) {
        pwm_calculated -= deadzone_threshold;
    }
    
    // Giới hạn giá trị PWM
    if (pwm_calculated > max_pwm) return max_pwm;
    if (pwm_calculated < -max_pwm) return -max_pwm;
    return pwm_calculated;
}
