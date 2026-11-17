# Nạp & debug STM32 qua Internet không cần thiết bị trung gian

Nov 16, 2026 · @bình

## Tổng quan

Cho chính ESP32-S3 chạy VPN WireGuard: không cần Raspberry Pi hay máy tính đặt tại chỗ. Thư viện WireGuard cho ESP32 đã có sẵn (ví dụ `esp_wireguard` cho ESP-IDF), chỉ cần ghép vào firmware DAP (wireless-esp8266-dap).

Keil và elaphureLink giữ nguyên cách dùng; chỉ đổi IP nhập vào elaphureLink sang IP ảo của ESP32 trong mạng VPN.

&#91;embedded content: Đường đi dữ liệu · Cách 1\]

ESP32 chủ động kết nối ra ngoài qua router, nên router tại chỗ không cần mở cổng; chỉ cần mở 1 cổng UDP ở phía laptop.

## Cách 1 (khuyên dùng): WireGuard trên ESP32, laptop làm server

ESP32 tự kết nối ra Internet về laptop, nên phía đặt mạch không cần mở cổng và không cần thiết bị phụ.

- Hai đầu nói chuyện qua IP ảo: laptop `10.8.0.1`, ESP32 `10.8.0.2`. Trong elaphureLink chỉ nhập `10.8.0.2`.
- Dữ liệu được mã hóa; chỉ thiết bị có khóa mới kết nối được.

**Điều kiện:** phía laptop phải có IP công cộng.

- Router nhà mở 1 cổng UDP (ví dụ `51820`) trỏ về laptop, kèm DDNS (No-IP, DuckDNS) vì IP nhà mạng hay đổi.
- Gói cước dùng CGNAT (không có IP công cộng thật) thì cách này không chạy: xin nhà mạng cấp IP công cộng, hoặc dùng Cách 2.

## Cách 2: WireGuard + VPS làm điểm hẹn

Dùng khi nhà không có IP công cộng. Thuê một VPS giá rẻ (khoảng 3–5 USD/tháng) hoặc gói miễn phí (Oracle Cloud Free Tier) và cài WireGuard server lên đó.

ESP32 và laptop cùng kết nối ra VPS, nên không bên nào phải mở cổng. VPS là máy chủ trên mạng, không phải phần cứng đặt tại chỗ.

## Cách 3 (không khuyên dùng): mở cổng thẳng vào ESP32

Mở cổng (port forward) trên router ở chỗ đặt mạch trỏ thẳng vào ESP32. Chạy được ngay nhưng không an toàn: giao thức DAP không có mật khẩu, ai dò ra cổng là đọc hoặc ghi được flash của STM32. Chỉ dùng tạm để thử rồi tắt ngay.

## So sánh và việc cần chuẩn bị

| Cách | Thiết bị trung gian | Cần IP công cộng | An toàn | Độ khó |
| --- | --- | --- | --- | --- |
| WireGuard trên ESP32, laptop làm server | Không | Có, ở nhà | Cao | Trung bình (sửa firmware) |
| WireGuard trên ESP32 + VPS | Không (chỉ cần VPS) | Không | Cao | Trung bình |
| Mở cổng thẳng | Không | Có, ở chỗ đặt mạch | Thấp | Dễ |

**Tốc độ:** qua Internet chậm hơn mạng LAN. Ping dưới 50 ms (cùng nước) thì nạp và debug vẫn ổn. ESP32-S3 thừa sức xử lý mã hóa WireGuard ở tốc độ SWD cần.

**Cần chuẩn bị để làm firmware:**

- [ ] Nhà có IP công cộng không, hay dùng VPS?
- [ ] Tên và mật khẩu Wi-Fi ở chỗ đặt mạch (băng tần 2.4 GHz)
- [x] Build firmware trên máy có sẵn ESP-IDF
