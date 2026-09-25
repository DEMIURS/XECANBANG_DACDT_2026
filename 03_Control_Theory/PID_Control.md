---

### File 2: `03_Control_Theory/PID_Control.md`

```markdown
# Thuật Toán Điều Khiển PID (PID Control)

Tài liệu mô tả kiến trúc thuật toán điều khiển PID đơn vòng và PID lồng kép (Cascade PID) ứng dụng để giữ thăng bằng và ổn định vị trí cho xe 2 bánh tự cân bằng.

---

## 1. Nguyên lý Bộ điều khiển PID cơ bản

Bộ điều khiển PID điều chỉnh tín hiệu ngõ ra dựa trên sai số $e(t)$ giữa giá trị đặt (SetPoint) và phản hồi thực tế (Process Variable):

$$u(t) = K_p \, e(t) + K_i \int_{0}^{t} e(\tau) \, d\tau + K_d \, \frac{de(t)}{dt}$$
SetPoint
r(t)    +      e(t)     +-------------------+  u(t)   +-------+
-------->(+)------------>|   Bộ điều khiển   |-------->| Robot |-----+---> y(t)
^ -            |        PID        |         +-------+     |
|              +-------------------+                       |
+----------------------------------------------------------+
### Vai trò các khâu trong xe cân bằng:
* **Khâu Tỉ lệ ($K_p$)**: Cung cấp lực đẩy tức thời để kéo xe về vị trí thẳng đứng. Nếu $K_p$ quá nhỏ, xe không kịp nâng người; nếu quá lớn, xe sẽ rung lắc dữ dội quanh trục cân bằng.
* **Khâu Tích phân ($K_i$)**: Tích lũy sai lệch theo thời gian nhằm loại bỏ sai số xác lập (ví dụ khi trọng tâm xe bị lệch cơ khí về một bên).
* **Khâu Vi phân ($K_d$)**: Phản ứng dựa trên tốc độ thay đổi sai số, tạo lực cản động học (damping) nhằm dập tắt dao động và chống vọt lố.

---

## 2. Kiến trúc Điều khiển Vòng Lặp Kép (Cascade PID)

Nếu chỉ sử dụng 1 vòng PID cân bằng góc (Angle Loop), xe sẽ đứng thăng bằng nhưng trôi tự do theo quán tính trên mặt sàn. Để khắc phục, hệ thống áp dụng cấu trúc **PID lồng kép**:
[Vị trí / Tốc độ đặt] = 0
|
v
+--------------+     Góc đặt (θ_target)     +--------------+     PWM Output     +----------+
| Vòng Ngoài   |--------------------------->| Vòng Trong   |------------------->| Động cơ  |
| (Speed PID)  |                            | (Angle PID)  |                    +----------+
+--------------+                            +--------------+                          |
^                                            ^                                 |
|                                            |                                 v
[Encoder Feedback]                           [IMU Pitch Angle]                    [Khung xe]
### 2.1. Vòng trong - Điều khiển Góc (Angle PID Loop)
* **Tần số thực thi**: $200\text{ Hz}$ ($\Delta t = 5\text{ ms}$).
* **Mục tiêu**: Phản ứng cực nhanh để xe không bị ngã.
* **Phương trình rời rạc**:
  $$u_{angle} = K_{p1} \cdot (\theta_{\text{target}} - \theta_{\text{actual}}) + K_{i1} \sum e_{\theta} \cdot \Delta t - K_{d1} \cdot \omega_y$$
  *(Lưu ý: Khâu $D$ lấy trực tiếp từ giá trị vận tốc góc $\omega_y$ của con quay hồi chuyển Gyroscope nhằm tránh hiện tượng nhảy vọt đạo hàm khi SetPoint thay đổi).*

### 2.2. Vòng ngoài - Điều khiển Vận tốc/Vị trí (Speed PID Loop)
* **Tần số thực thi**: $20\text{ Hz} \sim 50\text{ Hz}$ ($\Delta t = 20\text{ ms} \sim 50\text{ ms}$).
* **Mục tiêu**: Giữ xe cố định tại tọa độ đứng yên hoặc di chuyển theo lệnh điều khiển qua Bluetooth.
* **Đầu vào**: Tốc độ trung bình đọc từ 2 kênh Encoder của động cơ JGA25-370.
* **Đầu ra**: Góc nghiêng mục tiêu $\theta_{\text{target}}$ cho vòng trong:
  * Khi xe bị đẩy trôi về phía trước $\rightarrow$ vòng ngoài sinh ra góc nghiêng về phía sau $\rightarrow$ vòng trong kéo xe lùi lại để triệt tiêu vị trí trôi.

---

## 3. Kỹ thuật Chống bão hòa Tích phân (Anti-Windup)

Khi góc nghiêng bị giữ lệch quá lâu hoặc xe bị giữ chặt bằng tay, khâu tích phân $\int e \, dt$ sẽ cộng dồn thành giá trị cực lớn. Khi thả tay, động cơ sẽ vọt hết công suất gây mất kiểm soát.

Giải pháp **Clamping (Kẹp ngưỡng tích phân)**:
```c
integral_error += error * dt;

// Giới hạn giá trị khâu tích phân trong phạm vi an toàn
if (integral_error > INTEGRAL_MAX)  integral_error = INTEGRAL_MAX;
if (integral_error < -INTEGRAL_MAX) integral_error = -INTEGRAL_MAX;
