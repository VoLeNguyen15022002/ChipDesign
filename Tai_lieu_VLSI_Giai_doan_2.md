# TÀI LIỆU CHUYÊN SÂU: GIAI ĐOẠN 2 (TUẦN 3 ĐẾN TUẦN 6)
## LÀM CHỦ VERILOG HDL & NGOẠI VI NHÚNG TRÊN SILICON

---

## 💻 TUẦN 3: TƯ DUY THIẾT KẾ PHẦN CỨNG VỚI VERILOG

### 1. Phân biệt Wire vs. Reg trong phần cứng thực tế

*   **`wire` (Net):**
    *   **Bản chất:** Đại diện cho kết nối vật lý (dây dẫn) giữa các cổng logic hoặc linh kiện. Không có khả năng lưu trữ trạng thái.
    *   **Hành vi:** Giá trị thay đổi ngay lập tức khi ngõ vào điều khiển nó thay đổi. Nếu không được lái (floating), nó sẽ rơi vào trạng thái trở kháng cao (`z`).
    *   **Quy tắc:** Chỉ được gán bằng câu lệnh liên tục `assign` ngoài khối procedural block.
*   **`reg` (Register/Variable):**
    *   **Bản chất:** Biến đại diện cho phần tử lưu trữ dữ liệu.
    *   **Hành vi:** Giữ nguyên giá trị cũ cho đến khi có một phép gán mới thực thi (chạy theo sự kiện).
    *   **Quy tắc:** Bắt buộc phải được gán bên trong các khối procedural block (`initial` hoặc `always`).
    *   **Lưu ý:** Việc khai báo một biến kiểu `reg` **không đồng nghĩa** với việc trình tổng hợp (synthesis tool) sẽ tạo ra một Flip-Flop vật lý. Nó có thể tổng hợp thành mạch tổ hợp (nếu nằm trong `always @(*)`) hoặc D-Flip-Flop (nếu nằm trong `always @(posedge clk)`).

```
               [Mạch Tổ Hợp]
   in1 ------->|             |---------> wire out_comb (assign out_comb = ...)
   in2 ------->|_____________|

               [Procedural Block always @(*)]
   in1 ------->|             |---------> reg out_reg (Cần case/if-else đủ)
   in2 ------->|_____________|

               [Procedural Block always @(posedge clk)]
   in ------->[ D-FF (reg) ]-----------> reg out_dff (Lưu trạng thái đồng bộ)
                  ^
   clk -----------+
```

---

### 2. Phép gán Nghẽn (Blocking) vs. Không nghẽn (Non-blocking)

*   **Blocking (`=`):**
    *   **Cách hoạt động:** Lệnh gán được thực thi tuần tự từ trên xuống dưới trong khối `always`. Dòng lệnh tiếp theo chỉ được đánh giá sau khi dòng lệnh hiện tại hoàn thành.
    *   **Ứng dụng:** Thiết kế mạch tổ hợp (`always @(*)`).
*   **Non-blocking (`<=`):**
    *   **Cách hoạt động:** Tất cả các biểu thức bên vế phải (RHS) được đánh giá (evaluate) tại thời điểm kích hoạt sự kiện sườn xung nhịp. Sau đó, các biến vế trái (LHS) mới được cập nhật đồng thời ở cuối chu kỳ mô phỏng.
    *   **Ứng dụng:** Thiết kế mạch tuần tự (`always @(posedge clk)`).
    *   **Quy tắc vàng (Cliff Cummings):**
        1. Sử dụng gán không nghẽn (`<=`) khi mô tả mạch tuần tự (Sequential logic).
        2. Sử dụng gán nghẽn (`=`) khi mô tả mạch tổ hợp (Combinational logic).
        3. Tuyệt đối không trộn lẫn cả hai phép gán trong cùng một khối `always`.

```
Blocking (=) thực thi tuần tự:
   a = b;  // a lấy giá trị của b lập tức
   c = a;  // c lấy giá trị mới của a (tức là b) -> Tổng hợp thành 1 dây dẫn từ b sang c.

Non-blocking (<=) cập nhật đồng thời:
   a <= b; // a nhận giá trị cũ của b
   c <= a; // c nhận giá trị cũ của a -> Tổng hợp thành chuỗi thanh ghi dịch (Shift Register).
```

---

### 3. Lỗi Inferred Latch (Tự động suy luận chốt)

*   **Nguyên nhân:** Xảy ra khi một biến kiểu `reg` được gán trong khối logic tổ hợp (`always @(*)`) nhưng không được định nghĩa giá trị ở mọi trường hợp ngõ vào có thể xảy ra (ví dụ: thiếu nhánh `else` trong cấu trúc `if-else`, hoặc thiếu nhánh `default` trong cấu trúc `case`).
*   **Hệ quả:** Vì mạch tổ hợp không có xung nhịp để lưu trữ dữ liệu, để giữ lại giá trị cũ của biến khi điều kiện không thỏa mãn, trình tổng hợp buộc phải tự động chèn thêm một Latch nhạy mức (Level-sensitive latch). Latch này gây ra lỗi phân tích timing (STA), tạo glitch và dẫn tới sai khác hành vi giữa mô phỏng RTL và thực tế silicon.
*   **Cách khắc phục:** 
    *   Đặt giá trị mặc định (Default assignments) ở ngay đầu khối `always @(*)`.
    *   Đảm bảo tất cả nhánh `if` đều có `else`, tất cả `case` đều có `default`.

```verilog
// SAI - Gây ra Inferred Latch trên biến out:
always @(*) begin
    if (en)
        out = data;
    // Thiếu nhánh else để định nghĩa biến out khi en = 0
end

// ĐÚNG - Khắc phục bằng giá trị mặc định:
always @(*) begin
    out = 1'b0; // Thiết lập giá trị mặc định ban đầu
    if (en)
        out = data;
end
```

---

### 4. Thiết kế Thực tế: ALU 8-bit và Testbench mô phỏng

#### A. Mã nguồn RTL (`alu_8bit.v`)
```verilog
module alu_8bit (
    input  wire [7:0] a,       // Toán hạng A
    input  wire [7:0] b,       // Toán hạng B
    input  wire [2:0] op,      // Mã lệnh thao tác (Operation code)
    output reg  [7:0] result,  // Kết quả ra ALU
    output reg        carry_out// Cờ nhớ (cho phép cộng/trừ)
);

    always @(*) begin
        // Giá trị mặc định để tránh Latch
        result    = 8'h00;
        carry_out = 1'b0;
        
        case (op)
            3'b000: {carry_out, result} = a + b;       // Cộng
            3'b001: {carry_out, result} = a - b;       // Trừ
            3'b010: result = a & b;                   // AND
            3'b011: result = a | b;                   // OR
            3'b100: result = a ^ b;                   // XOR
            3'b101: result = a << b[2:0];             // Dịch trái logic
            3'b110: result = a >> b[2:0];             // Dịch phải logic
            default: begin
                result    = 8'h00;
                carry_out = 1'b0;
            end
        endcase
    end

endmodule
```

#### B. Mã nguồn Testbench (`tb_alu_8bit.v`)
```verilog
`timescale 1ns/1ps

module tb_alu_8bit;

    reg  [7:0] a;
    reg  [7:0] b;
    reg  [2:0] op;
    wire [7:0] result;
    wire       carry_out;

    // Khởi tạo thực thể ALU
    alu_8bit uut (
        .a(a),
        .b(b),
        .op(op),
        .result(result),
        .carry_out(carry_out)
    );

    initial begin
        // Ghi lại file sóng để xem trên GTKWave
        $dumpfile("tb_alu_8bit.vcd");
        $dumpvars(0, tb_alu_8bit);

        // Kịch bản kiểm thử
        $display("Starting ALU Testbench...");
        
        // Testcase 1: Cộng không nhớ (15 + 10)
        a = 8'd15; b = 8'd10; op = 3'b000; #10;
        $display("ADD: %d + %d = %d (CO: %b)", a, b, result, carry_out);

        // Testcase 2: Cộng tràn cờ nhớ (250 + 10)
        a = 8'd250; b = 8'd10; op = 3'b000; #10;
        $display("ADD overflow: %d + %d = %d (CO: %b)", a, b, result, carry_out);

        // Testcase 3: Phép Trừ (20 - 5)
        a = 8'd20; b = 8'd5; op = 3'b001; #10;
        $display("SUB: %d - %d = %d (CO: %b)", a, b, result, carry_out);

        // Testcase 4: Phép toán logic AND
        a = 8'hAA; b = 8'h55; op = 3'b010; #10;
        $display("AND: %h & %h = %h", a, b, result);

        // Testcase 5: Dịch trái logic
        a = 8'h01; b = 8'd3; op = 3'b101; #10;
        $display("SLL: %h << %d = %h", a, b, result);

        $display("ALU Testbench Completed.");
        $finish;
    end

endmodule
```

---

## 📅 TUẦN 4: VIẾT MÃ NGUỒN MÔ TẢ FSM TRONG THỰC TẾ

### 1. Phương pháp thiết kế FSM 3 khối (3-Always Block FSM)

Trong thiết kế công nghiệp, phương pháp viết FSM 3 khối được ưu tiên nhờ việc tách biệt rõ ràng các nhiệm vụ vật lý của mạch logic số, giúp dễ đọc, dễ debug và tối ưu hóa timing tốt nhất.

```
                  +-----------------------------------+
                  |                                   |
   Inputs ------->|  Khối 2: Logic trạng thái kế tiếp  |
                  |             (Tổ hợp)              |
                  +-----------------------------------+
                                    | Next State
                                    v
                  +-----------------------------------+
     clk -------->|   Khối 1: Cập nhật trạng thái     |
   rst_n -------->|             (Tuần tự)             |
                  +-----------------------------------+
                                    | Current State
                                    +---------+
                                    |         |
                                    v         v
                  +-----------------------------------+
                  |      Khối 3: Logic ngõ ra         |
                  |     (Tổ hợp hoặc Tuần tự)         |
                  +-----------------------------------+
                                    |
                                    v Outputs
```

*   **Khối 1 (State Register):** Khối tuần tự nhạy sườn clock, làm nhiệm vụ gán trạng thái hiện tại bằng trạng thái tiếp theo (`current_state <= next_state`). Sử dụng phép gán không nghẽn.
*   **Khối 2 (Next State Logic):** Khối tổ hợp quyết định trạng thái chuyển tiếp tiếp theo dựa trên dữ liệu đầu vào và trạng thái hiện tại (`next_state = ...`). Sử dụng phép gán nghẽn.
*   **Khối 3 (Output Logic):** Khối xác định đầu ra dựa trên trạng thái hiện tại (và đầu vào nếu là Mealy FSM). Sử dụng ngõ ra kiểu tổ hợp (gán nghẽn) hoặc tuần tự để triệt tiêu nhiễu sụt áp (glitch free).

---

### 2. Kỹ thuật mã hóa: Binary vs. One-hot Encoding

| Đặc tính so sánh | Binary Encoding (Nhị phân) | One-hot Encoding |
| :--- | :--- | :--- |
| **Số Flip-Flop sử dụng** | Ít hơn ($\log_2(N)$ cho $N$ trạng thái) | Nhiều hơn ($N$ Flip-Flops cho $N$ trạng thái) |
| **Tài nguyên logic giải mã** | Phức tạp (Cần nhiều cổng logic MUX/AND/OR để giải mã trạng thái) | Cực kỳ đơn giản (Chỉ cần kiểm tra 1 bit tương ứng) |
| **Tần số hoạt động ($F_{max}$)** | Thấp hơn do đường trễ của mạch tổ hợp giải mã dài hơn | Cao hơn rất nhiều (tối ưu hóa timing tốt hơn) |
| **Ứng dụng tối ưu** | Thích hợp cho vi mạch ASIC hạn chế diện tích | Thích hợp cho thiết kế FPGA (vốn giàu Flip-Flop) |

---

### 3. Thực hành: Bộ điều khiển đèn giao thông (Traffic Light Controller)

Thiết kế mạch điều khiển đèn giao thông tại ngã tư của một đường chính (Main Road) và đường phụ (Side Road) có cảm biến phát hiện xe trên đường phụ (`car_sensor`).
*   **Trạng thái:**
    *   `S0_M_GREEN`: Đèn xanh cho đường chính, đỏ cho đường phụ.
    *   `S1_M_YELLOW`: Đèn vàng cho đường chính, đỏ cho đường phụ.
    *   `S2_S_GREEN`: Đèn đỏ cho đường chính, xanh cho đường phụ.
    *   `S3_S_YELLOW`: Đèn đỏ cho đường chính, vàng cho đường phụ.

#### A. Mã nguồn RTL (`traffic_controller.v`)
```verilog
module traffic_controller (
    input  wire       clk,
    input  wire       rst_n,
    input  wire       car_sensor, // 1: Có xe ở đường phụ, 0: Không có xe
    output reg  [1:0] main_light, // 2'b00: Đỏ, 2'b01: Vàng, 2'b10: Xanh
    output reg  [1:0] side_light
);

    // Định nghĩa trạng thái bằng One-hot Encoding
    localparam S0_M_GREEN  = 4'b0001;
    localparam S1_M_YELLOW = 4'b0010;
    localparam S2_S_GREEN  = 4'b0100;
    localparam S3_S_YELLOW = 4'b1000;

    reg [3:0] current_state;
    reg [3:0] next_state;
    reg [3:0] timer;               // Bộ đếm thời gian giản lược

    // Khối 1: Cập nhật trạng thái tuần tự
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            current_state <= S0_M_GREEN;
            timer         <= 4'd0;
        end else begin
            current_state <= next_state;
            if (current_state != next_state)
                timer <= 4'd0; // Reset bộ đếm khi đổi trạng thái
            else
                timer <= timer + 4'd1;
        end
    end

    // Khối 2: Logic xác định trạng thái tiếp theo
    always @(*) begin
        next_state = current_state;
        case (current_state)
            S0_M_GREEN: begin
                // Chuyển sang đèn vàng nếu có xe đường phụ và đã xanh đủ 10 chu kỳ
                if (car_sensor && (timer >= 4'd10))
                    next_state = S1_M_YELLOW;
            end
            S1_M_YELLOW: begin
                // Đèn vàng sáng trong 3 chu kỳ
                if (timer >= 4'd3)
                    next_state = S2_S_GREEN;
            end
            S2_S_GREEN: begin
                // Cho đường phụ xanh trong 6 chu kỳ hoặc về lại khi hết xe
                if (!car_sensor || (timer >= 4'd6))
                    next_state = S3_S_YELLOW;
            end
            S3_S_YELLOW: begin
                // Đèn vàng sáng trong 3 chu kỳ
                if (timer >= 4'd3)
                    next_state = S0_M_GREEN;
            end
            default: next_state = S0_M_GREEN;
        endcase
    end

    // Khối 3: Xác định ngõ ra
    always @(*) begin
        // Giá trị mặc định
        main_light = 2'b00; // Đỏ
        side_light = 2'b00; // Đỏ
        case (current_state)
            S0_M_GREEN: begin
                main_light = 2'b10; // Xanh
                side_light = 2'b00; // Đỏ
            end
            S1_M_YELLOW: begin
                main_light = 2'b01; // Vàng
                side_light = 2'b00; // Đỏ
            end
            S2_S_GREEN: begin
                main_light = 2'b00; // Đỏ
                side_light = 2'b10; // Xanh
            end
            S3_S_YELLOW: begin
                main_light = 2'b00; // Đỏ
                side_light = 2'b01; // Vàng
            end
        endcase
    end

endmodule
```

---

## 🖧 TUẦN 5: THIẾT KẾ NGOẠI VI NHÚNG UART TRÊN SILICON

### 1. Kiến trúc khối UART và nguyên lý truyền nối tiếp

Giao tiếp UART (Universal Asynchronous Receiver-Transmitter) là giao thức bất đồng bộ truyền dữ liệu nối tiếp từng bit qua hai đường dẫn độc lập TX và RX.

```
          +-------------------------------------------------------+
          |                      CHIP SILICON                     |
          |                                                       |
          |  [Baud Rate Generator]                                |
          |           | (Xung baud phát / 16x xung lấy mẫu RX)    |
          |           +---------------+                           |
          |           v               v                           |
  TX <----+=== [UART Transmit]   [UART Receive] <=================+=== RX
          |    (Shift Reg TX)    (Shift Reg RX & 16x Sampler)     |
          |           ^               |                           |
          |  8-bit    |               v  8-bit                    |
          |  Data ----+               +-------------> Data out    |
          |                                                       |
          +-------------------------------------------------------+
```

---

### 2. Kỹ thuật lấy mẫu Oversampling 16x trong UART RX

*   **Tại sao cần Oversampling?** Vì UART là giao tiếp bất đồng bộ, bộ thu RX không nhận được đường truyền clock từ bộ phát TX. Xung clock của RX độc lập và có thể lệch pha nhẹ. Để bắt chính xác điểm giữa của bit truyền, tránh nhiễu sườn xung, RX sử dụng một xung lấy mẫu có tần số chạy nhanh gấp 16 lần tốc độ Baud (16x Baud Rate).
*   **Nguyên lý hoạt động:**
    1. Trạng thái nghỉ (Idle) của đường truyền RX luôn ở mức cao (`1`).
    2. Khi phát hiện sườn xuống ($1 \rightarrow 0$), bộ thu nhận dạng đó là tín hiệu của **Start Bit**. Bộ đếm mẫu bắt đầu chạy từ `0`.
    3. Khi bộ đếm đạt giá trị `7` (chính giữa Start Bit), RX kiểm tra lại đường truyền. Nếu đúng là mức thấp (`0`), việc xác nhận Start Bit thành công.
    4. Kể từ thời điểm này, cứ sau mỗi $16$ nhịp đếm của xung lấy mẫu (tức là khoảng cách giữa tâm các bit tiếp theo), RX sẽ chốt giá trị đường truyền để đọc các bit dữ liệu từ $D_0$ đến $D_7$ và cuối cùng là **Stop Bit**.

```
RX Line:   ---111111\__ 0 __/ \__ D0 __/ \__ D1 __/
                    ^       ^          ^
                 Start      Mid-Start  Mid-D0
采样时钟 (16x):  |||||||||||||||||||||||||||||||||||
Bộ đếm mẫu:           0     7    0     15  0     15
```

---

### 3. Hiện thực hóa UART Transmitter (`uart_tx.v`)

```verilog
module uart_tx (
    input  wire       clk,         // Clock hệ thống (ví dụ: 50MHz)
    input  wire       rst_n,
    input  wire [7:0] tx_data,     // Dữ liệu 8-bit cần gửi
    input  wire       tx_start,    // Tín hiệu kích hoạt truyền (pulse)
    input  wire [15:0] clk_div_val,// Giá trị chia tần số tương ứng với Baud Rate
    output reg        tx_pin,      // Pin đầu ra truyền nối tiếp
    output reg        tx_busy      // Báo hiệu đang trong quá trình truyền
);

    // Trạng thái FSM
    localparam IDLE  = 2'b00;
    localparam START = 2'b01;
    localparam DATA  = 2'b10;
    localparam STOP  = 2'b11;

    reg [1:0]  state;
    reg [15:0] baud_counter;
    reg        baud_tick;
    reg [2:0]  bit_index;
    reg [7:0]  data_buffer;

    // Bộ phát xung Baud Tick (Baud Rate Generator)
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            baud_counter <= 16'd0;
            baud_tick    <= 1'b0;
        end else if (tx_busy) begin
            if (baud_counter >= clk_div_val - 1) begin
                baud_counter <= 16'd0;
                baud_tick    <= 1'b1;
            end else begin
                baud_counter <= baud_counter + 16'd1;
                baud_tick    <= 1'b0;
            end
        end else begin
            baud_counter <= 16'd0;
            baud_tick    <= 1'b0;
        end
    end

    // FSM truyền UART
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state       <= IDLE;
            tx_pin      <= 1'b1; // Đường truyền nghỉ ở mức 1
            tx_busy     <= 1'b0;
            bit_index   <= 3'd0;
            data_buffer <= 8'h00;
        end else begin
            case (state)
                IDLE: begin
                    tx_pin  <= 1'b1;
                    tx_busy <= 1'b0;
                    if (tx_start) begin
                        data_buffer <= tx_data;
                        tx_busy     <= 1'b1;
                        state       <= START;
                    end
                end
                START: begin
                    tx_pin <= 1'b0; // Start bit = 0
                    if (baud_tick) begin
                        state <= DATA;
                        bit_index <= 3'd0;
                    end
                end
                DATA: begin
                    tx_pin <= data_buffer[bit_index];
                    if (baud_tick) begin
                        if (bit_index >= 3'd7) begin
                            state <= STOP;
                        end else begin
                            bit_index <= bit_index + 3'd1;
                        end
                    end
                end
                STOP: begin
                    tx_pin <= 1'b1; // Stop bit = 1
                    if (baud_tick) begin
                        state <= IDLE;
                        tx_busy <= 1'b0;
                    end
                end
                default: state <= IDLE;
            endcase
        end
    end

endmodule
```

---

## 🔌 TUẦN 6: THIẾT KẾ NGOẠI VI SPI MASTER & TIMER

### 1. Phân cực và Pha xung nhịp SPI (4 Chế độ hoạt động)

Giao thức SPI (Serial Peripheral Interface) hoạt động đồng bộ dựa trên 2 tham số cấu hình trạng thái xung clock:
*   `CPOL` (Clock Polarity): Xác định mức logic nhàn rỗi (idle) của dây xung SCK.
    *   `CPOL = 0`: Khi nhàn rỗi, xung clock ở mức `0`.
    *   `CPOL = 1`: Khi nhàn rỗi, xung clock ở mức `1`.
*   `CPHA` (Clock Phase): Xác định sườn xung nhịp nào sẽ chốt dữ liệu (sample) và sườn nào dịch chuyển dữ liệu (shift).

```
                 SPI MODES TABLE
+------+------+-------------------+-------------------+
| Mode | CPOL | CPHA | Sample Edge| Shift Edge        |
+------+------+------+------------+-------------------+
|  0   |  0   |  0   | Rising (1) | Falling (0)       |
|  1   |  0   |  1   | Falling (0)| Rising (1)        |
|  2   |  1   |  0   | Falling (0)| Rising (1)        |
|  3   |  1   |  1   | Rising (1) | Falling (0)       |
+------+------+------+------------+-------------------+
```

---

### 2. Sơ đồ khối thanh ghi dịch hai chiều (Shift Register)

SPI hoạt động theo kiến trúc Full-duplex sử dụng thanh ghi dịch vòng tròn. Trong mỗi chu kỳ xung nhịp, 1 bit dữ liệu được đẩy ra đồng thời 1 bit dữ liệu được thu vào.

```
       Master (SPI Master)                        Slave (Peripheral)
+-------------------------------+          +-------------------------------+
|  Thanh ghi dịch 8-bit         |   MOSI   |  Thanh ghi dịch 8-bit         |
|  [D7 D6 D5 D4 D3 D2 D1 D0]    |--------->|  [D7 D6 D5 D4 D3 D2 D1 D0]    |
|   ^                           |          |                           |   |
|   +---------------------------|<---------|---------------------------+   |
+-------------------------------+   MISO   +-------------------------------+
                |                                          |
             [SCK] ------------------------------------> [SCK]
```

---

### 3. Thiết kế khối Timer 32-bit có tính năng Auto-reload

Timer là module đếm chu kỳ clock hệ thống để sinh ra ngắt có chu kỳ. Tính năng Auto-reload tự động nạp lại giá trị giới hạn đếm ban đầu khi thanh ghi đếm chạm ngưỡng `0`, duy trì chu kỳ đếm chính xác mà không cần phần mềm can thiệp viết lại.

#### Mã nguồn RTL (`timer_32bit.v`)
```verilog
module timer_32bit (
    input  wire        clk,
    input  wire        rst_n,
    input  wire [31:0] reload_val, // Giá trị tự động nạp lại
    input  wire        timer_en,   // Cho phép Timer hoạt động
    output reg         timer_irq   // Ngắt đầu ra (ngắt nháy 1 xung clk)
);

    reg [31:0] counter;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            counter   <= 32'd0;
            timer_irq <= 1'b0;
        end else if (timer_en) begin
            if (counter == 32'd0) begin
                counter   <= reload_val; // Tự động nạp lại giá trị
                timer_irq <= 1'b1;       // Kích hoạt ngắt
            end else begin
                counter   <= counter - 32'd1;
                timer_irq <= 1'b0;
            end
        end else begin
            timer_irq <= 1'b0;
        end
    end

endmodule
```

---

### 4. Tích hợp Hệ thống: Sơ đồ Vi kiến trúc (Microarchitecture)

Để tích hợp các ngoại vi này vào một thiết kế chip hoàn chỉnh, chúng được kết nối với một Bus điều khiển trung tâm (như APB/AHB Lite hoặc Bus tùy biến đơn giản).

```
                     +---------------------------------------+
                     |            BUS ĐIỀU KHIỂN             |
                     |     Address, Data In/Out, Control     |
                     +---------------------------------------+
                         |                 |               |
         +---------------+                 |               +---------------+
         | Write/Read                      | Write/Read                    | Write/Read
         v                                 v                               v
+------------------+              +------------------+            +------------------+
|     UART IP      |              |      SPI IP      |            |     TIMER IP     |
| (Control, Status |              | (Control, Status |            | (Reload register,|
|   Data Regs)     |              |   Data Regs)     |            |  Counter Regs)   |
+------------------+              +------------------+            +------------------+
    |           |                     |   |   |   |                   |
    v           v                     v   v   v   v                   v
  TX Pin      RX Pin                 CS SCK MOSI MISO              Interrupt Line
```
