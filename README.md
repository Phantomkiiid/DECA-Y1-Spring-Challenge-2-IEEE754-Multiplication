# DECA-Y1-Spring-Challenge-2-IEEE754-Multiplication
This is the repo for my DECA Y1 Spring Term Challenge work (IEEE-754 format fast multiplication design).

# Design Inspirations
This multiplication aims to directly yield the result of multiplication between two IEEE-754 formatted numbers, `A(15:0)` and `B(15:0)`. Each number consists of three parts:
* Sign bit `X(15)`
* Exponent bits `X(14:10)`
* Mantissa (also called "significand" or "fraction") `X(9:0)`
<br>
The following diagram shows how an IEEE-754 formatted number could be intepreted: <br>
![floating_number_intepretation](https://github.com/user-attachments/assets/8e403b36-b68b-4344-9501-0cf677c41b06)




