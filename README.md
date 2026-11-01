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
