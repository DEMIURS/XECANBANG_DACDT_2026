# ⚙️ Điều Khiển Động Cơ DC & Xử Lý Tín Hiệu Encoder

Tài liệu chi tiết về nguyên lý điều khiển phần cứng cơ cấu chấp hành (động cơ DC có hộp giảm tốc JGA25-370), mạch cầu H công suất TB6612FNG, kỹ thuật giải mã Quadrature Encoder qua STM32 Timer, và giải pháp xử lý vùng chết phi tuyến (Deadzone Compensation).

---

## 1. Mạch cầu H & Driver Động Cơ TB6612FNG

Mạch cầu H (H-Bridge) cho phép đảo chiều dòng điện chạy qua cuộn dây phần ứng của động cơ DC, từ đó điều khiển chiều quay thuận (CW) hoặc nghịch (CCW) và điều chỉnh tốc độ thông qua tín hiệu điều chế độ rộng xung (PWM).

### 1.1. Sơ đồ nguyên lý logic mạch cầu H
```
            +V_Motor (Pin 3S: ~11.1V - 12.6V)
                 |
         +-------+-------+
         |               |
       [ Q1 ]          [ Q3 ]  (High-side MOSFETs)
         |               |
         +----( M )------+
         |    Motor      |
       [ Q2 ]          [ Q4 ]  (Low-side MOSFETs)
         |               |
         +-------+-------+
                 |
                GND
```

* **Quay thuận:** Đóng Q1 và Q4 $\rightarrow$ Dòng điện chạy từ trái sang phải qua động cơ.
* **Quay nghịch:** Đóng Q3 and Q2 $\rightarrow$ Dòng điện đảo chiều chạy từ phải sang trái.
* **Phanh hãm (Brake):** Đóng đồng thời Q2 và Q4 (nối tắt 2 cực động cơ xuống GND) $\rightarrow$ Sinh ra sực điện động ngược dập tắt quán tính nhanh chóng.
* **Thả trôi (Coast):** Mở tất cả các khóa transistor $\rightarrow$ Động cơ dừng quay tự do theo quán tính.

### 1.2. Bảng chân lý điều khiển Driver TB6612FNG
Driver **TB6612FNG** sử dụng transistor công nghệ MOSFET công suất thấp, tản nhiệt tốt và hiệu suất vượt trội so với dòng IC BJT cũ như L298N.

| Chân STBY | Chân IN1 | Chân IN2 | Chân PWM | Trạng thái kênh A/B | Chế độ hoạt động |
| :---: | :---: | :---: | :---: | :---: | :--- |
| `0` (LOW) | $\times$ | $\times$ | $\times$ | Ngắt hoàn toàn | **Chế độ chờ (Standby Mode) - Tiết kiệm điện** |
| `1` (HIGH)| `1` | `0` | `Duty` | $V_{out} = +V_{bat} \times \text{Duty}$ | **Quay thuận (Forward Drive)** |
| `1` (HIGH)| `0` | `1` | `Duty` | $V_{out} = -V_{bat} \times \text{Duty}$ | **Quay nghịch (Reverse Drive)** |
| `1` (HIGH)| `0` | `0` | $\times$ | Cực hở (High-Z) | **Thả trôi theo quán tính (Coast)** |
| `1` (HIGH)| `1` | `1` | $\times$ | Nối đất (GND) | **Phanh động cơ chủ động (Short Brake)** |

### 1.3. Lựa chọn tần số phát xung PWM trên STM32
Tần số điều chế xung PWM cấp cho chân `PWMA` và `PWMB` được chọn tối ưu trong khoảng **$10\text{ kHz} \div 20\text{ kHz}$**:
* **Không chọn tần số thấp ($< 8\text{ kHz}$):** Cuộn cảm động cơ sẽ dao động tạo tiếng rít tần số âm thanh chói tai và làm dòng điện gợn sóng lớn.
* **Không chọn tần số quá cao ($> 50\text{ kHz}$):** Tổn hao đóng cắt (switching loss) trên MOSFET của TB6612FNG tăng cao gây phát nhiệt.
* **Cấu hình Timer STM32 (Timer Clock = 72 MHz):**
  * `Prescaler (PSC)`: $0$ (Clock đếm 72 MHz)
  * `Auto-Reload Register (ARR)`: $3599 \rightarrow f_{PWM} = \frac{72\text{ MHz}}{3600} = 20\text{ kHz}$.
  * Dải điều chỉnh độ rộng xung: $0 \div 3599$.

---

## 2. Đo Tốc Độ & Vị Trí Qua Quadrature Encoder

Động cơ **JGA25-370** tích hợp đĩa từ encoder 2 kênh Hall lệch pha $90^\circ$ điện (Phase A và Phase B).

```
Kênh A:  ___|‾‾‾|___|‾‾‾|___|‾‾‾|___   (Vuông góc lệch pha)
Kênh B:  _____|‾‾‾|___|‾‾‾|___|‾‾‾|_
```

### 2.1. Chế độ giải mã 4x (4X Decoding Mode)
Bằng cách kích hoạt chế độ **Encoder Mode (TIM_ENCODERMODE_TI12)** trên Timer của STM32, phần cứng sẽ tự động bắt cả 2 sườn (cạnh lên và cạnh xuống) của cả 2 kênh A và B mà không tốn tài nguyên CPU:
* Độ phân giải tăng gấp 4 lần so với chỉ đếm cạnh lên kênh A.
* Hướng quay (tiến hay lùi) được phần cứng Timer tự động nhận diện qua thứ tự pha:
  * Nếu Kênh A dẫn trước Kênh B: Timer tự đếm tiến (Up-counting).
  * Nếu Kênh B dẫn trước Kênh A: Timer tự đếm lùi (Down-counting).

### 2.2. Công thức quy đổi vận tốc
* Số xung gốc cảm biến Hall: $\text{PPR}_{sensor} = 11\text{ xung/kênh/vòng}$.
* Ở chế độ giải mã 4X: $11 \times 4 = 44\text{ xung/vòng trục motor}$.
* Tỉ số truyền hộp giảm tốc (Gear Ratio) $i$ (ví dụ: $i \approx 21.3$ đối với bản 280 RPM):
  $$\text{Total Ticks Per Rev (Trục bánh xe)} = \text{PPR} \times 4 \times i \approx 44 \times 21.3 \approx 937.2\text{ xung/vòng}$$

#### Công thức tính vận tốc góc của bánh xe (vòng/phút - RPM):
$$\omega_{RPM} = \frac{\Delta \text{Ticks}}{\text{Ticks\_Per\_Rev} \times \Delta t} \times 60$$

#### Vận tốc dài của xe ($v$ tính bằng $\text{m/s}$):
$$v = \frac{\omega_{RPM} \times 2\pi}{60} \times R_{wheel} = \frac{\Delta \text{Ticks}}{\text{Ticks\_Per\_Rev} \times \Delta t} \times (2\pi R_{wheel})$$
*(Trong đó $R_{wheel}$ là bán kính bánh xe bằng mét, thường bánh xe đường kính $65\text{mm} \rightarrow R \approx 0.0325\text{m}$).*

---

## 3. Hiện Tượng Vùng Chết Động Cơ (Deadzone Compensation)

Trong thực tế, do lực ma sát tĩnh (static friction / stiction) trong các cặp bánh răng của hộp giảm tốc và chổi than động cơ, khi điện áp đặt lên động cơ nhỏ hơn một ngưỡng điện áp tối thiểu $V_{dead}$ (tương ứng với xung PWM nhỏ), mô-men sinh ra không đủ thắng ma sát và động cơ **hoàn toàn không quay**.

Nếu không bù vùng chết:
* Xe khi lệch góc nhỏ sẽ xuất ra tín hiệu PWM nhỏ nằm trong vùng chết $\rightarrow$ Bánh xe không phản ứng $\rightarrow$ Góc nghiêng tiếp tục tăng đến khi PWM vượt qua vùng chết thì xe mới giật mạnh $\rightarrow$ Dẫn đến hiện tượng dao động lắc chu kỳ giới hạn (Limit Cycle Oscillation).

### Thuật toán bù vùng chết:
```c
int16_t Motor_CompensateDeadzone(int16_t pwm_raw, int16_t deadzone, int16_t max_pwm) {
    int16_t pwm_out = 0;

    if (pwm_raw > 0) {
        pwm_out = pwm_raw + deadzone;
    } else if (pwm_raw < 0) {
        pwm_out = pwm_raw - deadzone;
    } else {
        pwm_out = 0;
    }

    // Kẹp dải giá trị trong biên độ an toàn
    if (pwm_out > max_pwm) pwm_out = max_pwm;
    else if (pwm_out < -max_pwm) pwm_out = -max_pwm;

    return pwm_out;
}
```

---

## 4. Phối Hợp Điều Khiển Bánh Xe & Bẻ Lái (Steering / Yaw Control)

Xe hai bánh cân bằng di chuyển và đổi hướng bằng cơ chế vi sai (Differential Drive):

$$\begin{cases}
PWM_{Left} = PWM_{balance} + PWM_{speed} + PWM_{turn} \\
PWM_{Right} = PWM_{balance} + PWM_{speed} - PWM_{turn}
\end{cases}$$

Trong đó:
* $PWM_{balance}$: Tín hiệu từ vòng lặp cân bằng góc (Angle PID), giữ cho xe không ngã.
* $PWM_{speed}$: Tín hiệu từ vòng lặp vận tốc (Speed PID), điều khiển xe tiến/lùi.
* $PWM_{turn}$: Tín hiệu điều khiển xoay hướng nhận từ Bluetooth/App điều khiển từ xa.

---

## 5. Mã Nguồn C Driver Mẫu Cho STM32 HAL

### File header (`motor_driver.h`):

```c
#ifndef MOTOR_DRIVER_H
#define MOTOR_DRIVER_H

#include "stm32f1xx_hal.h"

#define PWM_MAX_VALUE       3599
#define MOTOR_DEADZONE      350     // Tùy chỉnh theo thực nghiệm

void Motor_Init(void);
void Motor_SetOutput(int16_t pwm_left, int16_t pwm_right);
void Motor_Stop(void);

int16_t Encoder_GetLeftTicks(void);
int16_t Encoder_GetRightTicks(void);

#endif
```

### File triển khai (`motor_driver.c`):

```c
#include "motor_driver.h"

extern TIM_HandleTypeDef htim1; // Timer PWM (CH1, CH2)
extern TIM_HandleTypeDef htim2; // Timer Encoder Trái
extern TIM_HandleTypeDef htim3; // Timer Encoder Phải

// Định nghĩa các chân hướng IN1, IN2 của TB6612
#define AIN1_PORT   GPIOA
#define AIN1_PIN    GPIO_PIN_4
#define AIN2_PORT   GPIOA
#define AIN2_PIN    GPIO_PIN_5

#define BIN1_PORT   GPIOB
#define BIN1_PIN    GPIO_PIN_0
#define BIN2_PORT   GPIOB
#define BIN2_PIN    GPIO_PIN_1

#define STBY_PORT   GPIOA
#define STBY_PIN    GPIO_PIN_6

void Motor_Init(void) {
    // Bật chân STBY của TB6612
    HAL_GPIO_WritePin(STBY_PORT, STBY_PIN, GPIO_PIN_SET);

    // Bắt đầu phát xung PWM
    HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1); // PWMA (Motor Trái)
    HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_2); // PWMB (Motor Phải)

    // Bắt đầu bộ đếm Encoder
    HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_ALL);
    HAL_TIM_Encoder_Start(&htim3, TIM_CHANNEL_ALL);
}

static int16_t Apply_Deadzone(int16_t val) {
    if (val > 0) val += MOTOR_DEADZONE;
    else if (val < 0) val -= MOTOR_DEADZONE;
    if (val > PWM_MAX_VALUE) val = PWM_MAX_VALUE;
    if (val < -PWM_MAX_VALUE) val = -PWM_MAX_VALUE;
    return val;
}

void Motor_SetOutput(int16_t pwm_left, int16_t pwm_right) {
    pwm_left = Apply_Deadzone(pwm_left);
    pwm_right = Apply_Deadzone(pwm_right);

    // --- Điều khiển Motor Trái ---
    if (pwm_left > 0) {
        HAL_GPIO_WritePin(AIN1_PORT, AIN1_PIN, GPIO_PIN_SET);
        HAL_GPIO_WritePin(AIN2_PORT, AIN2_PIN, GPIO_PIN_RESET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, (uint32_t)pwm_left);
    } else if (pwm_left < 0) {
        HAL_GPIO_WritePin(AIN1_PORT, AIN1_PIN, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(AIN2_PORT, AIN2_PIN, GPIO_PIN_SET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, (uint32_t)(-pwm_left));
    } else {
        HAL_GPIO_WritePin(AIN1_PORT, AIN1_PIN, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(AIN2_PORT, AIN2_PIN, GPIO_PIN_RESET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 0);
    }

    // --- Điều khiển Motor Phải ---
    if (pwm_right > 0) {
        HAL_GPIO_WritePin(BIN1_PORT, BIN1_PIN, GPIO_PIN_SET);
        HAL_GPIO_WritePin(BIN2_PORT, BIN2_PIN, GPIO_PIN_RESET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_2, (uint32_t)pwm_right);
    } else if (pwm_right < 0) {
        HAL_GPIO_WritePin(BIN1_PORT, BIN1_PIN, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(BIN2_PORT, BIN2_PIN, GPIO_PIN_SET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_2, (uint32_t)(-pwm_right));
    } else {
        HAL_GPIO_WritePin(BIN1_PORT, BIN1_PIN, GPIO_PIN_RESET);
        HAL_GPIO_WritePin(BIN2_PORT, BIN2_PIN, GPIO_PIN_RESET);
        __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_2, 0);
    }
}

int16_t Encoder_GetLeftTicks(void) {
    int16_t ticks = (int16_t)__HAL_TIM_GET_COUNTER(&htim2);
    __HAL_TIM_SET_COUNTER(&htim2, 0); // Đặt lại bộ đếm sau mỗi chu kỳ
    return ticks;
}

int16_t Encoder_GetRightTicks(void) {
    int16_t ticks = (int16_t)__HAL_TIM_GET_COUNTER(&htim3);
    __HAL_TIM_SET_COUNTER(&htim3, 0);
    return ticks;
}
```
