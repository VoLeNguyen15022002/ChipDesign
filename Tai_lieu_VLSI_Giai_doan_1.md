# TÀI LIỆU CHUYÊN SÂU: GIAI ĐOẠN 1 (TUẦN 1 & TUẦN 2)
## ĐIỆN TỬ SỐ NÂNG CAO & PHÂN TÍCH TIMING CHI TIẾT (STA)

---

## 📚 TUẦN 1: ĐIỆN TỬ SỐ NÂNG CAO (ADVANCED DIGITAL ELECTRONICS)

### 1. Mạch tổ hợp (Combinational) và Mạch tuần tự (Sequential)

#### A. Phân tích cấp độ Transistor (CMOS Level)
Mọi mạch logic số đều được xây dựng từ các bóng bán dẫn **NMOS** và **PMOS** hoạt động ở chế độ đóng/ngắt (switch). 
*   **Mạch tổ hợp CMOS (Combinational CMOS):** Được xây dựng từ mạng kéo lên (Pull-Up Network - PUN) gồm các bóng bán dẫn PMOS nối tiếp/song song, và mạng kéo xuống (Pull-Down Network - PDN) gồm các bóng bán dẫn NMOS song song/nối tiếp đối ngẫu.
    *   *Ví dụ cổng NAND 2 ngõ vào:*
        *   **PUN (PMOS):** Hai bóng PMOS đấu song song từ nguồn $V_{DD}$ xuống ngõ ra $Y$.
        *   **PDN (NMOS):** Hai bóng NMOS đấu nối tiếp từ ngõ ra $Y$ xuống đất $GND$.
        *   *Nguyên lý:* Chỉ khi cả hai ngõ vào $A$ và $B$ đều ở mức cao ($1$), mạng PDN mới dẫn thông xuống đất và ngõ ra $Y = 0$. Các trường hợp khác, ít nhất một PMOS dẫn kéo ngõ ra lên $V_{DD} = 1$.

```
              VDD (1)
               |
         +-----+-----+
         |           |
       [PMOS]      [PMOS] (Song song)
      A -|        B -|
         |           |
         +-----+-----+
               |
               +------ Ngõ ra Y
               |
             [NMOS]
            A -|
               | (Nối tiếp)
             [NMOS]
            B -|
               |
              GND (0)
```

*   **Mạch tuần tự CMOS (Sequential CMOS):** Bắt buộc phải có vòng phản hồi (feedback loop) để lưu trạng thái hiện tại. Hai cổng NAND đấu chéo (Cross-coupled NANDs) tạo nên mạch chốt cơ bản nhất: **SR Latch**. Khi cả hai ngõ vào đều không tác động, trạng thái trước đó được giữ nguyên nhờ vòng lặp phản hồi duy trì điện áp.

---

### 2. Cấu trúc Vật lý: Latch vs. Flip-Flop

#### A. Cổng truyền CMOS (CMOS Transmission Gate - TG)
Cổng truyền TG đóng vai trò như một chuyển mạch điện tử hai chiều hoàn hảo trong thiết kế vi mạch, gồm một NMOS và một PMOS đấu song song với tín hiệu điều khiển ngược pha nhau ($C$ và $\bar{C}$):
*   Khi $C = 1$ ($\bar{C} = 0$), cổng truyền dẫn thông (ON).
*   Khi $C = 0$ ($\bar{C} = 1$), cổng truyền ngắt hoàn toàn (OFF), ngõ ra rơi vào trạng thái trở kháng cao (High-Z).

```
          PMOS (C_bar)
             _
          --\ /--
     In ----| |---- Out
          --/ \--
             |
          NMOS (C)
```

#### B. Hiện thực hóa D-Latch bằng Cổng truyền (TG-based D-Latch)
D-Latch nhạy mức cao (Active-high D-Latch) sử dụng hai cổng truyền $TG_1$, $TG_2$ và hai cổng đảo (Inverter) $I_1$, $I_2$:
*   **Khi $CLK = 1$ ($\bar{CLK} = 0$):** $TG_1$ mở (ON), $TG_2$ đóng (OFF). Dữ liệu từ ngõ vào $D$ đi trực tiếp qua $TG_1$ và cổng đảo $I_1$ ra ngõ ra $Q$. Trạng thái này gọi là **mở chốt (transparent mode)**.
*   **Khi $CLK = 0$ ($\bar{CLK} = 1$):** $TG_1$ đóng (OFF), $TG_2$ mở (ON). Ngõ vào $D$ bị ngắt hoàn toàn. Vòng phản hồi qua $I_1$, $I_2$ và $TG_2$ được thiết lập, giữ nguyên trạng thái logic của $Q$ tại thời điểm xung nhịp chuyển sang mức thấp. Trạng thái này gọi là **chốt giữ (hold mode)**.

```
            +-------- TG1 --------+       I1
            |      (CLK = 1)      |     +----+
     D ---->|  In ---------> Out  |---->| -O |----+----> Q
            |                     |     +----+    |
            +---------------------+               |
                                                  v
            +-------- TG2 --------+       I2    +----+
            |      (CLK = 0)      |     +----+  | -O |
     Q <----+  Out <--------  In  |<----| -O |<-+----+
            |                     |     +----+
            +---------------------+
```

#### C. Hiện thực hóa D-Flip-Flop dạng Master-Slave
Một D-Flip-Flop nhạy sườn lên (Rising-edge triggered D-FF) được tạo ra bằng cách nối tiếp hai D-Latch: **Master Latch** (nhạy mức thấp) và **Slave Latch** (nhạy mức cao).

*   **Khi $CLK = 0$:** Master Latch ở chế độ *transparent*, nhận dữ liệu từ đầu vào $D$ nhưng dữ liệu này bị Slave Latch chặn lại vì Slave Latch đang ở chế độ *hold*.
*   **Khi $CLK$ chuyển tiếp từ $0 \rightarrow 1$ (Sườn lên):** 
    *   Master Latch chuyển sang chế độ *hold*, chốt giữ giá trị của $D$ ngay tại thời điểm sườn lên.
    *   Cùng lúc đó, Slave Latch chuyển sang chế độ *transparent*, truyền giá trị chốt từ Master Latch ra ngõ ra cuối cùng $Q$.
    *   Nhờ cơ chế này, dữ liệu ngõ ra chỉ thay đổi tại khoảnh khắc sườn lên của clock.

---

### 3. FSM (Finite State Machine): Mealy vs. Moore

#### A. So sánh chi tiết về thiết kế mạch vật lý
1.  **Độ trễ và Tốc độ đáp ứng:**
    *   **Mealy FSM:** Đầu ra thay đổi ngay khi đầu vào thay đổi. Do đó, đường truyền từ đầu vào qua khối logic tổ hợp ngõ ra đến đích sẽ dài hơn, làm tăng thời gian trễ lan truyền và làm giảm tần số hoạt động tối đa của toàn hệ thống.
    *   **Moore FSM:** Đầu ra được cô lập khỏi ngõ vào bất đồng bộ thông qua các Flip-flop trạng thái. Mạch tổ hợp ngõ ra của Moore chỉ phụ thuộc vào trạng thái hiện tại (đã được đồng bộ), nên tần số hoạt động của mạch thường cao hơn.
2.  **Khả năng xuất hiện Glitch (Nhiễu sụt áp/xung nhọn):**
    *   Vì ngõ ra của Mealy phụ thuộc trực tiếp vào ngõ vào, bất kỳ glitch nào trên đường dây ngõ vào hoặc trong khối logic tổ hợp ngõ ra sẽ xuất hiện ngay lập tức ở ngõ ra. Điều này cực kỳ nguy hiểm nếu ngõ ra này điều khiển chân Reset hoặc Enable của một module khác. Do đó, ngõ ra của Mealy thường phải được đồng bộ hóa lại qua một Flip-Flop đệm (Registered Output).

---

### 4. Thiết kế Chi tiết Mạch nhận diện chuỗi "1101" (Sequence Detector)

#### A. Thiết kế logic chi tiết (Logic Gate Derivation) cho Moore FSM
Sử dụng 5 trạng thái đã định nghĩa ở phần trước: S0 (000), S1 (001), S2 (010), S3 (011), S4 (100). Ta mã hóa trạng thái bằng 3 Flip-flops ($Q_2, Q_1, Q_0$).

Bảng mã hóa và chuyển trạng thái chi tiết:

| Trạng thái hiện tại ($Q_2Q_1Q_0$) | Ngõ vào ($X$) | Trạng thái tiếp theo ($D_2D_1D_0$) | Ngõ ra ($Y$) |
| :---: | :---: | :---: | :---: |
| S0 (000) | 0 | S0 (000) | 0 |
| S0 (000) | 1 | S1 (001) | 0 |
| S1 (001) | 0 | S0 (000) | 0 |
| S1 (001) | 1 | S2 (010) | 0 |
| S2 (010) | 0 | S3 (011) | 0 |
| S2 (010) | 1 | S2 (010) | 0 |
| S3 (011) | 0 | S0 (000) | 0 |
| S3 (011) | 1 | S4 (100) | 0 |
| S4 (100) | 0 | S0 (000) | 1 |
| S4 (100) | 1 | S2 (010) | 1 |

Dùng bản đồ Karnaugh để tối ưu hóa phương trình ngõ vào cho các Flip-flop ($D_2, D_1, D_0$):
*   $D_2 = Q_1 \cdot Q_0 \cdot X$
*   $D_1 = (\bar{Q_2} \cdot Q_0 \cdot X) + (Q_1 \cdot \bar{Q_0} \cdot \bar{X}) + (Q_2 \cdot X)$
*   $D_0 = (\bar{Q_2} \cdot \bar{Q_1} \cdot \bar{Q_0} \cdot X) + (Q_1 \cdot \bar{Q_0} \cdot \bar{X})$
*   Phương trình ngõ ra: $Y = Q_2$

#### B. Hiện thực bằng ngôn ngữ Verilog HDL (Mô hình FSM 3 khối chuẩn)

```verilog
module sequence_detector_1101 (
    input  wire clk,
    input  wire rst_n,
    input  wire x,
    output reg  y
);

    // Định nghĩa các trạng thái bằng Parameter
    localparam S0 = 3'b000; // IDLE
    localparam S1 = 3'b001; // Đã nhận '1'
    localparam S2 = 3'b010; // Đã nhận '11'
    localparam S3 = 3'b011; // Đã nhận '110'
    localparam S4 = 3'b100; // Đã nhận '1101'

    reg [2:0] current_state;
    reg [2:0] next_state;

    // Khối 1: Cập nhật trạng thái đồng bộ (Sequential Block)
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            current_state <= S0;
        end else begin
            current_state <= next_state;
        end
    end

    // Khối 2: Logic xác định trạng thái tiếp theo (Combinational Block)
    always @(*) begin
        case (current_state)
            S0: begin
                if (x) next_state = S1;
                else   next_state = S0;
            end
            S1: begin
                if (x) next_state = S2;
                else   next_state = S0;
            end
            S2: begin
                if (x) next_state = S2; // Chồng lấn '111...' giữ ở S2
                else   next_state = S3;
            end
            S3: begin
                if (x) next_state = S4;
                else   next_state = S0;
            end
            S4: begin
                if (x) next_state = S2; // Xét chồng lấn bit '1' cuối
                else   next_state = S0;
            end
            default: next_state = S0;
        endcase
    end

    // Khối 3: Logic xác định ngõ ra đồng bộ (Registered Output) tránh Glitch
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            y <= 1'b0;
        end else begin
            if (next_state == S4) begin
                y <= 1'b1;
            end else begin
                y <= 1'b0;
            end
        end
    end

endmodule
```

---

## ⏱️ TUẦN 2: PHÂN TÍCH TIMING TĨNH CHUYÊN SÂU (STA)

### 1. Phân tích chi tiết đường truyền dữ liệu trong STA

Phân tích tĩnh timing không chạy các phép mô phỏng động, mà nó phân tích mọi đường dẫn vật lý (paths) từ điểm bắt đầu (Startpoint - chân CLK của FF nguồn) đến điểm kết thúc (Endpoint - chân D của FF đích) để kiểm tra các ràng buộc.

#### A. Launch Path và Capture Path
*   **Launch Path (Đường phát):** Đường đi của dữ liệu bắt đầu từ sườn xung nhịp xuất phát (Launch Edge) tại FF1 qua mạch tổ hợp đến ngõ vào D của FF2.
    *   *Thời gian dữ liệu đến đích thực tế (Data Arrival Time):*
        $$T_{arrival} = T_{clk\_source\_delay} + T_{cq} + T_{comb}$$
*   **Capture Path (Đường thu):** Đường đi của xung nhịp từ chân Clock nguồn qua mạng lưới phân phối xung nhịp đến chân CLK của FF2 để chốt dữ liệu tại sườn xung nhịp tiếp theo (Capture Edge).
    *   *Thời gian dữ liệu bắt buộc phải sẵn sàng (Data Required Time - Setup):*
        $$T_{required\_setup} = T_{clk} + T_{clk\_dest\_delay} - T_{setup}$$
    *   *Thời gian dữ liệu bắt buộc phải giữ nguyên (Data Required Time - Hold):*
        $$T_{required\_hold} = T_{clk\_dest\_delay} + T_{hold}$$

#### B. Khái niệm Timing Slack (Độ dư thời gian)
*   **Setup Slack:**
    $$Slack_{setup} = T_{required\_setup} - T_{arrival}$$
    $$Slack_{setup} = (T_{clk} + T_{clk\_dest\_delay} - T_{setup}) - (T_{clk\_source\_delay} + T_{cq} + T_{comb})$$
    Nếu định nghĩa $T_{skew} = T_{clk\_dest\_delay} - T_{clk\_source\_delay}$, ta có:
    $$Slack_{setup} = T_{clk} + T_{skew} - T_{cq} - T_{comb} - T_{setup}$$
    *   **$Slack_{setup} \ge 0$:** Đạt yêu cầu về thời gian (MET).
    *   **$Slack_{setup} < 0$:** Vi phạm thời gian (VIOLATED). Chip hoạt động sai ở tần số mục tiêu.

*   **Hold Slack:**
    $$Slack_{hold} = T_{arrival} - T_{required\_hold}$$
    $$Slack_{hold} = (T_{clk\_source\_delay} + T_{cq} + T_{comb}) - (T_{clk\_dest\_delay} + T_{hold})$$
    $$Slack_{hold} = T_{cq} + T_{comb} - T_{skew} - T_{hold}$$
    *   **$Slack_{hold} \ge 0$:** Đạt yêu cầu (MET).
    *   **$Slack_{hold} < 0$:** Vi phạm thời gian (VIOLATED). Dữ liệu mới ghi đè dữ liệu cũ trước khi nó được chốt an toàn.

---

### 2. Ví dụ số liệu thực tế từ công cụ EDA (OpenROAD/OpenSTA)

Xem xét báo cáo STA sau khi chạy Place & Route (PnR) trên thiết kế với chu kỳ đích $T_{clk} = 20\text{ ns}$:

#### Báo cáo Setup Slack (`max.rpt`):
```
Startpoint: reg_a (rising edge-triggered flip-flop clocked by clock)
Endpoint: reg_sum (rising edge-triggered flip-flop clocked by clock)
Path Group: clock
Path Type: max (setup check)

Point                                    Delay      Time
---------------------------------------------------------
clock clock (rising edge)                            0.00
clock network delay (source)              0.50       0.50
reg_a/CLK (rising edge-triggered D-FF)               0.50
reg_a/Q (Clock-to-Q delay)                0.20       0.70
u_adder/add_block/Y (logic delay)         2.50       3.20
reg_sum/D (data arrival time)                        3.20

clock clock (rising edge + Tclk)                    20.00
clock network delay (destination)         0.45      20.45
reg_sum/setup (setup time constraint)    -0.31      20.14
reg_sum/D (data required time)                      20.14
---------------------------------------------------------
Data Required Time                                  20.14
Data Arrival Time                                   -3.20
---------------------------------------------------------
Slack (MET)                                         16.94 ns
```
*   **Phân tích:** 
    *   Thời gian đến của dữ liệu là $3.20\text{ ns}$.
    *   Thời gian yêu cầu dữ liệu phải sẵn sàng là $20.14\text{ ns}$.
    *   Độ dư thời gian (Setup Slack) cực kỳ lớn: $16.94\text{ ns}$ (MET).
    *   *Tối ưu hóa:* Mạch có thể thu nhỏ chu kỳ clock tối đa xuống mức: $T_{clk\_min} = 3.20\text{ ns} + 0.31\text{ ns} - (0.45\text{ ns} - 0.50\text{ ns}) = 3.56\text{ ns}$.
    *   Tần số hoạt động tối đa thực tế là: $F_{max} = \frac{1}{3.56\text{ ns}} \approx 280\text{ MHz}$.

#### Báo cáo Hold Slack (`min.rpt`):
```
Startpoint: reg_a (rising edge-triggered flip-flop clocked by clock)
Endpoint: reg_sum (rising edge-triggered flip-flop clocked by clock)
Path Group: clock
Path Type: min (hold check)

Point                                    Delay      Time
---------------------------------------------------------
clock clock (rising edge)                            0.00
clock network delay (source)              0.50       0.50
reg_a/CLK (rising edge-triggered D-FF)               0.50
reg_a/Q (Clock-to-Q delay)                0.15       0.65
u_adder/short_path (logic delay)          0.10       0.75
reg_sum/D (data arrival time)                        0.75

clock clock (rising edge)                            0.00
clock network delay (destination)         0.45       0.45
reg_sum/hold (hold time constraint)       0.20       0.65
reg_sum/D (data required time)                       0.65
---------------------------------------------------------
Data Arrival Time                                    0.75
Data Required Time                                  -0.65
---------------------------------------------------------
Slack (MET)                                          0.10 ns
```
*   **Phân tích:** 
    *   Dữ liệu đến lúc $0.75\text{ ns}$.
    *   Yêu cầu giữ ổn định đến ít nhất $0.65\text{ ns}$.
    *   Độ dư Hold Slack rất nhỏ nhưng vẫn đạt yêu cầu: $+0.10\text{ ns}$ (MET). Nếu đường dây tổ hợp bị trễ ít hơn $0.10\text{ ns}$ nữa thì chip sẽ bị hỏng hoàn toàn.

---

### 3. Metastability (Giả bền) dưới góc nhìn vật lý bán dẫn

#### A. Mô hình "Quả bóng trên đỉnh đồi" (Ball-on-Hill Analogy)
Khi Flip-flop hoạt động bình thường, điện áp đầu ra của nó giống như một quả bóng nằm ở một trong hai thung lũng ổn định đại diện cho trạng thái `0` (thung lũng trái) hoặc `1` (thung lũng phải).

```
        Trạng thái Giả bền
            (Metastable)
                 O  <- Quả bóng lơ lửng trên đỉnh
                / \
               /   \
              /     \
             /       \
            /         \
     (0)  O             O  (1)
     Thung lũng tả     Thung lũng hữu
```

Khi vi phạm Setup/Hold, đầu ra Flip-flop bị đẩy lên đỉnh đồi (trạng thái giả bền). Do lực cân bằng tạm thời, quả bóng lơ lửng trên đỉnh một khoảng thời gian không xác định trước khi lăn xuống một trong hai bên.

#### B. Công thức tính MTBF của Metastability
Thời gian trung bình giữa các lần gặp lỗi của hệ thống do Metastability gây ra được tính theo công thức:

$$MTBF = \frac{e^{\frac{t_{res}}{\tau}}}{T_W \cdot f_{clk} \cdot f_{data}}$$

Trong đó:
*   $t_{res}$: Thời gian cho phép để tín hiệu tự ổn định về mức logic (thời gian phân giải).
*   $\tau$: Hằng số thời gian phân giải của Flip-Flop (quyết định bởi độ dốc của đồi, phụ thuộc vào công nghệ chế tạo như 130nm, 28nm, 7nm). Công nghệ càng nhỏ, $\tau$ càng nhỏ, tốc độ tự phân giải càng nhanh.
*   $T_W$ (hoặc $T_0$): Cửa sổ thời gian nhạy cảm (Aperture window) xung quanh sườn clock mà nếu tín hiệu dữ liệu thay đổi trong đó, lỗi chắc chắn xảy ra.
*   $f_{clk}$: Tần số xung nhịp lấy mẫu của miền nhận dữ liệu.
*   $f_{data}$: Tần số thay đổi trung bình của tín hiệu dữ liệu đầu vào.

---

### 4. Giải pháp CDC nâng cao: Bộ đệm FIFO bất đồng bộ (Asynchronous FIFO)

Khi cần truyền một bus dữ liệu nhiều bit (ví dụ: bus dữ liệu 32-bit) qua lại giữa các miền clock khác nhau, ta **không thể** sử dụng độc lập các bộ đồng bộ 2 tầng Flip-flop cho từng bit. Do độ trễ đi dây trên chip không đồng đều (Bus Skew), các bit dữ liệu sẽ đến miền nhận lệch nhau về thời gian, dẫn đến việc giải mã sai dữ liệu (data corruption).

Giải pháp tiêu chuẩn công nghiệp là sử dụng **FIFO bất đồng bộ (Asynchronous FIFO)**:

```
Miền ghi (Wr_clk)                                Miền đọc (Rd_clk)
+-----------------+                            +-----------------+
|                 |                            |                 |
|  --> Write -->  |=======[ Dữ liệu RAM ]======|  --> Read -->   |
|                 |                            |                 |
|  Wr_ptr (Nhị   |                            |  Rd_ptr (Nhị   |
|  phân -> Gray)  |                            |  phân -> Gray)  |
|        ||       |                            |        ||       |
|        v        |                            |        v        |
|   Wr_ptr_gray   | ---(Bộ đồng bộ 2-stage)--->|   Wr_ptr_sync   | ==> Logic tạo
|                 |                            |                 |     cờ Empty
|                 |<---(Bộ đồng bộ 2-stage)----|   Rd_ptr_gray   |
|   Rd_ptr_sync   |                            |                 |
|        ||       |                            |                 |
|        v        |                            |                 |
|  ==> Logic tạo  |                            |                 |
|      cờ Full    |                            |                 |
+-----------------+                            +-----------------+
```

#### Các thành phần chính và nguyên lý đồng bộ:
1.  **Dùng mã Gray thay vì mã Nhị phân cho con trỏ (Pointer):**
    *   Trong mã nhị phân, khi con trỏ chuyển từ `3 (011)` sang `4 (100)`, cả 3 bit đều thay đổi đồng thời. Nếu lấy mẫu tại thời điểm CDC, sự thay đổi không đồng bộ này sẽ tạo ra trạng thái giả ngẫu nhiên sai lệch nghiêm trọng.
    *   **Mã Gray** chỉ thay đổi **duy nhất 1 bit** giữa các trạng thái kế tiếp (ví dụ: `3 (010)` sang `4 (110)`). Khi chỉ có 1 bit thay đổi, ngay cả khi xảy ra metastability, miền nhận chỉ có thể giải mã ra giá trị cũ hoặc giá trị mới, hoàn toàn không tạo ra giá trị rác.
2.  **Đồng bộ hóa con trỏ qua bộ đồng bộ 2 tầng Flip-flop:**
    *   Con trỏ ghi `Wr_ptr` (đã mã hóa Gray) được truyền sang miền đọc `Rd_clk` qua bộ đồng bộ 2 tầng để so sánh và tạo ra tín hiệu báo rỗng **(Empty flag)**.
    *   Con trỏ đọc `Rd_ptr` (đã mã hóa Gray) được truyền sang miền ghi `Wr_clk` qua bộ đồng bộ 2 tầng để so sánh và tạo ra tín hiệu báo đầy **(Full flag)**.
3.  **Thuật toán tạo cờ báo:**
    *   **Cờ rỗng (Empty):** Khi con trỏ ghi được đồng bộ trùng hoàn toàn với con trỏ đọc hiện tại.
        $$Wr\_ptr\_sync == Rd\_ptr\_gray$$
    *   **Cờ đầy (Full):** Con trỏ ghi đuổi kịp con trỏ đọc sau một vòng RAM. Khi mã hóa Gray, điều này xảy ra khi hai bit MSB (bit trọng số lớn nhất) và bit kế tiếp của hai con trỏ ngược nhau, trong khi các bit còn lại trùng nhau.
