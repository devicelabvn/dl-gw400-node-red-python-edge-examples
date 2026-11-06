<p align="left">
  <a href="https://devicelab.vn/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-dark.svg?v=3.45.0">
      <source media="(prefers-color-scheme: light)" srcset="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-light.svg?v=3.45.0">
      <img alt="DeviceLab - Industrial IoT Hardware & Embedded R&D" src="https://devicelab.vn/wp-content/themes/devicelab/assets/img/logo-devicelab-on-light.svg?v=3.45.0" height="38">
    </picture>
  </a>
</p>

# DL-GW400 4-Port Isolated RS485 Edge Gateway & DL-RS485 Serial Server - Node-RED, Python & Modbus TCP Examples 🖧

> **Tài liệu kỹ thuật, kịch bản Python đọc đa luồng 4 cổng RS485 cách ly độc lập (gom 128 thiết bị Modbus), luồng mẫu Node-RED (`flows.json`) trên Edge Gateway Linux `DL-GW400` và hướng dẫn cấu hình Modbus TCP ↔ RTU trên bộ chuyển đổi `DL-RS485`.**  
> Phát triển và sản xuất bởi **[DeviceLab](https://devicelab.vn/)** (CÔNG TY TNHH ĐẦU TƯ CÔNG NGHỆ ALPHA TECH - MST: `0111171963`).  
> 🔗 **Edge Gateway 4 Cổng RS485:** [Bộ Edge Gateway 4 Cổng RS485 Cách Ly DL-GW400](https://devicelab.vn/san-pham/dl-gw400/) ([English Version](https://devicelab.vn/en/products/dl-gw400/))  
> 🔗 **Bộ Chuyển Đổi RS485 Cách Ly:** [Bộ Chuyển Đổi RS485 Sang Ethernet Công Nghiệp DL-RS485](https://devicelab.vn/san-pham/dl-rs485/) ([English Version](https://devicelab.vn/en/products/dl-rs485/))  
> 🔗 **Giải Pháp Cho Nhà Tích Hợp (SI/M&E):** [Gateway OEM Cho Nhà Tích Hợp Hệ Thống](https://devicelab.vn/gateway-oem-cho-nha-tich-hop/)

---

## 📌 1. Khi Nào Chọn `DL-RS485`, `DL-GW200` Hay `DL-GW400` Cho Tủ Điện Dự Án?

Bảng đối chiếu nhanh 3 dòng thiết bị truyền thông công nghiệp chủ lực của **DeviceLab** để kỹ sư M&E / SCADA lựa chọn chính xác cho hồ sơ thầu (BOQ):

| Tiêu chí kỹ thuật | **[DeviceLab DL-RS485](https://devicelab.vn/san-pham/dl-rs485/)** | **[DeviceLab DL-GW200](https://devicelab.vn/san-pham/dl-gw200/)** | **[DeviceLab DL-GW400](https://devicelab.vn/san-pham/dl-gw400/)** |
| :--- | :--- | :--- | :--- |
| **Phân khúc thiết bị** | Industrial RS485-to-Ethernet Server & Modbus Gateway | Industrial IoT Edge Gateway (RS485 sang MQTT/Cloud) | Multi-Port Linux Edge Computing Gateway (Tủ MSB / BMS lớn) |
| **Số cổng RS485 cách ly 3.0kV** | **1 cổng RS485** (tối đa 32 Slave) | **1–2 cổng RS485** (tối đa 32–64 Slave) | **4 cổng RS485 độc lập** (gom tới **128 Slave** đồng thời) |
| **Cổng mạng Ethernet / Di động** | 1x RJ45 10/100Mbps | 1x RJ45 (hỗ trợ PoE) + Wi-Fi + Tùy chọn 4G LTE | **2x Gigabit RJ45 độc lập (Dual Subnet OT/IT)** + Tùy chọn 4G LTE |
| **Chế độ hoạt động chính** | Modbus RTU ↔ Modbus TCP (6 TCP Masters), TCP Server/Client, Virtual COM | Modbus Multi-Master Caching (`<2ms`), JSON MQTT/MQTTS, REST API | Hệ điều hành **Linux nhúng**, chạy **Node-RED, Python, Docker**, BACnet IP, OPC UA, MQTT |
| **Bộ nhớ đệm Offline** | RAM Queue chống xung đột lệnh | **Flash 100.000 bản ghi** Store & Forward | **8GB eMMC** (lưu cơ sở dữ liệu SQLite / InfluxDB hàng triệu bản ghi tại biên) |
| **Nguồn cấp công nghiệp** | `9–36VDC` (Domino) | `9–36VDC` + `PoE 802.3af` | **Nguồn đôi dự phòng Redundant `9–48VDC`** (`V1+`/`V1-` & `V2+`/`V2-`) |

---

## ⚡ 2. Kiến Trúc Đọc Song Song 4 Nhánh Bus RS485 Không Nghẽn Trên `DL-GW400`

Trong các tủ điện tổng (`MSB`) có từ `60` đến `120` đồng hồ đo điện hoặc hệ thống BMS tòa nhà kết nối đồng thời Chiller, AHU, biến tần và đồng hồ nước:
- Nếu mắc nối tiếp toàn bộ thiết bị trên 1 đường dây RS485 duy nhất (tốc độ `9600 bps`), thời gian quét hết 1 vòng (`Polling Cycle`) sẽ mất từ `15–30 giây`, và chỉ cần 1 thiết bị chập chân `A/B` là toàn bộ hệ thống mất kết nối.
- Trên **[Edge Gateway DL-GW400](https://devicelab.vn/san-pham/dl-gw400/)**, 4 cổng RS485 (`/dev/ttyS1`, `/dev/ttyS2`, `/dev/ttyS3`, `/dev/ttyS4`) có **IC cách ly quang 3.0kV, nguồn cách ly DC-DC và bộ đệm UART phần cứng hoàn toàn tách biệt**:

```text
[DL-GW400 Quad-Core Linux Edge Gateway]
  ├── Cổng RS485-1 (/dev/ttyS1 - 9600 8N1)   ---> 30 Đồng hồ điện nhánh Sản xuất 1
  ├── Cổng RS485-2 (/dev/ttyS2 - 9600 8E1)   ---> 30 Đồng hồ điện nhánh Sản xuất 2
  ├── Cổng RS485-3 (/dev/ttyS3 - 19200 8N1)  ---> 25 Biến tần INVT / Delta (Tủ bơm & quạt)
  └── Cổng RS485-4 (/dev/ttyS4 - 38400 8N1)  ---> Cụm PLC Siemens / Mitsubishi & Cảm biến nhiệt ẩm
  │
  ├── ETH1 (Mạng OT nội bộ: 192.168.10.x)    ---> Phục vụ SCADA / HMI tại phòng điều khiển nhà máy
  └── ETH2 (Mạng IT / Internet: 10.0.20.x)   ---> Đẩy dữ liệu đã lọc qua MQTTS (TLS 1.3) lên Cloud EMS
```

> 📂 Xem code mẫu Python đọc song song 4 cổng RS485 (`asyncio`) tại: [`examples/dl_gw400_multibus_poller.py`](./examples/dl_gw400_multibus_poller.py)  
> 📂 Xem file cấu hình luồng Node-RED mẫu tại: [`node-red/dl_gw400_modbus_to_mqtt_flow.json`](./node-red/dl_gw400_modbus_to_mqtt_flow.json)

---

## 🛡️ 3. Chế Độ Multi-Master Modbus TCP ↔ RTU & Watchdog Kép Trên `DL-RS485`

Đối với các dự án cần kéo đường truyền RS485 từ các tủ điện phân tán về máy chủ SCADA trung tâm qua hạ tầng mạng LAN nội bộ, **[Bộ chuyển đổi RS485 sang Ethernet DL-RS485](https://devicelab.vn/san-pham/dl-rs485/)** thay thế hoàn hảo các dòng Moxa NPort 5150 / USR-TCP232 ở cấp độ công nghiệp nặng:

1. **Cách ly quang 3.0kVAC & Chống sét TVS 600W:** Ngăn chặn 100% hiện tượng sốc áp đất làm cháy cổng LAN của Switch hoặc PLC trung tâm.
2. **Hỗ trợ đồng thời 6 kết nối Modbus TCP Master:** Cho phép cả phần mềm SCADA chính, máy chủ dự phòng (Redundant Server) và màn hình HMI cùng truy vấn `Port 502` mà không bị treo socket.
3. **Dual Watchdog (Phần cứng + Phần mềm):** Tự động giải phóng socket TCP bị kẹt (`Half-open TCP connection`) khi cáp mạng bị rút đột ngột và tự phục hồi trong `0.5 giây`.

---

## 📖 Tài Liệu Hướng Dẫn Kỹ Thuật Liên Quan Trên `devicelab.vn`

- [Top 5 Bộ chuyển đổi RS485 sang Ethernet công nghiệp đáng mua nhất 2026](https://devicelab.vn/kien-thuc/top-bo-chuyen-doi-rs485-sang-ethernet-cong-nghiep/)
- [Báo giá Bộ chuyển đổi RS485 sang Ethernet công nghiệp & Dự toán BOQ M&E](https://devicelab.vn/kien-thuc/gia-bo-chuyen-doi-rs485-sang-ethernet/)
- [Bộ chuyển đổi RS485 sang Ethernet có bộ đệm dữ liệu (Buffer)](https://devicelab.vn/kien-thuc/bo-chuyen-doi-rs485-sang-ethernet-co-bo-dem-du-lieu/)
- [Tiêu chí chọn bộ chuyển đổi RS485 sang Ethernet gắn thanh ray DIN-rail tủ điện](https://devicelab.vn/kien-thuc/bo-chuyen-doi-rs485-sang-ethernet-din-rail/)
- [So sánh Gateway COTS (Moxa/USR) và Gateway thiết kế tùy chỉnh OEM](https://devicelab.vn/kien-thuc/so-sanh-gateway-cots-moxa-usr-va-gateway-thiet-ke-tuy-chinh/)

### 🌐 Đọc bài phân tích chi tiết tiếng Việt trên Viblo & Tinh Tế:
- **[Viblo.asia]** [Phân biệt Transparent Converter và Modbus Gateway: Cách xử lý nhiễu biến tần và lưu đệm Flash khi rớt mạng 4G](https://viblo.asia/p/phan-biet-transparent-converter-va-modbus-gateway-cach-xu-ly-nhieu-bien-tan-va-luu-dem-flash-khi-rot-mang-4g-bA46807wLKv)
- **[Viblo.asia]** [Từ bo mạch mẫu (Prototype) đến sản xuất 1.000 thiết bị IoT công nghiệp: Checklist thiết kế PCB chuẩn DFM & chống nhiễu EMC](https://viblo.asia/p/tu-bo-mach-mau-prototype-den-san-xuat-1000-thiet-bi-iot-cong-nghiep-checklist-thiet-ke-pcb-chuan-dfm-chong-nhieu-emc-OXLA0WqvJGr)
- **[Tinh Tế (`tinhte.vn`)]** [Từ bo mạch mẫu (Prototype) đến sản xuất 1.000 thiết bị IoT: Checklist thiết kế PCB & EMC](https://tinhte.vn/thread/tu-bo-mach-mau-prototype-den-san-xuat-1-000-thiet-bi-iot-checklist-thiet-ke-pcb-emc.4185069/)
- **[Viblo.asia]** [Kinh nghiệm xử lý mất dữ liệu trạm quan trắc & Quy chuẩn phần cứng Datalogger theo Điều 39 Thông tư 10/2021/TT-BTNMT](https://viblo.asia/p/kinh-nghiem-xu-ly-mat-du-lieu-tram-quan-trac-quy-chuan-phan-cung-datalogger-theo-dieu-39-thong-tu-102021tt-btnmt-kNLr3eobVgA)
- **[Tinh Tế (`tinhte.vn`)]** [Kinh nghiệm xử lý mất dữ liệu trạm quan trắc & chuẩn phần cứng Datalogger Thông tư 10](https://tinhte.vn/thread/kinh-nghiem-xu-ly-mat-du-lieu-tram-quan-trac-chuan-phan-cung-datalogger-thong-tu-10.4185068/)

---

## 📞 Liên Hệ Kỹ Sư Dự Án & Nhận Báo Giá Kỹ Thuật

- **Đơn vị sản xuất:** CÔNG TY TNHH ĐẦU TƯ CÔNG NGHỆ ALPHA TECH (**[DeviceLab](https://devicelab.vn/)**) - MST: `0111171963`
- **Địa chỉ R&D Lab:** Số 32/4, Hà Trì 4, Phường Hà Đông, TP. Hà Nội, Việt Nam
- **Hotline / Zalo Kỹ thuật:** `0982 503 355` | **Email:** `contact@devicelab.vn` / `hi@devicelab.vn`
- **Yêu cầu báo giá dự án M&E / SI:** [https://devicelab.vn/san-pham/dl-gw400/#bao-gia-du-an](https://devicelab.vn/san-pham/dl-gw400/#bao-gia-du-an)

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
