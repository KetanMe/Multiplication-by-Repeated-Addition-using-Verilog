# Multiplication-by-Repeated-Addition-using-Verilog

This Verilog code describes a basic example of a hardware design for multiplication using a sequential approach and includes a controller for managing the state transitions of the operation.

```verilog
module MUL_datapath(eqz, ldA, ldB, decB, ldP, clrP, data_in, clk);
    input ldA, ldB, ldP, clrP, decB, clk;
    input [15:0] data_in;
    output eqz;
    wire [15:0] X, Y, Z, Bout, Bus;
 
    PIPO1 A (X, Bus, ldA, clk);
    PIPO2 P (Y, Z, ldP, clrP, clk);
    CNTR B (Bout, Bus, ldB, decB, clk);
    ADD AD (Z, X, Y);
    EQZ COMP (eqz, Bout);
endmodule

module PIPO1 (dout, din, ld, clk);
    input [15:0] din;
    input ld, clk;
    output reg [15:0] dout;
 
    always @(posedge clk)
        if (ld) dout <= din;
endmodule

module ADD (out, in1, in2);
    input [15:0] in1, in2;
    output reg [15:0] out;
 
    always @(*)
        out = in1 + in2;
endmodule

module PIPO2 (dout, din, ld, clr, clk);
    input [15:0] din;
    input ld, clr, clk;
    output reg [15:0] dout;
 
    always @(posedge clk)
        if (clr) dout <= 16'b0;
        else if (ld) dout <= din;
endmodule

module EQZ (eqz, data);
    input [15:0] data;
    output eqz;
 
    assign eqz = (data == 0);
endmodule

module CNTR (dout, din, ld, dec, clk);
    input [15:0] din;
    input ld, dec, clk;
    output reg [15:0] dout;
 
    always @(posedge clk)
        if (ld) dout <= din;
        else if (dec) dout <= dout - 1;
endmodule

module controller (ldA, ldB, ldP, clrP, decB, done, clk, eqz, start);
    input clk, eqz, start;
    output reg ldA, ldB, ldP, clrP, decB, done;
    reg [2:0] state;
    parameter S0 = 3'b000, S1 = 3'b001, S2 = 3'b010, S3 = 3'b011, S4 = 3'b100;
 
    always @(posedge clk)
    begin
        case (state)
            S0: if (start) state <= S1;
            S1: state <= S2;
            S2: state <= S3;
            S3: #2 if (eqz) state <= S4;
            S4: state <= S4;
            default: state <= S0;
        endcase
    end
 
    always @(state)
    begin 
        case (state)
            S0: begin #1 ldA = 0; ldB = 0; ldP = 0; clrP = 0; decB = 0; end
            S1: begin #1 ldA = 1; end
            S2: begin #1 ldA = 0; ldB = 1; clrP = 1; end
            S3: begin #1 ldB = 0; ldP = 1; clrP = 0; decB = 1; end
            S4: begin #1 done = 1; ldB = 0; ldP = 0; decB = 0; end
            default: begin #1 ldA = 0; ldB = 0; ldP = 0; clrP = 0; decB = 0; end
        endcase
    end
endmodule

// TEST BENCH 

module MUL_test;
    reg [15:0] data_in;
    reg clk, start;
    wire done;
 
    MUL_datapath DP (eqz, ldA, ldB, ldP, clrP, decB, data_in, clk);
    controller CON (ldA, ldB, ldP, clrP, decB, done, clk, eqz, start);
 
    initial
    begin
        clk = 1'b0;
        #3 start = 1'b1;
        #500 $finish;
    end
   
    always #5 clk = ~clk;
 
    initial
    begin
        #17 data_in = 17;
        #10 data_in = 5;
    end
  
    initial
    begin
        $monitor($time, " %d %b", DP.Y, done);
    end
endmodule
```

### Definations:

1. **MUL_datapath**: The main datapath module which instantiates other modules to handle different operations.
   - `PIPO1`, `PIPO2`: Parallel In Parallel Out registers.
   - `CNTR`: A counter module.
   - `ADD`: A simple adder module.
   - `EQZ`: A module to check if the output is zero.

2. **PIPO1**: A parallel-in, parallel-out register with a load enable.
3. **ADD**: Adds two 16-bit inputs and produces a 16-bit output.
4. **PIPO2**: Another parallel-in, parallel-out register with a clear and load enable.
5. **EQZ**: Compares if the input data is zero and sets the output accordingly.
6. **CNTR**: A counter that can load a value and decrement it.
7. **Controller**: Controls the state transitions and outputs the control signals based on the current state and inputs.

8. **MUL_test**: A testbench to simulate the entire datapath and controller. It initializes the clock and input values and monitors the output during simulation.

### Output

![image](https://github.com/KetanMe/Multiplication-by-Repeated-Addition-using-Verilog/assets/121623546/6073dace-22ba-4a59-a23d-227a5e2bdd10)


