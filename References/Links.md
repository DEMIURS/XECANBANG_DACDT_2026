# 📚 Tài Liệu Tham Khảo (References & Technical Links)

Tổng hợp các nguồn tài liệu kỹ thuật, bài báo khoa học, datasheet chính hãng và công cụ phần mềm phục vụ nghiên cứu, thiết kế phần cứng và lập trình cho Dự án Xe Hai Bánh Tự Cân Bằng STM32.

---

## 1. Datasheet & Hướng Dẫn Kỹ Thuật Linh Kiện (Hardware Datasheets)

### 1.1. Vi Điều Khiển Trung Tâm - STM32F103C8T6
* [STMicroelectronics STM32F103x8/xB Datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf): Thông số điện áp, dòng điện, sơ đồ chân và đặc tính ngoại vi.
* [STM32F10xxx Reference Manual (RM0008)](https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf): Hướng dẫn chi tiết cấu hình thanh ghi Timer, I2C, UART, ADC và RCC.
* [STM32 Cortex®-M3 Programming Manual (PM0056)](https://www.st.com/resource/en/programming_manual/pm0056-stm32f10xxx20xxx21xxxl1xxxx-cortexm3-programming-manual-stmicroelectronics.pdf): Kiến trúc tập lệnh lõi ARM Cortex-M3 và ngắt NVIC.

### 1.2. Cảm Biến Đo Lường Quán Tính - Bosch GY-BMI160
* [Bosch Sensortec BMI160 Datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmi160-ds000.pdf): Thanh ghi I2C/SPI, dải đo gia tốc $\pm 2g \div \pm 16g$, dải đo con quay $\pm 125^\circ/\text{s} \div \pm 2000^\circ/\text{s}$.
* [Bosch Sensortec BMI160 Sensor API (Official GitHub)](https://github.com/boschsensortec/BMI160_SensorAPI): Thư viện mã nguồn C chính thức của hãng để giao tiếp và khởi tạo BMI160.

### 1.3. Mạch Điều Khiển Động Cơ - Toshiba TB6612FNG
* [Toshiba TB6612FNG Datasheet](https://toshiba.semicon-storage.com/info/TB6612FNG_datasheet_en_20141001.pdf): Đặc tính cầu H công nghệ MOSFET, bảo vệ quá nhiệt (TSD), dòng liên tục 1.2A, dòng đỉnh 3.2A.

### 1.4. Mạch Hạ Áp Nguồn Xung - Monolithic Power Systems MP1584EN
* [MPS MP1584 3A, 1.5MHz, 28V Step-Down Converter](https://www.monolithicpower.com/en/documentview/productdocument/index/version/2/document_type/Datasheet/lang/en/sku/MP1584/): Nguyên lý tính toán tụ lọc, cuộn cảm và điện trở chia áp hồi tiếp.

### 1.5. Module Truyền Thông Bluetooth HC-05
* [HC-05 Bluetooth Module User's Manual & AT Command Set](https://components101.com/sites/default/files/component_datasheet/HC-05%20Datasheet.pdf): Tập lệnh AT để cài đặt Baudrate, Name, PIN code và chế độ Master/Slave.

---

## 2. Tài Liệu Học Thuật & Bài Báo Khoa Học (Academic Papers)

### 2.1. Mô hình hóa toán học Xe hai bánh tự cân bằng (TWIP - Two-Wheeled Inverted Pendulum)
1. **Mathematical Modeling and Control of a Two-Wheeled Self-Balancing Robot**
   * *Nội dung:* Xây dựng hệ phương trình vi phân chuyển động phi tuyến dựa trên phương pháp cơ học Lagrange (Euler-Lagrange equations) và tuyến tính hóa quanh điểm cân bằng thẳng đứng.
   * *Định dạng:* Lưu tại thư mục `References/Papers/`.
2. **Comparison of Complementary and Kalman Filter for Inverted Pendulum Tilt Estimation**
   * *Tác giả & Nhà xuất bản:* IEEE Conference on Robotics and Automation.
   * *Nội dung:* So sánh tài nguyên tính toán (RAM/Flash, thời gian thực thi CPU) và độ chính xác ước lượng góc giữa Bộ lọc bù và Bộ lọc Kalman trên vi điều khiển ARM 32-bit.
3. **Dual-Loop PID Controller Design for Two-Wheeled Self-Balancing Mobile Robot**
   * *Nội dung:* Phân tích ổn định Lyapunov cho cấu trúc điều khiển phân tầng: vòng trong (PD góc nghiêng) và vòng ngoài (PI vận tốc xe).

---

## 3. Công Cụ Phát Triển & Phần Mềm Phục Vụ Dự Án (Tools & Utilities)

* **Phần mềm biên dịch & nạp code:**
  * [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html): Môi trường phát triển chính thức tích hợp trình tạo cấu hình phần cứng STM32CubeMX và trình biên dịch GCC ARM.
  * [STM32CubeProg](https://www.st.com/en/development-tools/stm32cubeprog.html): Nạp firmware qua mạch nạp ST-Link V2 hoặc nạp Serial Bootloader qua UART.
* **Công cụ vẽ đồ thị và giám sát dữ liệu thời gian thực (Real-time Tuning & Telemetry):**
  * [Teleplot (VS Code Extension)](https://marketplace.visualstudio.com/items?itemName=alexnesnes.teleplot): Vẽ đồ thị góc nghiêng và tín hiệu PID từ cổng Serial UART cực nhanh và trực quan.
  * [Serial Studio](https://serial-studio.github.io/): Ứng dụng nguồn mở giám sát trực quan telemetry xe cân bằng qua đồ thị và la bàn ảo 3D.
* **Thiết kế phần cứng & Mạch in (EDA):**
  * Altium Designer: Thiết kế Schematic, Layout mạch 2 lớp và xuất Gerber công nghiệp.
