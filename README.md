<p align="left">
  <a href="https://devicelab.vn/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-dark.svg?v=3.45.0">
      <source media="(prefers-color-scheme: light)" srcset="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-light.svg?v=3.45.0">
      <img alt="DeviceLab - Industrial IoT Hardware & Embedded R&D" src="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-light.svg?v=3.45.0" height="38">
    </picture>
  </a>
</p>

# Industrial Machinery Retrofit (OEE, Air Compressor `MAM-860/880`) & 3-Phase EMS Energy Metering (`DL-M400` / `DL-P300`) 🛠️

> **Tài liệu kỹ thuật mở hướng dẫn Retrofit thu thập dữ liệu máy nén khí trục vít (`MAM-860 / MAM-880`), đếm sản lượng chu kỳ máy CNC / máy ép nhựa tính OEE không can thiệp PLC bằng module `DL-M400`, và Bảng thanh ghi đo đếm điện năng 3 pha True RMS (`DL-P300`) phục vụ quản lý năng lượng EMS & kiểm kê khí nhà kính.**  
> Phát triển và sản xuất bởi **[DeviceLab](https://devicelab.vn/)** (CÔNG TY TNHH ĐẦU TƯ CÔNG NGHỆ ALPHA TECH - MST: `0111171963`).  
> 🔗 **Module IoT Tủ Máy & OEE:** [Module IoT Gắn Tủ Máy Nén Khí & OEE DL-M400](https://devicelab.vn/san-pham/dl-m400/) · [Giải pháp IoT cho Hãng chế tạo máy & Tủ điện](https://devicelab.vn/iot-cho-may-moc-tu-dien/)  
> 🔗 **Module Đo Điện Năng 3 Pha:** [Module Đo Đếm Điện Năng Tủ Điện DL-P300](https://devicelab.vn/san-pham/dl-p300/) · [Giải pháp Giám sát Điện năng & Dữ liệu Máy cũ](https://devicelab.vn/giam-sat-dien-nang-nha-may/)  
> 🔗 **Nền tảng OEM cho Hãng máy:** [Nền tảng IoT White-label cho Hãng máy công nghiệp](https://devicelab.vn/nen-tang-iot-white-label/)  
> 📖 **Đọc bài phân tích chi tiết tiếng Việt trên Viblo & Tinh Tế:** [Viblo - Phân biệt Transparent Converter và Modbus Gateway: Xử lý nhiễu biến tần & lưu đệm Flash](https://viblo.asia/p/phan-biet-transparent-converter-va-modbus-gateway-cach-xu-ly-nhieu-bien-tan-va-luu-dem-flash-khi-rot-mang-4g-bA46807wLKv) · [Viblo - Checklist thiết kế PCB chuẩn DFM & chống nhiễu EMC](https://viblo.asia/p/tu-bo-mach-mau-prototype-den-san-xuat-1000-thiet-bi-iot-cong-nghiep-checklist-thiet-ke-pcb-chuan-dfm-chong-nhieu-emc-OXLA0WqvJGr) · [Tinh Tế - Từ bo mạch mẫu đến sản xuất 1.000 thiết bị IoT: Checklist PCB & EMC](https://tinhte.vn/thread/tu-bo-mach-mau-prototype-den-san-xuat-1-000-thiet-bi-iot-checklist-thiet-ke-pcb-emc.4185069/)

---

## 📌 1. Kết Nối Modbus RTU Bộ Điều Khiển Máy Nén Khí Trục Vít (`MAM-860` / `MAM-880` / `MAM-6080`)

Các dòng máy nén khí trục vít công nghiệp phổ biến tại Việt Nam thường sử dụng bộ điều khiển vi xử lý **Plank `MAM-860`, `MAM-880` hoặc `MAM-6080`**. Khi tích hợp với **[Module IoT gắn tủ máy DL-M400](https://devicelab.vn/san-pham/dl-m400/)**, kỹ thuật viên chỉ mất 15 phút đấu 2 dây RS485 (`A+`, `B-`) và nguồn `24VDC` để đọc toàn bộ thông số vận hành lên Web/Mobile App mà không làm ảnh hưởng chương trình điều khiển của máy.

### Bảng Thanh Ghi Modbus RTU Chuẩn Của Bộ Điều Khiển `MAM-860 / MAM-880` (`9600 bps, 8-N-1`, Function Code `0x03`)

| Địa chỉ Hex | Địa chỉ Dec | Thông số vận hành máy nén khí | Đơn vị / Hệ số quy đổi | Ghi chú kỹ thuật |
| :--- | :--- | :--- | :--- | :--- |
| **`0x0064`** | `100` | Áp suất khí nén đầu ra (`Discharge Pressure`) | `0.01 MPa` (VD: `75` = `0.75 MPa` = `7.5 bar`) | Cảnh báo quá áp khi `> 0.85 MPa` |
| **`0x0065`** | `101` | Nhiệt độ đầu nén trục vít (`Exhaust Temperature`) | `1 °C` (hoặc `Offset -50` tùy phiên bản ROM) | Cảnh báo nhiệt độ cao khi `> 95 °C`, ngắt máy ở `105 °C` |
| **`0x0066`** | `102` | Dòng điện động cơ chính Pha A (`Main Motor Current`) | `0.1 A` | Phát hiện quá tải trục vít hoặc kẹt đầu nén |
| **`0x0068`** | `104` | Trạng thái vận hành & Cờ cảnh báo (`Run/Load Status Word`) | `Bitmask 16-bit` | `Bit 0`: Đang chạy (`Run`), `Bit 1`: Có tải (`Load`), `Bit 4`: Lỗi (`Fault`) |
| **`0x006A`–`006B`** | `106`–`107` | Tổng số giờ máy chạy (`Total Running Hours`) | `UInt32` (Giờ) | Dùng để nhắc lịch bảo dưỡng định kỳ (2.000h / 4.000h) |
| **`0x006C`–`006D`** | `108`–`109` | Tổng số giờ chạy có tải (`Total Loaded Hours`) | `UInt32` (Giờ) | Tính hiệu suất tiêu thụ điện năng trên mỗi $\text{m}^3$ khí nén |

> 📖 *Hướng dẫn chuyên sâu:*  
> - [Kết nối Modbus bộ điều khiển máy nén khí MAM-860 & MAM-880](https://devicelab.vn/kien-thuc/ket-noi-modbus-bo-dieu-khien-may-nen-khi-mam-860-mam-880/)  
> - [Giám sát máy nén khí từ xa bằng IoT & Cảnh báo bảo trì dự đoán](https://devicelab.vn/kien-thuc/giam-sat-may-nen-khi-tu-xa-iot/)

---

## ⚙️ 2. Giải Pháp Retrofit Đếm Sản Lượng Chu Kỳ Máy CNC, Máy Ép Nhựa, Máy Dập Cơ (Tính OEE)

Đối với các máy cơ khí đời cũ không có cổng truyền thông PLC mở (hoặc bị khóa mật khẩu hãng), **[Module DL-M400](https://devicelab.vn/san-pham/dl-m400/)** thu thập dữ liệu trực tiếp qua các ngõ vào Digital cách ly quang (`DI 10kHz`) và biến dòng:

1. **`DI1` (Machine Power ON):** Đấu song song đèn báo nguồn tủ máy (`24VDC`).
2. **`DI2` (Spindle / Cycle Run):** Đấu từ tiếp điểm phụ (`Auxiliary Contact`) của Contactor trục chính hoặc van điện từ đóng khuôn (`Mold Close Solenoid`).
3. **`DI3` (Part Counter):** Nhận xung từ rơ-le cắt phôi, cảm biến tiệm cận đếm sản phẩm hoàn thành (`Cycle Complete`) với bộ lọc chống dội xung phần mềm (`Software Debounce Filter` tùy chỉnh `20ms–500ms`).
4. **`DI4` (Alarm / Fault):** Đấu vào chân đèn đỏ cột đèn tháp (`Andon Tower Light Red`) để ghi nhận thời gian dừng máy sự cố (`Downtime`).

> 📖 *Xem chi tiết kỹ thuật:*  
> - [Thu thập tín hiệu chu kỳ và sản lượng máy CNC, máy dập cơ](https://devicelab.vn/kien-thuc/thu-thap-tin-hieu-chu-ky-va-san-luong-may-cnc-may-dap-co/)  
> - [Giải pháp lấy dữ liệu Run/Stop và sản lượng máy ép nhựa không cổng truyền thông](https://devicelab.vn/kien-thuc/giai-phap-lay-du-lieu-run-stop-san-luong-may-ep-nhua/)

---

## ⚡ 3. Bảng Thanh Ghi Đo Đếm Điện Năng 3 Pha True RMS (`DL-P300` - Class 0.5S)

**[Module đo đếm điện năng tủ điện DL-P300](https://devicelab.vn/san-pham/dl-p300/)** sử dụng biến dòng kẹp hở (**Split-Core CT** từ `50A` đến `1000A`), lắp đặt trong 10 phút không cần cắt cáp hay ngắt điện nhà xưởng, đạt cấp chính xác **Class 0.5S (IEC 62053-22)**:

| Địa chỉ Thanh ghi | Mã Thông Số | Mô tả thông số đo lường | Kiểu dữ liệu | Đơn vị | Quyền truy cập |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`30001` – `30006`** | `Ua`, `Ub`, `Uc` | Điện áp pha `L1-N`, `L2-N`, `L3-N` (True RMS) | `Float32 (ABCD)` | `V` | Read Only (`FC 04`) |
| **`30007` – `30012`** | `Uab`, `Ubc`, `Uca` | Điện áp dây `L1-L2`, `L2-L3`, `L3-L1` | `Float32 (ABCD)` | `V` | Read Only (`FC 04`) |
| **`30013` – `30018`** | `Ia`, `Ib`, `Ic` | Dòng điện tải 3 pha `L1`, `L2`, `L3` | `Float32 (ABCD)` | `A` | Read Only (`FC 04`) |
| **`30019` – `30020`** | `P_Total` | Tổng công suất tác dụng 3 pha ($P$) | `Float32 (ABCD)` | `kW` | Read Only (`FC 04`) |
| **`30021` – `30022`** | `Q_Total` | Tổng công suất phản kháng 3 pha ($Q$) | `Float32 (ABCD)` | `kVAR` | Read Only (`FC 04`) |
| **`30025` – `30026`** | `PF_Avg` | Hệ số công suất trung bình ($\cos\varphi$) | `Float32 (ABCD)` | `-` | Read Only (`FC 04`) |
| **`30027` – `30028`** | `Freq` | Tần số lưới điện | `Float32 (ABCD)` | `Hz` | Read Only (`FC 04`) |
| **`30029` – `30030`** | `kWh_Import` | Tổng điện năng tiêu thụ tích lũy (`Active Energy`) | `Float32` / `UInt32` | `kWh` | Read Only (`FC 04`) |
| **`30041` – `30042`** | `Temp_Busbar` | Nhiệt độ thanh cái đồng / đầu cốt tủ điện (NTC/PT100) | `Float32 (ABCD)` | `°C` | Read Only (`FC 04`) |

> 📖 *Tài liệu Quản lý năng lượng EMS & Kiểm kê khí nhà kính:*  
> - [Giải pháp Công tơ điện thông minh & Giám sát tủ điện phân phối](https://devicelab.vn/cong-to-dien-thong-minh/)  
> - [Kiểm kê khí nhà kính và đo đếm điện năng tự động theo QĐ 42/2026/QĐ-TTg](https://devicelab.vn/kien-thuc/kiem-ke-khi-nha-kinh-do-dem-dien-nang-tu-dong-qd-42-2026/)  
> - [Cách đo đếm điện năng và nhiệt độ tủ điện máy nén khí trục vít chuẩn EMS](https://devicelab.vn/kien-thuc/do-dem-dien-nang-va-nhiet-do-tu-dien-may-nen-khi-truc-vit/)

---

## 📞 Liên Hệ Tư Vấn Tích Hợp & Đặt Hàng Thiết Bị OEM

- **Đơn vị sản xuất:** CÔNG TY TNHH ĐẦU TƯ CÔNG NGHỆ ALPHA TECH (**[DeviceLab](https://devicelab.vn/)**) - MST: `0111171963`
- **Địa chỉ R&D Lab:** Số 32/4, Hà Trì 4, Phường Hà Đông, TP. Hà Nội, Việt Nam
- **Hotline / Zalo Kỹ thuật:** `0982 503 355` | **Email:** `contact@devicelab.vn` / `hi@devicelab.vn`
- **Website (EN):** [https://devicelab.vn/en/products/dl-m400/](https://devicelab.vn/en/products/dl-m400/) · [https://devicelab.vn/en/products/dl-p300/](https://devicelab.vn/en/products/dl-p300/)

---

## Industrial IoT Hardware, Telemetry & Custom R&D Reference Index (DeviceLab Vietnam)

Official open engineering reference hubs, hardware datasheets, and compliance application notes maintained by **DeviceLab R&D Center** (Hanoi, Vietnam):

### 1. Industrial RS485, Modbus RTU/TCP & Edge Gateways
- [Bộ chuyển đổi RS485 sang Ethernet công nghiệp (Cách ly 3.0kV & Modbus RTU/TCP)](https://devicelab.vn/kien-thuc/bo-chuyen-doi-rs485-sang-ethernet/)
- [Thiết bị chuyển đổi tín hiệu RS485 cách ly DL-RS485 (Datasheet & Sơ đồ đấu nối)](https://devicelab.vn/san-pham/dl-rs485/)
- [Cẩm nang chọn Gateway Modbus RS485 sang Ethernet/4G LTE công nghiệp](https://devicelab.vn/kien-thuc/gateway-modbus-rs485/)
- [Bộ chuyển đổi Modbus RTU sang Modbus TCP: Kiến trúc Polling & Multi-Master](https://devicelab.vn/kien-thuc/bo-chuyen-doi-modbus-rtu-sang-tcp/)
- [Tiêu chuẩn cáp tín hiệu RS485 chống nhiễu 120 Ohm (22AWG/24AWG STP)](https://devicelab.vn/kien-thuc/cap-tin-hieu-rs485-chong-nhieu-tieu-chuan-chon/)
- [Đối chiếu thay thế Moxa MGate MB3180, Advantech EKI-1221, USR & 3onedata](https://devicelab.vn/kien-thuc/so-sanh-thay-the-modbus-gateway-moxa-mgate-advantech-usr-3onedata/)
- [Bộ chuyển đổi 4-20mA Analog sang RS485 Modbus & Module Remote I/O](https://devicelab.vn/kien-thuc/bo-chuyen-doi-4-20ma-analog-sang-rs485-remote-io-modbus/)
- [Industrial RS485 to Ethernet Converter Selection Guide (English)](https://devicelab.vn/en/articles/rs485-to-ethernet-converter-selection-guide/)

### 2. Circular 10/2021/TT-BTNMT Environmental Telemetry & SCADA RTU
- [Giải pháp Datalogger quan trắc môi trường tự động chuẩn Thông tư 10/2021/TT-BTNMT](https://devicelab.vn/datalogger-quan-trac-moi-truong-tt10/)
- [Kiến trúc trạm quan trắc nước thải tự động, liên tục & kết nối FTP Sở TN&MT](https://devicelab.vn/kien-thuc/datalogger-quan-trac-nuoc-thai-tu-dong/)
- [Thiết bị Datalogger quan trắc môi trường DL-ENV10 (Datasheet PDF)](https://devicelab.vn/san-pham/dl-env10/)
- [Quy chuẩn nước thải công nghiệp QCVN 40:2025/BTNMT & lộ trình quan trắc tự động](https://devicelab.vn/kien-thuc/quy-chuan-nuoc-thai-cong-nghiep-qcvn-40-quan-trac-tu-dong/)
- [Data logger nhiệt độ đa kênh PT100 (RTD 3 dây) & bộ ghi nhiệt độ công nghiệp](https://devicelab.vn/kien-thuc/data-logger-nhiet-do-da-kenh-pt100-bo-ghi-nhiet-do-cong-nghiep/)
- [Tủ RTU & Thiết bị Remote Terminal Unit SCADA cho nhà máy nước, thủy điện](https://devicelab.vn/kien-thuc/tu-rtu-thiet-bi-remote-terminal-unit-scada-nha-may-nuoc-thuy-dien/)
- [Hệ thống quan trắc nước ngầm tự động giếng khoan chuẩn Nghị định 53/2024/NĐ-CP](https://devicelab.vn/kien-thuc/he-thong-quan-trac-nuoc-ngam-tu-dong/)
- [Giải pháp thay thế Datalogger Campbell Scientific CR1000X / CR300 OEM Việt Nam](https://devicelab.vn/kien-thuc/giai-phap-thay-the-datalogger-campbell-cr1000x-cr300-oem-viet-nam/)
- [Circular 10 Environmental Monitoring Datalogger Solution (English)](https://devicelab.vn/en/environmental-monitoring-datalogger/)

### 3. 3-Phase Power Metering, Solar Zero Export & PLC OEE Telemetry
- [Hệ thống giám sát điện năng nhà máy 3 pha & quản lý năng lượng EMS ISO 50001](https://devicelab.vn/giam-sat-dien-nang-nha-may/)
- [Hướng dẫn chọn đồng hồ đo điện 3 pha RS485 Modbus & thanh ghi chuẩn](https://devicelab.vn/kien-thuc/dong-ho-do-dien-3-pha-rs485-giam-sat-dien-nang/)
- [Bộ điều khiển công suất phát PPC, SmartLogger 3000 & Zero Export SCADA EVN](https://devicelab.vn/kien-thuc/bo-dieu-khien-cong-suat-phat-ppc-smartlogger-3000-zero-export-evn/)
- [Thiết bị thu thập dữ liệu PLC (Siemens, Mitsubishi, Delta) & IoT Gateway](https://devicelab.vn/kien-thuc/thiet-bi-thu-thap-du-lieu-plc-iot-gateway/)
- [Hệ thống giám sát máy móc sản xuất Andon OEE thời gian thực](https://devicelab.vn/kien-thuc/he-thong-giam-sat-may-moc-san-xuat-andon-oee/)

### 4. Custom PCB Design, Turnkey PCBA SMT & Embedded Firmware R&D (Hanoi, Vietnam)
- [Dịch vụ thiết kế mạch điện tử theo yêu cầu & phát triển phần cứng công nghiệp](https://devicelab.vn/thiet-ke-mach-dien-tu/)
- [Công ty thiết kế mạch điện tử tại Hà Nội: Năng lực Lab R&D, EMC & Sản xuất OEM](https://devicelab.vn/kien-thuc/cong-ty-thiet-ke-mach-dien-tu-ha-noi/)
- [Dịch vụ thiết kế PCB 2-8 lớp & vẽ mạch nguyên lý chuẩn IPC-2221](https://devicelab.vn/kien-thuc/dich-vu-thiet-ke-pcb/)
- [Quy trình đặt làm mạch điện tử theo yêu cầu từ EVT/DVT đến sản xuất hàng loạt](https://devicelab.vn/kien-thuc/lam-mach-dien-tu-theo-yeu-cau/)
- [Gia công PCB & hàn mạch điện tử SMT (PCBA) trọn gói tại Việt Nam](https://devicelab.vn/gia-cong-pcb/)
- [Dịch vụ lập trình Firmware nhúng (STM32, ESP32, FreeRTOS, Embedded Linux)](https://devicelab.vn/phat-trien-firmware/)
- [Công ty lập trình nhúng, Firmware & IoT công nghiệp tại Việt Nam](https://devicelab.vn/kien-thuc/cong-ty-lap-trinh-nhung-firmware-iot/)
- [Custom PCB Design & Turnkey Electronics Engineering in Vietnam (English)](https://devicelab.vn/en/custom-pcb-design/)

### 5. Community Engineering Publications (Viblo, Dev.to & Tinh Tế)
- [Viblo: Kỹ thuật chống nhiễu RS485, tính toán Bias & cách ly 2500Vrms](https://viblo.asia/p/ky-thuat-chong-nhieu-rs485-trong-nha-may-tinh-toan-dien-tro-pull-uppull-down-va-cach-ly-quang-2500vrms-bA46807wLKv)
- [Viblo: Kiến trúc Store-and-Forward & Ring-Buffer Flash cho Datalogger Thông tư 10](https://viblo.asia/p/kien-truc-store-and-forward-cho-datalogger-quan-trac-moi-truong-thong-tu-102021tt-btnmt-chong-mat-du-lieu-khi-dut-cap-4g-kNLr3eobVgA)
- [Viblo: Đọc đồng hồ điện 3 pha Selec, Schneider, Eastron qua RS485 & MQTT](https://viblo.asia/p/doc-dong-ho-dien-3-pha-selec-schneider-eastron-qua-modbus-rtu-va-day-mqtt-len-cloud-bang-python-c-OXLA0WqvJGr)
- [Viblo: Checklist 12 điểm kiểm tra Schematic & Layout PCB công nghiệp vượt chuẩn EMC](https://viblo.asia/p/checklist-12-diem-kiem-tra-schematic-layout-pcb-cong-nghiep-truoc-khi-gui-fab-de-vuot-bai-test-emc-iec-61000-4-wd43EXawLX9)
- [Dev.to: Designing Industrial RS485 Circuits for High-EMI Factory Environments](https://dev.to/devicelabvn/designing-industrial-rs485-circuits-for-high-emi-factory-environments-2500vrms-isolation-600w-2114)
- [Dev.to: Store-and-Forward Telemetry Architecture for Environmental Monitoring Dataloggers](https://dev.to/devicelabvn/store-and-forward-telemetry-architecture-for-environmental-monitoring-dataloggers-circular-4b2i)
