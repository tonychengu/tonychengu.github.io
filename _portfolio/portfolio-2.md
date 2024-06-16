---
title: "sin, cos and CORDIC in FPGA"
excerpt: "Use Verilog to approximate sine and cosine value without using division. <br/><img src='/images/rtl_view_matrix_mul.jpg'>"
collection: portfolio
---

## Motivation
We have long been using sine and cosine function in our life and take it as granted on all those cheap calculators. However, it's not as simple as addition, subtraction and multiplication. On a limited device such FPGA, we cannot calculate the exact value of sine and cosine. Instead, we have to approximate it. This project aims to implement a sine and cosine module in FPGA using CORDIC algorithm.

## Project outline
1. Using Python to verify the algorithm, so that the logical structure of the algorithm is clear and accurate.
2. Using Verilog to implement the module.
3. Test it thoroughly and fix any bugs that arise.

## CORDIC algorithm
CORDIC algorithm is commonly used to approximate the value of sine and cosine. It's an iterative algorithm and it only uses simple modules such as adder, shift register, multipliers and comparators. Therefore, it's very suitable for FPGA implementation.
The general idea of the algorithm is that in a rectangle coordinate, you are rotating a vector starting from (1,0), which is angle 0, gradually to the desired angle. In order to rotate a vector, you need to times the vector with a rotation matrix.
A rotation matrix in general sense is:  
[cos(theta), -sin(theta)]  
[sin(theta), cos(theta)]  
However, in CORDIC algorithm, we don't really need to calculate the cosine and sine value of theta. Instead, we choose to rotate the vector by a fixed angle, which is called micro-rotation, and these angles are selected in a way that we only need to perform bit shift on x and y component of the vector to achieve the effect of timing a rotation matrix.

## Python Implementation

