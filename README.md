# DECA-Y1-Spring-Challenge-2-IEEE754-Multiplication
This is the repo for my DECA Y1 Spring Term Challenge work (IEEE-754 format fast multiplication design).

# Design Inspirations
This multiplication aims to directly yield the result of multiplication between two IEEE-754 formatted numbers, `A(15:0)` and `B(15:0)`. Each number consists of three parts:
* Sign bit `X(15)`
* Exponent bits `X(14:10)`
* Mantissa (also called "significand" or "fraction") `X(9:0)`
<br>
The following diagram shows how an IEEE-754 formatted number could be intepreted: <br>
![image](https://github.com/user-attachments/assets/d0efe06e-8b40-4d1b-918c-ec3f29d66329)



