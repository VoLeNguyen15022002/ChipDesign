# GIÁO ÁN TỰ HỌC THIẾT KẾ VI MẠCH (VLSI/IC DESIGN) 10 TUẦN
**Đối tượng:** Kỹ sư đã có nền tảng Hệ thống nhúng (Embedded Systems)  
**Mục tiêu:** Chuyển đổi tư duy từ phần mềm nhúng (chạy tuần tự) sang phần cứng vi mạch (chạy song song), làm chủ ngôn ngữ Verilog/SystemVerilog, nắm vững quy trình ASIC Design Flow, và tự thiết kế các khối ngoại vi (UART, SPI) cùng lõi CPU RISC-V cơ bản.

---

## 📅 LỘ TRÌNH TỔNG QUAN

- **Tuần 1 - 2:** Điện tử số & Timing (Setup, Hold, Metastability)
- **Tuần 3 - 4:** Cú pháp Verilog, tư duy thiết kế phần cứng & Thiết kế FSM
- **Tuần 5 - 6:** Thiết kế ngoại vi nhúng UART, SPI & Timer
- **Tuần 7 - 8:** Mô phỏng nâng cao (Icarus/GTKWave) & Tổng hợp mạch (Yosys Synthesis)
- **Tuần 9 - 10:** Kiến trúc CPU RISC-V & Thiết kế Lõi CPU RISC-V (Capstone Project)

---

## 📚 GIAI ĐOẠN 1: NỀN TẢNG KỸ THUẬT SỐ & TIMING (TUẦN 1 - TUẦN 2)

### Tuần 1: Điện tử số nâng cao dưới góc nhìn Vi mạch
*   **Mục tiêu:** Ôn tập các cấu trúc mạch tổ hợp và mạch tuần tự, hiểu rõ cách chuyển đổi logic từ sơ đồ nguyên lý sang phần cứng thực tế.
*   **Kiến thức trọng tâm:**
    *   Mạch tổ hợp: Multiplexer (MUX), Demultiplexer, Decoder, Encoder, Priority Encoder.
    *   Mạch tuần tự: Sự khác biệt bản chất giữa Latch (nhạy theo mức - level-sensitive) và Flip-Flop (nhạy theo sườn - edge-triggered).
    *   Máy trạng thái hữu hạn (FSM): Mô hình Mealy (ngõ ra phụ thuộc trạng thái hiện tại và ngõ vào) vs Moore (ngõ ra chỉ phụ thuộc trạng thái hiện tại).
*   **Tài liệu tham khảo:**
    *   *Digital Design and Computer Architecture* (Harris & Harris) - Chương 1, 2 & 3.
    *   Khóa học miễn phí: *Neso Academy - Digital Electronics* (YouTube).
*   **Bài tập thực hành:**
    *   Vẽ tay sơ đồ mạch logic của một bộ đếm vòng (Ring Counter) 4-bit và bộ chia tần số (Clock Divider) dùng D-FF.
    *   Vẽ sơ đồ bong bóng trạng thái (State Diagram) cho mạch nhận diện chuỗi bit phát hiện chuỗi trùng lặp `1101`.

### Tuần 2: Phân tích Timing - Setup Time, Hold Time & Metastability
*   **Mục tiêu:** Làm chủ khái niệm quan trọng nhất trong thiết kế mạch đồng bộ (Synchronous Design) để chuẩn bị cho phỏng vấn.
*   **Kiến thức trọng tâm:**
    *   **Setup Time ($T_{setup}$):** Thời gian dữ liệu ngõ vào FF phải ổn định *trước* khi sườn xung clock tích cực xuất hiện.
    *   **Hold Time ($T_{hold}$):** Thời gian dữ liệu ngõ vào FF phải giữ nguyên *sau* khi sườn xung clock tích cực xuất hiện.
    *   Công thức tính tần số hoạt động tối đa: $T_{clk} \ge T_{c2q} + T_{comb\_max} + T_{setup}$.
    *   Hiện tượng bất ổn định trạng thái (**Metastability**) và mạch đồng bộ hóa nhiều miền xung nhịp (Clock Domain Crossing - CDC) dùng bộ đồng bộ 2 tầng Flip-Flop (2-stage synchronizer).
*   **Tài liệu tham khảo:**
    *   *Static Timing Analysis for Nanometer Designs* (J. Bhasker).
    *   Các bài viết chuyên sâu về CDC trên trang *Sunburst Design* (Cliff Cummings).
*   **Bài tập thực hành:**
    *   Giải 10 bài tập phân tích Setup/Hold time khi có độ lệch pha xung clock (Clock Skew).
    *   Vẽ mạch đồng bộ hóa tín hiệu 1-bit đi từ miền xung nhịp nhanh sang miền xung nhịp chậm.

---

## 💻 GIAI ĐOẠN 2: LÀM CHỦ VERILOG HDL & NGOẠI VI NHÚNG (TUẦN 3 - TUẦN 6)

### Tuần 3: Tư duy thiết kế phần cứng với Verilog
*   **Mục tiêu:** Làm quen với cú pháp Verilog và phân biệt rạch ròi giữa lập trình phần mềm C (nhúng) và mô tả phần cứng.
*   **Kiến thức trọng tâm:**
    *   Cấu trúc Module, cổng I/O (`input`, `output`, `inout`), kiểu dữ liệu (`reg` vs `wire`).
    *   Khối lệnh `always @(*)` (cho mạch tổ hợp) và `always @(posedge clk)` (cho mạch tuần tự).
    *   **Quy tắc tối thượng:** Sử dụng phép gán nghẽn `blocking (=)` cho mạch tổ hợp, và phép gán không nghẽn `non-blocking (<=)` cho mạch tuần tự.
    *   Nhận diện các lỗi thiết kế phổ biến: Tự động suy luận ra Latch ngoài ý muốn (Inferred Latch) do thiếu nhánh `default` hoặc `else`.
*   **Tài liệu tham khảo:**
    *   *Verilog HDL* (Samir Palnitkar).
    *   Trang web luyện tập tương tác trực tuyến: [HDLBits](https://hdlbits.01xz.net/). (Yêu cầu hoàn thành tối thiểu 50 bài đầu tiên).
*   **Bài tập thực hành:**
    *   Viết code Verilog cho mạch ALU 8-bit hỗ trợ các phép toán: Cộng, Trừ, AND, OR, XOR, Dịch trái, Dịch phải.
    *   Viết Testbench cơ bản để cấp xung clock, reset và kiểm thử chức năng của bộ ALU này.

### Tuần 4: Viết mã nguồn mô tả FSM trong thực tế
*   **Mục tiêu:** Học cách viết máy trạng thái tối ưu, dễ hiểu và có thể tổng hợp mạch (synthesizable).
*   **Kiến thức trọng tâm:**
    *   Phương pháp viết FSM 3 khối (3-always block): Khối 1 cập nhật trạng thái hiện tại (tuần tự), Khối 2 tính toán trạng thái tiếp theo (tổ hợp), Khối 3 điều khiển ngõ ra (tổ hợp/tuần tự).
    *   Kỹ thuật mã hóa trạng thái: Binary Encoding vs One-hot Encoding (tối ưu tốc độ cho FPGA/ASIC).
*   **Tài liệu tham khảo:**
    *   *Synthesizable FSM Design Techniques Using SystemVerilog* (Cliff Cummings).
*   **Bài tập thực hành:**
    *   Thiết kế mạch điều khiển đèn giao thông tại ngã tư (Traffic Light Controller) có cảm biến phát hiện xe trên đường phụ. Viết mã Verilog hoàn chỉnh và testbench mô phỏng.

### Tuần 5: Thiết kế ngoại vi nhúng UART trên Silicon
*   **Mục tiêu:** Tái thiết kế một ngoại vi nhúng cực kỳ quen thuộc để hiểu cách quản lý luồng dữ liệu song song và nối tiếp.
*   **Kiến thức trọng tâm:**
    *   Kiến trúc khối UART TX (Bộ truyền) và UART RX (Bộ nhận).
    *   Khối tạo tốc độ Baud (Baud Rate Generator) bằng bộ chia tần số linh hoạt.
    *   Kỹ thuật lấy mẫu (Oversampling) gấp 16 lần tần số Baud để giảm thiểu nhiễu và bắt sườn dữ liệu chính xác ở khối nhận RX.
*   **Bài tập thực hành:**
    *   **Project 1:** Hiện thực hóa module `uart_tx.v` và `uart_rx.v` hỗ trợ khung truyền 8-N-1 (8-bit data, No parity, 1-stop bit).
    *   Viết Testbench mô phỏng kết nối TX trực tiếp vào RX (Loopback test), truyền thử chuỗi ký tự "HELLO VLSI" và quan sát dạng sóng trên GTKWave.

### Tuần 6: Thiết kế ngoại vi SPI Master & Bộ đếm Timer
*   **Mục tiêu:** Xử lý giao tiếp đồng bộ nhiều thiết bị và quản lý phân chia thời gian.
*   **Kiến thức trọng tâm:**
    *   Bốn chế độ hoạt động của SPI (CPOL và CPHA).
    *   Thanh ghi dịch dữ liệu (Shift Register) hai chiều.
    *   Thiết kế hệ thống phân cấp: Ghép nối các module con UART, SPI thành một cụm IP điều khiển.
*   **Bài tập thực hành:**
    *   **Project 2:** Viết module `spi_master.v`. Thực hiện truyền dữ liệu 8-bit ra ngoại vi.
    *   Tạo một sơ đồ thiết kế vi mô (Microarchitecture) cho khối Timer 32-bit hỗ trợ ngắt tự động nạp lại (Auto-reload).

---

## 🔬 GIAI ĐOẠN 3: EDA TOOLCHAIN, SYNTHESIS & RISC-V DESIGN (TUẦN 7 - TUẦN 10)

### Tuần 7: Mô phỏng chức năng nâng cao & Giới thiệu EDA Tools
*   **Mục tiêu:** Chuyển đổi từ mô phỏng lý thuyết sang việc sử dụng các trình biên dịch công nghiệp hoặc mã nguồn mở.
*   **Kiến thức trọng tâm:**
    *   Cài đặt và sử dụng toolchain mô phỏng mã nguồn mở: **Icarus Verilog (iverilog)** kết hợp trình xem dạng sóng **GTKWave**.
    *   Tìm hiểu về SystemVerilog Testbench: Cấu trúc hướng đối tượng (OOP), khái niệm Interface, Program Block.
    *   Giới thiệu các trình mô phỏng công nghiệp: ModelSim/QuestaSim (Siemens), VCS (Synopsys).
*   **Tài liệu hướng dẫn thực hành:**
    *   Sử dụng lệnh dòng lệnh (Command line) để biên dịch:
        ```bash
        # Biên dịch code thiết kế và testbench
        iverilog -o sim.out design.v testbench.v
        # Chạy mô phỏng để xuất file sóng .vcd
        vvp sim.out
        # Mở xem dạng sóng trực quan
        gtkwave waveform.vcd
        ```

### Tuần 8: Logic Synthesis (Tổng hợp mạch) sang Gate-level Netlist
*   **Mục tiêu:** Nhìn thấy mạch logic thực tế được tạo ra từ dòng code bạn viết, hiểu khái niệm thư viện công nghệ (Standard Cell Library).
*   **Kiến thức trọng tâm:**
    *   Quy trình biên dịch code RTL thành danh sách các cổng logic (Netlist).
    *   Sử dụng công cụ tổng hợp mã nguồn mở **Yosys**.
    *   Phân tích báo cáo tổng hợp: Area (Diện tích mạch), Timing slack (Thời gian dư thừa), Power (Công suất tiêu thụ ước tính).
*   **Tài liệu hướng dẫn thực hành:**
    *   Cài đặt Yosys và viết mã lệnh script tổng hợp cơ bản:
        ```tcl
        # Đọc file verilog
        read_verilog alu.v
        # Tổng hợp logic tổng quát
        synth -top alu
        # Ánh xạ mạch sang các cổng cơ bản logic
        show -format png -prefix alu_circuit
        ```

### Tuần 9: Tìm hiểu Kiến trúc CPU RISC-V (RV32I)
*   **Mục tiêu:** Hiểu rõ cấu trúc tập lệnh và thiết kế vi kiến trúc của một bộ vi xử lý thực tế.
*   **Kiến thức trọng tâm:**
    *   Tập lệnh tối giản RV32I của RISC-V: Định dạng lệnh R-type, I-type, S-type, B-type, U-type, J-type.
    *   Các khối chức năng trong CPU Đơn chu kỳ (Single-Cycle CPU):
        *   Program Counter (PC) & PC Increment.
        *   Instruction Memory (ROM nạp lệnh).
        *   Register File (Tập thanh ghi gồm 32 thanh ghi 32-bit).
        *   ALU & Control Unit (Khối điều khiển trung tâm giải mã lệnh).
        *   Data Memory (RAM chứa dữ liệu).
*   **Tài liệu tham khảo:**
    *   *Computer Organization and Design RISC-V Edition* (Patterson & Hennessy).
*   **Bài tập thực hành:**
    *   Vẽ sơ đồ khối dòng dữ liệu (Datapath diagram) cho CPU RISC-V đơn chu kỳ khi thực thi hai lệnh: Lệnh cộng `ADD x1, x2, x3` và lệnh tải dữ liệu `LW x4, 4(x5)`.

### Tuần 10: Hiện thực hóa Core CPU RISC-V (Capstone Project)
*   **Mục tiêu:** Ghép nối tất cả kiến thức đã học trong 10 tuần để tạo ra một lõi CPU RISC-V đơn chu kỳ hoạt động được.
*   **Kiến thức trọng tâm:**
    *   Hiện thực hóa các cấu trúc Module con của RISC-V bằng Verilog.
    *   Tích hợp hệ thống (System Integration): Viết module cha `riscv_core.v` nối các khối chức năng lại với nhau.
    *   Kiểm thử CPU bằng việc nạp một chương trình viết bằng hợp ngữ (Assembly) đơn giản (Ví dụ: chương trình tính dãy số Fibonacci).
*   **Sản phẩm cuối khóa:** 
    *   Một lõi CPU RISC-V (RV32I Subset) hoàn chỉnh trên GitHub cá nhân kèm tài liệu thiết kế vi kiến trúc chi tiết.

---

## 🛠️ PHẦN PHỤ LỤC: MÃ NGUỒN MẪU & BẮT ĐẦU NHANH

### Mẫu thiết kế FSM 3 khối (3-Block FSM) tiêu chuẩn:

```verilog
module fsm_template (
    input  wire clk,
    input  wire rst_n,
    input  wire in_bit,
    output reg  out_detect
);

    // Định nghĩa các trạng thái sử dụng Parameter
    localparam STATE_IDLE = 2'b00;
    localparam STATE_A    = 2'b01;
    localparam STATE_B    = 2'b10;

    reg [1:0] current_state;
    reg [1:0] next_state;

    // Khối 1: Cập nhật trạng thái (Mạch tuần tự)
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            current_state <= STATE_IDLE;
        end else begin
            current_state <= next_state;
        end
    end

    // Khối 2: Logic xác định trạng thái tiếp theo (Mạch tổ hợp)
    always @(*) begin
        case (current_state)
            STATE_IDLE: begin
                if (in_bit) next_state = STATE_A;
                else        next_state = STATE_IDLE;
            end
            STATE_A: begin
                if (!in_bit) next_state = STATE_B;
                else         next_state = STATE_A;
            end
            STATE_B: begin
                if (in_bit) next_state = STATE_IDLE;
                else        next_state = STATE_A;
            end
            default: next_state = STATE_IDLE;
        endcase
    end

    // Khối 3: Logic xác định ngõ ra (Mạch tổ hợp)
    always @(*) begin
        case (current_state)
            STATE_IDLE: out_detect = 1'b0;
            STATE_A:    out_detect = 1'b0;
            STATE_B:    out_detect = 1'b1; // Phát hiện chuỗi
            default:    out_detect = 1'b0;
        endcase
    end

endmodule
```
