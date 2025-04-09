# Verilog Project Repository

This repository contains Verilog implementations of various digital circuits. Each project includes its source code, testbench, and simulation results. The purpose of this repository is to demonstrate digital design concepts and verify them through simulations.

---

## 🧠 Project 1: Inverter

### 🔍 Theory
An **Inverter** (or **NOT gate**) outputs the complement of its input. It is one of the most basic digital logic gates.

In CMOS, an inverter is made using one PMOS and one NMOS transistor. It is used to flip binary values.

In VLSI design, inverters are implemented using CMOS (Complementary Metal-Oxide-Semiconductor) technology, which utilizes a combination of **PMOS** and **NMOS** transistors to achieve high speed and low power operation.

The inverter performs a **logical negation** operation. It simply flips the input logic level.

### ⚙️ Functionality
- Input `0` → Output `1`
- Input `1` → Output `0`

### 📊 Truth Table

| A | Y = ~A |
|---|--------|
| 0 |   1    |
| 1 |   0    |

### 🔗 Logic Symbol
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/invsym.png" alt="Inverter symbol">
</div>

### 💾 Source Code
```verilog
module inverter(a, y);
    input a;
    output y;
    assign y = ~a;
endmodule
```

### 🧪 Testbench
```verilog
module inverter_tb;
    reg a;
    wire y;

    inverter inv(.a(a), .y(y));

    initial begin
        $monitor("a=%b, y=%b", a, y);
        a = 0; #10;
        a = 1; #10;
        $finish;
    end
endmodule
```

### 📉 Simulation Result
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/inv.png?raw=true" alt="Inverter Simulation Waveform">Inverter Simulation Waveform
</div>

---

## 🧮 Project 2: 2-Digit BCD Adder

### 🔍 Theory
A **BCD (Binary-Coded Decimal) Adder** adds two decimal numbers represented in 8-bit BCD format (two digits). Each digit (0–9) is encoded using 4 bits. If the sum exceeds 9, it adds 6 (0110) to correct it back into BCD format.

### ⚙️ Functionality
- Handles addition of two 2-digit BCD numbers with carry-in.
- Corrects results that exceed `9` (1001).
  
### 🔗 Logic Symbol
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/clasum.png" alt=" symbol">
</div>

### 📊 Truth Table
| Input BCD A | Input BCD B | Sum | Correction |
|-------------|--------------|-----|------------|
| 0100 (4)    | 0101 (5)     | 1001 (9) | -      |
| 0101 (5)    | 0101 (5)     | 1010 (10) | +6 → 0000 (0) carry 1 |

### 💾 Source Code
```verilog
module twodigbcdadd(x, y, cin, sum, cout);
    input [7:0] x, y;
    input cin;
    output [8:0] sum;
    output cout;

    wire [3:0] lsdx, lsdy, msdx, msdy;
    wire [4:0] lsdbin_sum, lsdbcd_sum, msd_binsum, msd_bcdsum;
    wire clsd, cmsd;

    assign lsdx = x[3:0];
    assign msdx = x[7:4];
    assign lsdy = y[3:0];
    assign msdy = y[7:4];

    assign lsdbin_sum = lsdx + lsdy + cin;
    assign lsdbcd_sum = (lsdbin_sum > 9) ? (lsdbin_sum + 6) : lsdbin_sum;
    assign clsd = (lsdbin_sum > 9) ? 1 : 0;

    assign msd_binsum = msdx + msdy + clsd;
    assign msd_bcdsum = (msd_binsum > 9) ? (msd_binsum + 6) : msd_binsum;
    assign cmsd = (msd_binsum > 9) ? 1 : 0;

    assign sum = {msd_bcdsum[3:0], lsdbcd_sum[3:0]};
    assign cout = cmsd;
endmodule
```

### 🧪 Testbench
```verilog
module twodigbcdadd_tb;
    reg [7:0] x, y;
    reg cin;
    wire [8:0] sum;
    wire cout;

    twodigbcdadd uut (.x(x), .y(y), .cin(cin), .sum(sum), .cout(cout));

    initial begin
        $monitor("x=%b y=%b cin=%b | sum=%b cout=%b", x, y, cin, sum, cout);
        x = 8'b00000000; y = 8'b00000000; cin = 0; #10;
        x = 8'b00001001; y = 8'b00000001; cin = 0; #10;
        x = 8'b00001001; y = 8'b00001001; cin = 0; #10;
        x = 8'b00010010; y = 8'b00001001; cin = 1; #10;
        $finish;
    end
endmodule
```

### 📉 Simulation Result
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/2digbcd.png?raw=true" alt="2-digit BCD Adder Simulation Waveform">2-digit BCD Adder Simulation Waveform
</div>

---

## ➕ Project 3: 4-bit Carry Lookahead Adder


### 🔍 Theory
The **Carry Lookahead Adder (CLA)** is a high-speed adder that reduces delay by computing carry signals in advance using generate (`g`) and propagate (`p`) logic.

### ⚙️ Functionality
- Faster than ripple-carry adder.
- Uses parallel logic to determine carry.


### 📊 Truth Table
| A | B | Cin | Sum | Cout |
|---|---|-----|-----|------|
| 0010 | 0011 | 0 | 0101 | 0 |
| 1111 | 0001 | 1 | 0001 | 1 |

### 💾 Source Code
```verilog
module claa(a, b, cin, sum, cout);
    input [3:0] a, b;
    input cin;
    output [3:0] sum;
    output cout;

    wire [3:0] g, p, c;

    assign c[0] = cin;

    fulladder FA0 (.a(a[0]), .b(b[0]), .cin(c[0]), .sum(sum[0]), .g(g[0]), .p(p[0]));
    fulladder FA1 (.a(a[1]), .b(b[1]), .cin(c[1]), .sum(sum[1]), .g(g[1]), .p(p[1]));
    fulladder FA2 (.a(a[2]), .b(b[2]), .cin(c[2]), .sum(sum[2]), .g(g[2]), .p(p[2]));
    fulladder FA3 (.a(a[3]), .b(b[3]), .cin(c[3]), .sum(sum[3]), .g(g[3]), .p(p[3]));

    assign c[1] = g[0] | (p[0] & c[0]);
    assign c[2] = g[1] | (p[1] & c[1]);
    assign c[3] = g[2] | (p[2] & c[2]);
    assign cout = g[3] | (p[3] & c[3]);
endmodule

module fulladder(a, b, cin, sum, g, p);
    input a, b, cin;
    output sum, g, p;

    assign sum = a ^ b ^ cin;
    assign g = a & b;
    assign p = a ^ b;
endmodule
```

### 🧪 Testbench
```verilog
module cla_tb;
    reg [3:0] a, b;
    reg cin;
    wire [3:0] sum;
    wire cout;

    claa cla1(.a(a), .b(b), .cin(cin), .sum(sum), .cout(cout));

    initial begin
        $monitor("a=%b b=%b cin=%b | sum=%b cout=%b", a, b, cin, sum, cout);
        a = 4'b0001; b = 4'b0010; cin = 0; #10;
        a = 4'b1111; b = 4'b0001; cin = 1; #10;
        $finish;
    end
endmodule
```

### 📉 Simulation Result
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/cla.png?raw=true" alt="CLA Simulation Waveform">CLA Simulation Waveform
</div>

---

## 🔢 Project 4: BCD to 7-Segment Display Decoder

### 🔍 Theory
This module converts a 4-bit BCD input into a 7-bit output to drive a seven-segment display (common cathode). Each segment corresponds to a bit in the output.

### ⚙️ Functionality
- Converts BCD (0–9) into appropriate pattern to light segments `a` to `g` in display.

### 📊 Truth Table

| BCD | Output `abcdefg` |
|-----|------------------|
| 0000 | 0111111 |
| 0001 | 0000110 |
| 0010 | 1011011 |
| 0011 | 1001111 |
| 0100 | 1100110 |
| 0101 | 1101101 |
| 0110 | 1111101 |
| 0111 | 0000111 |
| 1000 | 1111111 |
| 1001 | 1101111 |

### 💾 Source Code
```verilog
module sevseg(bcd, sevseg);
    input [3:0] bcd;
    output [6:0] sevseg;

    function automatic [6:0] convert;
        input [3:0] bcd;
        begin
            case (bcd)
                4'b0000: convert = 7'b0111111;
                4'b0001: convert = 7'b0000110;
                4'b0010: convert = 7'b1011011;
                4'b0011: convert = 7'b1001111;
                4'b0100: convert = 7'b1100110;
                4'b0101: convert = 7'b1101101;
                4'b0110: convert = 7'b1111101;
                4'b0111: convert = 7'b0000111;
                4'b1000: convert = 7'b1111111;
                4'b1001: convert = 7'b1101111;
                default: convert = 7'b0000000;
            endcase
        end
    endfunction

    assign sevseg = convert(bcd);
endmodule

// or simpler code

 module bcd_seven (bcd, seven);
  input [3:0] bcd;
  output[7:1] seven;
  reg   [7:1] seven;
  always @(bcd)
  begin
    case (bcd)
      4'b0000 : seven = 7'b0111111 ;
      4'b0001 : seven = 7'b0000110 ;
      4'b0010 : seven = 7'b1011011 ;
      4'b0011 : seven = 7'b1001111 ;
      4'b0100 : seven = 7'b1100110 ;
      4'b0101 : seven = 7'b1101101 ;
      4'b0110 : seven = 7'b1111101 ;
      4'b0111 : seven = 7'b0000111 ;
      4'b1000 : seven = 7'b1111111 ;
      4'b1001 : seven = 7'b1101111 ;
      default : seven = 7'b0000000 ;
    endcase
  end
 endmodule
```

### 🧪 Testbench
```verilog
module sevseg_tb;
    reg [3:0] bcd;
    wire [6:0] sevseg;

    sevseg uut (.bcd(bcd), .sevseg(sevseg));

    initial begin
        $monitor("BCD = %b | Segments = %b", bcd, sevseg);
        bcd = 4'b0000; #10;
        bcd = 4'b0001; #10;
        bcd = 4'b0010; #10;
        bcd = 4'b0011; #10;
        bcd = 4'b0100; #10;
        bcd = 4'b0101; #10;
        bcd = 4'b0110; #10;
        bcd = 4'b0111; #10;
        bcd = 4'b1000; #10;
        bcd = 4'b1001; #10;
        $finish;
    end
endmodule
```

### 📉 Simulation Result
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/bcdseven.png" alt="7-Segment Display Simulation"> 7-Segment Display Simulation
</div>

---

## 🔢 Project : Traffic Light Controller

### 🔍 Theory
This module converts a 4-bit BCD input into a 7-bit output to drive a seven-segment display (common cathode). Each segment corresponds to a bit in the output.

### ⚙️ Functionality
- Converts BCD (0–9) into appropriate pattern to light segments `a` to `g` in display.

### 📊 Truth Table

| BCD | Output `abcdefg` |
|-----|------------------|
| 0000 | 0111111 |
| 0001 | 0000110 |
| 0010 | 1011011 |
| 0011 | 1001111 |
| 0100 | 1100110 |
| 0101 | 1101101 |
| 0110 | 1111101 |
| 0111 | 0000111 |
| 1000 | 1111111 |
| 1001 | 1101111 |

### 💾 Source Code
```verilog
module sevseg(bcd, sevseg);
    input [3:0] bcd;
    output [6:0] sevseg;

    function automatic [6:0] convert;
        input [3:0] bcd;
        begin
            case (bcd)
                4'b0000: convert = 7'b0111111;
                4'b0001: convert = 7'b0000110;
                4'b0010: convert = 7'b1011011;
                4'b0011: convert = 7'b1001111;
                4'b0100: convert = 7'b1100110;
                4'b0101: convert = 7'b1101101;
                4'b0110: convert = 7'b1111101;
                4'b0111: convert = 7'b0000111;
                4'b1000: convert = 7'b1111111;
                4'b1001: convert = 7'b1101111;
                default: convert = 7'b0000000;
            endcase
        end
    endfunction

    assign sevseg = convert(bcd);
endmodule

// or simpler code

 module bcd_seven (bcd, seven);
  input [3:0] bcd;
  output[7:1] seven;
  reg   [7:1] seven;
  always @(bcd)
  begin
    case (bcd)
      4'b0000 : seven = 7'b0111111 ;
      4'b0001 : seven = 7'b0000110 ;
      4'b0010 : seven = 7'b1011011 ;
      4'b0011 : seven = 7'b1001111 ;
      4'b0100 : seven = 7'b1100110 ;
      4'b0101 : seven = 7'b1101101 ;
      4'b0110 : seven = 7'b1111101 ;
      4'b0111 : seven = 7'b0000111 ;
      4'b1000 : seven = 7'b1111111 ;
      4'b1001 : seven = 7'b1101111 ;
      default : seven = 7'b0000000 ;
    endcase
  end
 endmodule
```

### 🧪 Testbench
```verilog
module sevseg_tb;
    reg [3:0] bcd;
    wire [6:0] sevseg;

    sevseg uut (.bcd(bcd), .sevseg(sevseg));

    initial begin
        $monitor("BCD = %b | Segments = %b", bcd, sevseg);
        bcd = 4'b0000; #10;
        bcd = 4'b0001; #10;
        bcd = 4'b0010; #10;
        bcd = 4'b0011; #10;
        bcd = 4'b0100; #10;
        bcd = 4'b0101; #10;
        bcd = 4'b0110; #10;
        bcd = 4'b0111; #10;
        bcd = 4'b1000; #10;
        bcd = 4'b1001; #10;
        $finish;
    end
endmodule
```

### 📉 Simulation Result
<div align="center">
  <img src="https://github.com/ShravanaHS/verilog/blob/main/files/bcdseven.png" alt="7-Segment Display Simulation"> 7-Segment Display Simulation
</div>

---


## ✅ How to Run Simulations

1. Install [ModelSim](https://www.intel.com/content/www/us/en/software-kit/683612/modelsim-starter-edition.html) or any Verilog simulator.
2. Compile the `.v` files.
3. Run the testbench file.
4. Observe outputs in the console and waveform viewer.

---


