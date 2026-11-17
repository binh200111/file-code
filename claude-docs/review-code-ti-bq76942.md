# Review code TI\_BQ76942 (STM32 + BQ76942)

Nov 3, 2026 · @bình

## Tóm tắt

Code biên dịch được nhưng có một số lỗi logic. Vài lỗi trong đó ảnh hưởng trực tiếp tới an toàn khi sạc và cân bằng cell.

Phạm vi đã đọc: `main.c`, `lcd_pin9.c`, các file cấu hình CubeMX (`adc/tim/i2c/gpio/it`) và 2 file ghi chú `MANUAL_BALANCE_*.txt`. Các lỗi dưới đây xếp theo mức độ nghiêm trọng, kèm số dòng trong `main.c`.

## Nghiêm trọng (sai chức năng hoặc ảnh hưởng an toàn)

1. **Ghi cấu hình cân bằng khi chưa vào CONFIG\_UPDATE, và ghi lặp lại mỗi 200 ms** (dòng 1027–1043 và 1073–1090)
   - Đoạn này gọi `BQ769x2_SetRegister(...)` rồi `EXIT_CFGUPDATE`, nhưng không gọi `SET_CFGUPDATE` trước. Vì vậy lệnh EXIT không có tác dụng, và các thanh ghi bộ nhớ dữ liệu được ghi ngoài chế độ cấu hình.
   - Nhánh sạc (dòng 1071) lặp lại toàn bộ cụm ghi cấu hình này mỗi 200 ms trong suốt quá trình sạc.
   - Cách sửa: chuyển hết các thanh ghi này (`BalancingConfiguration=0`, `CellBalanceInterval`, `MinCellTemp`, `MaxCellTemp`, `MaxInternalTemp`, `TS3Config`) vào `BQ769x2_Init()`. Trong vòng lặp chính chỉ gửi `Subcommands(CB_ACTIVE_CELLS, mask, W2)`.
2. **Không bao giờ tắt cân bằng khi đã hết lệch** (dòng 1050, 1071)
   - Khi `delta_mv` xuống ≤ 15, code chuyển sang sạc nhưng không gửi `CB_ACTIVE_CELLS = 0`. Cell vừa được chọn có thể vẫn đang xả qua điện trở.
   - Mask chỉ được xoá ở nhánh có cell < 3000 mV.
   - Cần gửi lệnh tắt cân bằng ngay khi delta về dưới ngưỡng.
3. **Xoá sai bit trong AlarmStatus** (dòng 1018)
   - `DirectCommands(AlarmStatus, 256, W)` ghi `0x0100`, tức là bit ADSCAN. Bit FULLSCAN là bit 7, tức là `0x0080`, đúng như phép kiểm tra ở dòng 1007.
   - Hậu quả: FULLSCAN không bao giờ được xoá, nên điều kiện `if (AlarmBits & 0x80)` lúc nào cũng đúng.
   - Sửa thành `DirectCommands(AlarmStatus, 0x0080, W);`.
4. **Không kiểm tra lỗi I²C, nên có thể dùng dữ liệu cũ**
   - `I2C_ReadReg` luôn trả về 0 và bỏ qua kết quả của `HAL_I2C_Mem_Read`.
   - Nếu BQ mất giao tiếp, `RX_data` vẫn giữ giá trị lần đọc trước. `CellVoltage[]` khi đó trông bình thường, và bộ sạc vẫn chạy tiếp.
   - Đề xuất: trả về `HAL_StatusTypeDef`. Khi đọc lỗi thì tắt PWM, bật DFETOFF và xoá `Flag_PID`.
5. **Không có điều kiện kết thúc sạc và không đọc Safety Status**
   - Vòng CV giữ 37,6 V mãi, không có điều kiện dừng khi dòng giảm thấp, ví dụ dưới C/20.
   - Code không dừng khi cell cao nhất (`max_mv`) vượt khoảng 4,2 V, mà chỉ dựa vào ngưỡng COV 4,3 V của BQ.
   - `BQ769x2_ReadSafetyStatus()` và `ReadPFStatus()` không được gọi ở đâu, nên `ProtectionsTriggered` không bao giờ được cập nhật.
6. **Cell thấp làm khoá luôn chức năng sạc** (dòng 981–994)
   - Chỉ cần một cell < 3000 mV là `Flag_Button = 0`, nên pack đã xả sâu (ví dụ 2,9 V/cell) không thể sạc được. Có lẽ bạn muốn dùng ngưỡng này để chặn xả, hoặc để sạc dòng nhỏ (pre-charge), chứ không phải để chặn sạc.
   - Lúc mới khởi động `CellVoltage[]` đều bằng 0. Trong khoảng 200 ms đầu, mỗi vòng lặp gửi tới 9 lệnh I²C `CB_ACTIVE_CELLS = 0`, mỗi lệnh có delay. Chỉ cần gửi một lần, đặt ngoài vòng `for`.
