
# Verilog Project

## Introduction
This repository contains Verilog implementations of various digital circuits. Each project includes its source code, testbench, and simulation results. The purpose of this repository is to demonstrate digital design concepts and verify them through simulations.

## Projects

### Project 1: Inverter
### THEORY
An **Inverter**, also known as a **NOT gate**, is a basic logic gate that outputs the **complement** (inverse) of its input. It is one of the fundamental building blocks in digital logic design.

In digital terms:
- If the input is `1`, the output is `0`.
- If the input is `0`, the output is `1`.

In VLSI design, inverters are implemented using CMOS (Complementary Metal-Oxide-Semiconductor) technology, which utilizes a combination of **PMOS** and **NMOS** transistors to achieve high speed and low power operation.

The inverter performs a **logical negation** operation. It simply flips the input logic level.

- Input HIGH (`1`) → Output LOW (`0`)
- Input LOW (`0`) → Output HIGH (`1`)

***Truth Table***

| Input (A) | Output (Y = ~A) |
|-----------|-----------------|
|     0     |        1        |
|     1     |        0        |


**block diagram**
##  Logic Symbol
<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/invsym.png" alt="Inverter symbol" > inverter symbol </div>

#### Source Code
```verilog
// inverter.v
module inverter(a,y);
input a;
output y;
    assign y = ~a;
endmodule
```

#### Testbench
```verilog
// inverter_tb.v
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

#### Simulation Result
```
a=0, y=1
a=1, y=0
```
<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/inv.png?raw=true" alt="Inverter Simulation Waveform" > Inverter Simulation Waveform </div>

---

### Project 2: 2-Digit BCD Adder
#### Block Diagram 

#### Source Code
```verilog
// twodigbcdadd.v
module twodigbcdadd(x, y, cin, sum, cout);
// Defining input and outputs
input [7:0] x, y;
input cin;
output [8:0] sum;
output cout;

// Assigning wires for sum, corrected sum, carry
wire [3:0] lsdx, lsdy, msdx, msdy;
wire [4:0] lsdbin_sum, lsdbcd_sum, msd_binsum, msd_bcdsum;
wire clsd, cmsd;

// Splitting input into LSD and MSD
assign lsdx = x[3:0];
assign msdx = x[7:4];
assign lsdy = y[3:0];
assign msdy = y[7:4];

// Performing LSD addition
assign lsdbin_sum = lsdx + lsdy + cin;
assign lsdbcd_sum = (lsdbin_sum > 9) ? (lsdbin_sum + 6) : lsdbin_sum;
assign clsd = (lsdbin_sum > 9) ? 1 : 0;

// Performing MSD addition
assign msd_binsum = msdx + msdy + clsd;
assign msd_bcdsum = (msd_binsum > 9) ? (msd_binsum + 6) : msd_binsum;
assign cmsd = (msd_binsum > 9) ? 1 : 0;

// Correcting sum output (concatenating MSD and LSD)
assign sum = {msd_bcdsum[3:0], lsdbcd_sum[3:0]};
assign cout = cmsd;

endmodule
```

#### Testbench
```verilog
// twodigbcdadd_tb.v

`timescale 1ns/1ps

module twodigbcdadd_tb;
    reg [7:0] x, y;
    reg cin;
    wire [8:0] sum;
    wire cout;

    // Instantiate the Unit Under Test (UUT)
       twodigitbcd uut (
        .x(x),
        .y(y),
        .cin(cin),
        .sum(sum),
        .cout(cout)
    );
    initial begin
        // Monitor the signals
        $monitor("Time=%0t | x=%b (%d) y=%b (%d) cin=%b | sum=%b (%d) cout=%b", 
                 $time, x, x, y, y, cin, sum, sum, cout);

        // Test cases
        x = 8'b00000000; y = 8'b00000000; cin = 0; #10;
        x = 8'b00000001; y = 8'b00000001; cin = 0; #10;
        x = 8'b00000101; y = 8'b00000101; cin = 0; #10;
        x = 8'b00001000; y = 8'b00001001; cin = 0; #10;
        x = 8'b00001001; y = 8'b00001001; cin = 0; #10;
        x = 8'b00010010; y = 8'b00001001; cin = 1; #10;
        x = 8'b00011001; y = 8'b00011001; cin = 0; #10;

        $finish;
    end
endmodule

```

#### Simulation Result

<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/2digbcd.png?raw=true" alt="2-digit BCD Adder Simulation Waveform"> 2-digit BCD Adder Simulation Waveform </div>


---

### Project 3: 4-Bit Carry Lookahead Adder
#### block diagram
<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/clasum.png" alt="sumbol"> 4 Bit carry lookahead adder symbol </div>

#### Source Code
```verilog
// claa.v

module claa(a, b, cin, sum, cout);
  input [3:0] a, b;
  input cin;
  output [3:0] sum;
  output cout;

  wire [3:0] g, p, c;  // Generate, Propagate, Carry Wires

  assign c[0] = cin;  // First carry-in is cin

  // Instantiate Full Adders
  fulladder FA0 (.a(a[0]), .b(b[0]), .cin(c[0]), .sum(sum[0]), .g(g[0]), .p(p[0]));
  fulladder FA1 (.a(a[1]), .b(b[1]), .cin(c[1]), .sum(sum[1]), .g(g[1]), .p(p[1]));
  fulladder FA2 (.a(a[2]), .b(b[2]), .cin(c[2]), .sum(sum[2]), .g(g[2]), .p(p[2]));
  fulladder FA3 (.a(a[3]), .b(b[3]), .cin(c[3]), .sum(sum[3]), .g(g[3]), .p(p[3]));

  // Carry Look-Ahead Logic
  assign c[1] = g[0] | (p[0] & c[0]);
  assign c[2] = g[1] | (p[1] & c[1]);
  assign c[3] = g[2] | (p[2] & c[2]);
  assign cout = g[3] | (p[3] & c[3]);

endmodule

module fulladder(a,b,cin,sum,g,p);
input a,b;
input cin;
output sum;
//output cout;
output g,p; //carry generation and propogation

assign sum = a^b^cin;
assign g = a&b;
assign p = a^b;
//assign cout = a&b | (a^b)&cin;
endmodule


```

#### Testbench
```verilog
// cla_tb.v

module cla_tb();
    
    reg [3:0] a, b;
    reg cin;
    wire [3:0] sum;
    wire cout;
    
    
    claa  cla1(.a(a), .b(b), .cin(cin), .sum(sum), .cout(cout));
    
    initial begin
        // Test cases
        $monitor("Time=%0t | a=%b b=%b cin=%b | sum=%b cout=%b", $time, a, b, cin, sum, cout);
        
      #10  a = 4'b0000; b = 4'b0000; cin = 0; #10;
       #10  a = 4'b0001; b = 4'b0001; cin = 0; #10;
       #10 a = 4'b0011; b = 4'b0011; cin = 0; #10;
       #10 a = 4'b0101; b = 4'b0010; cin = 1; #10;
       #10 a = 4'b0110; b = 4'b0110; cin = 0; #10;
       #10 a = 4'b1111; b = 4'b1111; cin = 1; #10;
        
        // End simulation
       #100 $finish;
    end
endmodule ```

#### Simulation Result
```
<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/cla.png" alt="Carry Look Ahead Adder Simulation Waveform" > Carry Look Ahead Adder Simulation Waveform </div>
```

---

### Project 4: BCD to 7-Segment Display decoder
#### Source Code
```verilog
// sevseg.v
module sevseg(bcd, sevseg);
    input [3:0] bcd;
    output [6:0] sevseg;  


function automatic [6:0] convert;  
    input [3:0] bcd;
    begin
        case (bcd)
            4'b0000: convert = 7'b0111111; // 0
            4'b0001: convert = 7'b0000110; // 1
            4'b0010: convert = 7'b1011011; // 2
            4'b0011: convert = 7'b1001111; // 3
            4'b0100: convert = 7'b1100110; // 4
            4'b0101: convert = 7'b1101101; // 5
            4'b0110: convert = 7'b1111101; // 6
            4'b0111: convert = 7'b0000111; // 7
            4'b1000: convert = 7'b1111111; // 8
            4'b1001: convert = 7'b1101111; // 9
            default: convert = 7'b0000000; // Blank 
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

#### Testbench
```verilog
// sevseg_tb.v
module sevseg_tb();
    
    reg [3:0] bcd;
    wire [6:0] sevseg;  
    sevseg sevseg1(.bcd(bcd), .sevseg(sevseg));

    initial 
    begin
        #0  bcd = 4'b0000;  // 0  
        #10 bcd = 4'b0001;  // 1  
        #10 bcd = 4'b0010;  // 2  
        #10 bcd = 4'b0011;  // 3  
        #10 bcd = 4'b0100;  // 4  
        #10 bcd = 4'b0101;  // 5  
        #10 bcd = 4'b0110;  // 6  
        #10 bcd = 4'b0111;  // 7  
        #10 bcd = 4'b1000;  // 8  
        #10 bcd = 4'b1001;  // 9  
    end  
    initial 
    begin 
        $monitor($time, " | BCD = %b | Seven-Segment = %h", bcd, sevseg); 
        #100 $finish;  
    end

endmodule


```

#### Simulation Result

<div align="center"> <img src="https://github.com/ShravanaHS/verilog/blob/main/files/bcdseven.png" alt="decoder Simulation Waveform" >decoder Simulation Waveform </div>


## Getting Started
### Prerequisites
Ensure you have the following tool installed:
- **ModelSim** for simulation and waveform analysis

### Cloning the Repository
```sh
git clone https://github.com/yourusername/verilog-project.git
cd verilog-project
```


