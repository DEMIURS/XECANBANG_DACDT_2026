
### `03_Control_Theory/Motor_Control.md`

```markdown
# Điều Khiển Động Cơ & Đọc Phản Hồi Vận Tốc (Motor & Encoder Control)

Tài liệu chi tiết về kỹ thuật điều khiển động cơ DC giảm tốc JGA25-370 thông qua driver cầu H TB6612FNG và phương pháp đọc tín hiệu Encoder bằng chế độ phần cứng Timer trên STM32F103.

---

## 1. Giao tiếp Driver Cầu H TB6612FNG

Module **TB6612FNG** sử dụng transistor MOSFET hiệu suất cao, tỏa nhiệt ít và hiệu suất vượt trội so với dòng driver cũ như L298N.

### 1.1. Sơ đồ logic điều khiển
* **STBY (Standby)**: Kích hoạt driver (mức High để cho phép cầu H dẫn dòng).
* **PWMA / PWMB**: Tín hiệu điều chế độ rộng xung xuất từ Timer của STM32 để điều chỉnh điện áp trung bình đặt lên động cơ.
* **AIN1, AIN2 (Kênh trái) / BIN1, BIN2 (Kênh phải)**: Thiết lập chiều quay.

| Chân IN1 | Chân IN2 | PWM | Chiều quay / Chế độ |
| :---: | :---: | :---: | :--- |
| **HIGH** | **LOW** | Tín hiệu xung | Quay thuận (Forward) |
| **LOW** | **HIGH** | Tín hiệu xung | Quay ngược (Reverse) |
| **HIGH** | **HIGH** | X | Hãm ngắn mạch chủ động (Short brake) |
| **LOW** | **LOW** | X | Dừng tự do thả trôi (Stop / Coast) |

### 1.2. Tần số xung PWM tối ưu
* Cấu hình tần số PWM trong dải **$10\text{ kHz} \le f_{PWM} \le 20\text{ kHz}$**.
* Tần số trên $16\text{ kHz}$ nằm ngoài ngưỡng nghe thấy của tai người, giúp triệt tiêu hoàn toàn tiếng rít chói tai từ cuộn dây động cơ.

---

## 2. Kỹ thuật Đọc Tín Hiệu Encoder Bằng Phần Cứng Timer

Động cơ JGA25-370 tích hợp cụm encoder từ tính hai kênh pha $A$ và $B$ lệch pha nhau $90^\circ$ điện.

### 2.1. Chế độ đếm x4 (Encoder Mode 3)
Sử dụng chế độ ngoại vi **TIMx Encoder Interface** của STM32:
* Bộ đếm phần cứng tự động tăng hoặc giảm dựa vào trạng thái sườn lên/sườn xuống của cả kênh $A$ và kênh $B$.
* Không tiêu tốn thời gian ngắt CPU (Interrupt overhead).
* Tăng độ phân giải đo lên gấp 4 lần:

$$\text{Tổng số xung / 1 vòng bánh xe} = \text{PPR}_{\text{đĩa từ}} \times 4 \times \text{Tỉ số truyền hộp số}$$

```text
Kênh A:  __|‾‾|__|‾‾|__
Kênh B:  ___|‾‾|__|‾‾|_
Đếm x4:  ↑  ↑  ↑  ↑  (Đếm trên tất cả các sườn đổi mức)
// Đọc số xung tích lũy trong chu kỳ vừa qua
int16_t pulses_left = (int16_t)__HAL_TIM_GET_COUNTER(&htim2);
__HAL_TIM_SET_COUNTER(&htim2, 0); // Đặt lại bộ đếm về 0

int16_t pulses_right = (int16_t)__HAL_TIM_GET_COUNTER(&htim3);
__HAL_TIM_SET_COUNTER(&htim3, 0);
int16_t Motor_CompensateDeadzone(int16_t calculated_pwm) {
    const int16_t PWM_DEADZONE = 100; // Giá trị tìm qua thực nghiệm thử tải
    const int16_t PWM_MAX      = 1000;

    if (calculated_pwm > 0) {
        calculated_pwm += PWM_DEADZONE;
    } else if (calculated_pwm < 0) {
        calculated_pwm -= PWM_DEADZONE;
    }

    // Kẹp xung PWM không vượt quá ngưỡng trần của Timer
    if (calculated_pwm > PWM_MAX)  calculated_pwm = PWM_MAX;
    if (calculated_pwm < -PWM_MAX) calculated_pwm = -PWM_MAX;

    return calculated_pwm;
}
