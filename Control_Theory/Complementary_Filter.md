# 📡 Bộ lọc bù (Complementary Filter)

Bộ lọc bù kết hợp góc ước lượng từ gia tốc kế với tốc độ góc từ con quay hồi chuyển. Gia tốc kế giữ góc lâu dài ổn định nhưng nhiễu khi xe rung hoặc tăng tốc; con quay hồi chuyển phản ứng nhanh nhưng sai lệch bias sẽ làm góc tích phân bị trôi.

---

## 1. Phương trình rời rạc

Ở mỗi chu kỳ lấy mẫu, bộ lọc tích phân tốc độ góc rồi hiệu chỉnh kết quả bằng góc gia tốc kế:

$$\theta_k = \alpha\left(\theta_{k-1} + \omega_k\Delta t\right) + (1-\alpha)\theta_{acc,k}$$

Trong đó:

* $\theta_k$: góc sau lọc tại mẫu hiện tại, tính bằng độ.
* $\theta_{acc,k}$: góc tính từ gia tốc kế, tính bằng độ.
* $\omega_k$: tốc độ góc đã trừ bias, tính bằng độ/giây và cùng chiều trục với góc đang ước lượng.
* $\Delta t$: thời gian giữa hai mẫu, tính bằng giây.
* $\alpha$: trọng số của gyro, với $0 < \alpha < 1$. Giá trị thường dùng ở chu kỳ lấy mẫu ổn định là khoảng 0,95–0,99; cần hiệu chỉnh theo tần số lấy mẫu và mức rung thực tế.

Gia tốc kế và gyro phải dùng cùng quy ước trục, dấu và đơn vị. Công thức một trục dưới đây phù hợp với góc nghiêng quanh một trục, đặc biệt khi xe hoạt động gần tư thế thẳng đứng. Chọn góc và tốc độ góc tương ứng theo cách lắp BMI160; không ghép góc roll với tốc độ gyro của trục pitch.

## 2. Tính góc từ gia tốc kế

Với quy ước trục đã kiểm tra theo cách lắp cảm biến:

* Roll: $\phi = \operatorname{atan2}(A_y,A_z)\cdot 180/\pi$.
* Pitch: $\theta = \operatorname{atan2}(-A_x,\sqrt{A_y^2+A_z^2})\cdot 180/\pi$.

Đổi trục hoặc dấu nếu hướng đặt cảm biến trên xe khác với quy ước trên. Khi xe đang tăng tốc, gia tốc kế cũng đo gia tốc chuyển động nên góc tức thời có thể sai; đây là lý do bộ lọc giảm trọng số tín hiệu này ở dải tần cao.

## 3. Mã C minh họa cho một trục

Hàm nhận góc gia tốc kế và tốc độ gyro đã hiệu chuẩn. Phía gọi chịu trách nhiệm tính `accel_angle_deg` theo trục lắp đặt và đổi gyro sang `deg/s`.

```c
typedef struct {
    float alpha;        // 0 < alpha < 1
    float angle_deg;    // Góc ước lượng hiện tại
} ComplementaryFilter;

void CompFilter_Init(ComplementaryFilter *filter, float alpha,
                     float initial_angle_deg) {
    if (filter == 0) return;
    filter->alpha = alpha;
    filter->angle_deg = initial_angle_deg;
}

float CompFilter_Update(ComplementaryFilter *filter,
                        float accel_angle_deg,
                        float gyro_rate_dps,
                        float dt_s) {
    if (filter == 0 || dt_s <= 0.0f ||
        filter->alpha <= 0.0f || filter->alpha >= 1.0f) {
        return filter != 0 ? filter->angle_deg : 0.0f;
    }

    filter->angle_deg = filter->alpha *
                        (filter->angle_deg + gyro_rate_dps * dt_s) +
                        (1.0f - filter->alpha) * accel_angle_deg;
    return filter->angle_deg;
}
```

Trước khi gọi hàm, kiểm tra con trỏ và dữ liệu cảm biến hợp lệ; giá trị `dt_s` phải là thời gian thực giữa các mẫu, không phải mili giây. Nếu chu kỳ lấy mẫu thay đổi đáng kể, hãy tính lại `alpha` theo chu kỳ mới hoặc dùng hệ số phụ thuộc thời gian.

## 4. Tài liệu liên quan

* [Lý thuyết cảm biến IMU](IMU_Theory.md)
* [Tài liệu tham khảo và datasheet](../References/Links.md)
