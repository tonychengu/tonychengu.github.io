---
title: "Matrix Multiplication in FPGA"
excerpt: "Use System Verilog to implement a matrix multiplication module, and then use cocotb to automate unit testing with Python. <br/><img src='/images/rtl_view_matrix_mul.jpg'>"
collection: portfolio
---

# Matrix Multiplication

## Motivation
Matrix multiplication is a common operation in many applications, such as machine learning and image process. It also involves packing of input and output data, and sequential logic design. Therefore, it is a good practice for beginner to implement a matrix multiplication module in FPGA.  
The whole project can be found on Github [here](https://github.com/tonychengu/FPGA_matrix_multiplier/tree/main).

## Project outline
1. Using Python to realize a proof of concept. Verilog is very different from regular coding language. There's no console and everything you are dealing with is waveform and thus making it hard to debug. It's a common practice to use Python or C to verify the algorithm before implementing it in Verilog in order to maximize the efficiency.
2. Implementing the module in Verilog. This is the main part of the project. The module is implemented in System Verilog, which is a superset of Verilog. The main reason using System Verilog here is that it supports packing of output. With this ability, we could support "resizable" matrix multiplication. User could specify the size of input matrices and the module will automatically scale according to the given size.
3. Write a simple test bench in Verilog to verify the module is working as expected. This step aims to locate the obvious syntax or logic error in the module.Many simple bugs could arise just by compiling the file.
4. Use Python and cocotb to test extensively using randomized input. This could detect potential logical flaws in the design.

## Proof of concept - Python
Here is the python code. Since Verilog doesn't support 2D array very well, we map the input to a 1D array and do the operation according to index.
```python
def matrix_mul(A, B, Col1, Row1, Row2):
    C = Col1*Row2 * [0]
    for i in range(Col1):
        for j in range(Row1):
            for k in range(Row2):
                C[i*Row2+k] += A[i*Row1+j] * B[j*Row2+k]
    return C
```
Test it and it works as expected.  
![Python test img](/images/python_pof_test_matrix_mul.jpg)

## Verilog implementation
```verilog
`timescale 1ns / 1ps
module matrix_multiply #(
    parameter Col1 = 2,
    parameter Row1 = 2,
    parameter Row2 = 2
) (
    input [15:0] a[Col1*Row1-1:0],
    input [15:0] b[Row1*Row2-1:0],
    output logic [15:0] c[Col1*Row2-1:0]
);


  integer i, j, k;
  always @(*) begin
    for (i = 0; i < Col1; i=i+1) begin
      for (j = 0; j < Row1; j=j+1) begin
        for (k = 0; k < Row2; k=k+1) begin
          if (c[i*Row2+k] === 'z || c[i*Row2+k] === 'x) begin
            c[i*Row2+k] = 15'd0;
          end
          c[i*Row2+k] = c[i*Row2+k] + a[i*Row1+j] * b[j*Row2+k];
        end
      end
    end
  end
endmodule
```
It's almost the exact duplication of the python code logic. Let's do a simple test bench.
```verilog
module matrix_multiply_tb ();
  reg [15:0] a_in[0:3];
  reg [15:0] b_in[0:3];
  wire [15:0] c_out[0:3];
  matrix_multiply dut (.a(a_in), .b(b_in), .c(c_out));
  initial begin
    a_in[0] = 1;
    a_in[1] = 2;
    a_in[2] = 3;
    a_in[3] = 4;
    b_in[0] = 1;
    b_in[1] = 2;
    b_in[2] = 3;
    b_in[3] = 4;
    #3 $stop;
  end
endmodule
```
The `$stop` here tells the simulator to stop. Otherwise, it will run forever and crush when you run out of memory. Also, we need to noted that testbench data is not synthesizable on real Hardware.  
  
Here's the wave form of it. We use ModelSim to simulate the design and it works as expected.
![ModelSim Waveform](/images/matrix_mul_modelsim_wave.jpg)

## Automated testing with cocotb
cocotb is a Python framework for verifying digital logic in simulators and FPGA. It's a great tool for automated testing.
