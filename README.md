# Lab5_FAP201 - LFSR Timer & Demultiplexer (Điều hướng tín hiệu nhấp nháy)

## Thông tin chung
* **Môn học:** FAP201 - FPGA Programming
* **Bài thực hành:** Lab05 - Linear-Feedback Shift Register (LFSR) & Demultiplexer[cite: 29]
* **Mục tiêu:** Thiết kế bộ định thời (Timer) sử dụng thanh ghi dịch phản hồi tuyến tính (LFSR) để tạo tín hiệu nhấp nháy, kết hợp mạch giải mã / điều hướng (Demultiplexer) để chọn đèn LED sẽ nhấp nháy dựa trên tổ hợp phím nhấn ngõ vào[cite: 29].

## Môi trường và Phần cứng
* **Công cụ phát triển:** Gowin EDA
* **Bo mạch (Board):** Xiaomeige ACG525
* **Chip FPGA (Target Device):** GW5A-LV25UG324C2/I1
* **Xung nhịp hệ thống (Clock):** 50 MHz (Chu kỳ 20 ns)[cite: 28]
* **Ngôn ngữ:** VHDL (IEEE 1164 STD_LOGIC)[cite: 29]

## Nguyên lý hoạt động & Kiến trúc Thiết kế

### 1. Bộ định thời LFSR (Linear-Feedback Shift Register)
* Thay vì dùng bộ đếm nhị phân thông thường (`unsigned`), mạch sử dụng một LFSR độ rộng 22-bit (`G_WIDTH = 22`) để hoạt động như một bộ chia tần số / định thời[cite: 29].
* Tại mỗi cạnh lên của xung nhịp `i_Clk`, thanh ghi `r_LFSR` sẽ dịch trái 1 bit và bit mới được nạp vào là kết quả của phép toán `XNOR` giữa bit cao nhất (`G_WIDTH-1`) và bit kế kề (`G_WIDTH-2`)[cite: 29].
* Khi toàn bộ thanh ghi LFSR mang giá trị `0`, tín hiệu `w_Done` sẽ được bật lên mức `'1'` để báo hiệu hoàn thành một chu kỳ đếm[cite: 29].

### 2. Mạch tạo nhấp nháy (Toggle Process)
* Tín hiệu `r_Toggle` đóng vai trò lưu trạng thái sáng/tắt của đèn LED[cite: 29].
* Mỗi khi bộ định thời báo đếm xong (`w_Done = '1'`), Flip-Flop `r_Toggle` sẽ tự động lật trạng thái (`not r_Toggle`) để tạo ra một xung nhấp nháy tuần hoàn[cite: 29].

### 3. Mạch điều hướng (Demultiplexer / Decoder)
* Mạch sử dụng 2 switch ngõ vào (`i_Switch_1` và `i_Switch_2`) làm tín hiệu điều khiển bộ giải mã để hướng trạng thái `r_Toggle` ra một trong 4 đèn LED[cite: 29]. 
* Các đèn LED không được chọn sẽ mặc định luôn TẮT (mức `'0'`)[cite: 29]. Bảng chân lý giải mã như sau:
  * `Switch_2 = 0`, `Switch_1 = 0` $\rightarrow$ Nhấp nháy `o_LED_1`[cite: 29]
  * `Switch_2 = 0`, `Switch_1 = 1` $\rightarrow$ Nhấp nháy `o_LED_2`[cite: 29]
  * `Switch_2 = 1`, `Switch_1 = 0` $\rightarrow$ Nhấp nháy `o_LED_3`[cite: 29]
  * `Switch_2 = 1`, `Switch_1 = 1` $\rightarrow$ Nhấp nháy `o_LED_4`[cite: 29]

---

## Cấu hình Chân & Ràng buộc Thời gian (Constraints & Timing)

### 1. Ánh xạ chân vật lý (`lab05.cst`)
Tín hiệu được cấu hình trong file `.cst` với chuẩn I/O `LVCMOS33`[cite: 27]:

| Tín hiệu VHDL | Chân FPGA | Kéo trở (Pull Mode) | Chức năng |
| :--- | :---: | :--- | :--- |
| `i_Clk` | `T9` | `NONE` | Xung nhịp hệ thống[cite: 27] |
| `i_Switch_1` | `B16` | `UP` | Switch điều khiển Bit 0[cite: 27] |
| `i_Switch_2` | `A15` | `UP` | Switch điều khiển Bit 1[cite: 27] |
| `o_LED_1` | `D14` | `NONE` (DRIVE=8) | Đèn LED 1[cite: 27] |
| `o_LED_2` | `C14` | `NONE` (DRIVE=8) | Đèn LED 2[cite: 27] |
| `o_LED_3` | `B9` | `NONE` (DRIVE=8) | Đèn LED 3[cite: 27] |
| `o_LED_4` | `A9` | `NONE` (DRIVE=8) | Đèn LED 4[cite: 27] |

### 2. Ràng buộc thời gian (`lab05.sdc`)
File `.sdc` định nghĩa xung nhịp 50 MHz (Chu kỳ 20 ns) cho công cụ phân tích thời gian STA[cite: 28]:
```tcl
create_clock -name i_Clk -period 20 -waveform {0 10} [get_ports {i_Clk}]
```[cite: 28]

---

## Cấu trúc thư mục dự án (Project Structure)
* `src/lab05.vhd`: Mã nguồn VHDL thiết kế Timer LFSR và hệ thống giải mã LED[cite: 29].
* `constraints/lab05.cst`: File gán chân vật lý Gowin FPGA[cite: 27].
* `constraints/lab05.sdc`: File cấu hình định thì xung nhịp[cite: 28].
* `sim/lab05_tb.vhd`: Testbench giả lập xung clock và các tổ hợp thay đổi 2 phím Switch (Lưu ý: nên giảm `G_WIDTH` trong file TB để rút ngắn thời gian mô phỏng).
* `waveform/`: Ảnh chụp sóng quá trình lật trạng thái của LFSR và LED.
* `reports/`: Báo cáo Tổng hợp (Synthesis), RTL, và định thì.
* `evidence/`: Bằng chứng nạp mạch trên board thật.

---

## Hướng dẫn Build & Kiểm chứng

1. **Mô phỏng (Simulation):**
   * Trong quá trình viết Testbench, bạn có thể chỉnh tham số `generic (G_WIDTH : integer := 4)` để mô phỏng lật trạng thái nhanh hơn.
   * Giả lập cả 4 tổ hợp ngõ vào `00, 01, 10, 11` của Switch 2 và 1, quan sát xem có đúng tín hiệu `r_Toggle` được định tuyến đến LED tương ứng hay không[cite: 29].

2. **Tổng hợp & Phân tích định thì (Synthesis & STA):**
   * Chạy **Synthesis** và **Place & Route** trong Gowin EDA.
   * Đọc báo cáo Timing để đảm bảo bộ đếm LFSR 22-bit đáp ứng tốt tần số hoạt động 50MHz[cite: 28, 29].

3. **Nạp phần cứng (Hardware):**
   * Nạp file Bitstream xuống board ACG525[cite: 27].
   * Thay đổi trạng thái các phím B16, A15 và quan sát vị trí đèn LED đang nhấp nháy sẽ dịch chuyển theo tổ hợp phím bạn chọn[cite: 27, 29].
