# DECA-Y1-Spring-Challenge-2-IEEE754-Multiplication
This is the repo for my DECA Y1 Spring Term Challenge work (IEEE-754 format fast multiplication design).

# Design Inspirations
This multiplication aims to directly yield the result of multiplication between two IEEE-754 formatted numbers, `A(15:0)` and `B(15:0)`. Each number consists of three parts:
* Sign bit `X(15)`
* Exponent bits `X(14:10)`
* Mantissa (also called "significand" or "fraction") `X(9:0)`
<br>

![floating_number_intepretation](https://github.com/user-attachments/assets/1ee98fab-c44a-4a75-96a3-aef1068300b3)

The above diagram shows how can an IEEE-754 half precision floating number could be intepreted. In order to calculate the product between `A` and `B`, we need to proceed as follows.
* Step 1: calculate sign bit, which is `A(15) XOR B(15)`;
* Step 2: calculate exponent, which is `A(14:10) + B(14:10) - 15(decimal)`, note that this could be done using **two's complement** (i.e. `+ 0b10001`);
* Step 3: perform partial product calculation in mantissa, which is `A(9:0) * B(9:0)`, using a similar method comparing to Challenge 1, adding the hidden "1" at the MSB;
* Step 4: check the decimal point in mantissa and adjust exponent (`exponent + 1`);
* Step 5: combine the sign bit, exponent bits and mantissa (**Most Significant** 10 bits **after truncating** the leading `001` in `Rm`) for output. <br>

The Following figure shows the detailed process of dealing with mantissa calculation and normalization. <br>
![image](https://github.com/user-attachments/assets/672a50d9-b4aa-4b8b-9251-4cf8516eea21)

# New Hardware Included
* `ADD667`, similar to what is included in Challenge 1, outputs the sum of two 6-bit numbers with carry bit being the MSB.
  ![image](https://github.com/user-attachments/assets/54e6b0bb-ea2a-4cf2-b13b-456ab86c4adb)
* `MUL6`, which gives a 12-bit multiplication result between two 6-bit numbers. It uses five `ADD667` so costs 30 full adders.
  ![image](https://github.com/user-attachments/assets/15c16e25-b0b4-4063-8340-5703492c3586)
* `MLU`, short for `Multiplication Unit`, handles complex processing of numbers。 It uses two temporary 15-bit registers and one 24-bit `PPL` (short for `Partial Product Register`) to store mantissa, as well as two additional 5-bit adders (so 10 full adders used here).
  ![image](https://github.com/user-attachments/assets/2f13150c-2977-4d08-af35-ac4e6dd8157a)
* New Muxes and control signals are added to the `datapath` and `dpdecode` sheets for additional instructions.
  ![image](https://github.com/user-attachments/assets/e340e4f5-e4b0-4185-b134-6730ce42bec3)

# New Instructions Defined
Note: in the following comments, `RxL` means `Rx(5:0)` and `RxH` means `Rx(11:6)`. <br>

* `MOVC1 Ra, Rb` &emsp;  // `PPR(5:0) := RaL * RbL`; `Ra(15:0) := 0x0(15:6) + {{RaL * RbL}(11:6)}(5:0)` <br>
* `MOVC2 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaL * RbH` <br>
* `MOVC3 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaH * RbL` <br>
* `MOVC4 Ra, Rb` &emsp;  // `PPR(11:6) := Ra(5:0)`; `Ra(15:0) := 0x0(15:7) + {Ra(12:6)}(6:0)` -> updates PPR with the newly calculated bits; takes carry bit into account <br>
* `MOVC5 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaH * RbH` <br>
* `MOVC6 Ra, Rb` &emsp;  // `PPR(23:12) := Ra(11:0)` -> get overall mantissa product <br>
* `MOVC7 Ra, Rb` &emsp;  // `Ra(15:0) := sign || exp || normalized mantissa`, where `||` means "concatenate"

**Subroutine using new instructions** <br>
The following subroutine outputs the desired IEEE-754 formatted product correctly.

* `0x00` `MOV Ra, #num_1` <br>
* `0x01` `MOV Rb, #num_2` <br>
* `0x02` `MOVC1 Ra, Rb` &emsp;&ensp;  // `PPR(5:0)` written; `Ra` loaded with `0000 0000 00XX XXXX` <br>
* `0x03` `MOVC2 Rb, Ra` &emsp;&ensp;  // `Rb` loaded with `0000 XXXX XXXX XXXX` from `MUL` <br>
* `0x04` `ADD Ra, Ra, Rb` &emsp;  // sum updated to `Ra` <br>
* `0x05` `MOVC3 Rb, Ra` &emsp;&ensp;  // `Rb` loaded with `0000 XXXX XXXX XXXX` from `MUL` <br>
* `0x06` `ADD Ra, Ra, Rb` &emsp;  // sum updated to `Ra`, which is the correct bit sequence of the final mantissa product from the 6th to the 11th bit <br>
* `0x07` `MOVC4 Ra, Rb` &emsp;&ensp;  // `PPL` has its 6th to 11th bits updated to `Ra`; meanwhile, `Ra` has its internal bits shifted 5 positions rightwards (to account for any MSB carry) <br>
* `0x08` `MOVC5 Rb, Ra` &emsp;&ensp;   // `Rb` loaded with `0000 XXXX XXXX XXXX` from `MUL` (which is `AH * BH`) <br>
* `0x09` `ADD Ra, Ra, Rb` &emsp;  // the content of `Ra` is the correct bit sequence of the final mantissa product from the 12th to the 23rd bit <br>
* `0x0A` `MOVC6 Rb, Ra` &emsp;&ensp;  // normalizes the mantissa product and adjusting exponent <br>
* `0x0B` `MOVC7 Ra, Rb` &emsp;&ensp;  // assembles and outputs the final IEEE-754 formatted product 

Hence, if not considering the two `MOV` instructions at the start, the overall multiplication process would take **10 clock cycles** to execute for **any 16-bit IEEE-754 number**, yielding a product in the same format.

# Example Instructions
`0x00` `EXT 0xDB` <br>
`0x01` `MOV R1, #0xF7` <br>
`0x02` `EXT 0x3A` <br>
`0x03` `MOV R2, #0xA6` <br>
`0x04` `MOVC1 R1, R2` <br>
`0x05` `MOVC2 R2, R1` <br>
`0x06` `ADD R1, R1, R2` <br>
`0x07` `MOVC3 R2, R1` <br>
`0x08` `ADD R1, R1, R2` <br>
`0x09` `MOVC4 R1, R2` <br>
`0x0A` `MOVC5 R2, R1` <br> 
`0x0B` `ADD R1, R1, R2` <br>
`0x0C` `MOVC6 R1, R2` <br>
`0x0D` `MOVC7 R1, R2` <br>

**Expected Result** <br>
The two numbers in this example are `0xDBF7` and `0x3AA6` respectively (refer to the following figures for decimal intepretation). 
* `0xDBF7` equals `-254.875` in decimal.
![image](https://github.com/user-attachments/assets/7d26de1c-65a7-46cc-bd3f-27a5cae42d8c)
* `0x3AA6` equals `0.8310546875` in decimal.
![image](https://github.com/user-attachments/assets/4f9d4825-5cb0-46bb-b18f-c6523bf60de5)
* Hand-calculated result is `0xDA9E`, which equals `-211.75` in decimal.
![image](https://github.com/user-attachments/assets/b8cd1b42-2a61-4fb4-b4c3-efbd91673003)
* When putting into a calculator, `-254.875 * 0.8310546875 = -211.8150635`, leading to a difference of approximately 0.03%, which is considered accurate.
* Issie waveform simulation yields the same correct result (`0xDA9E` at clock cycle 14 in `REG1`).
![image](https://github.com/user-attachments/assets/a1a1293e-fdf8-46e1-938b-0420fd40850e)


# Evaluation
This combined hardware and software implementation is successful in processing IEEE-754 standard multiplication. It has the following features:
* It uses a total of **40 full adders**, which is significantly lower than the limitation of 64 full adders.
* It could be used to deal with **signed** IEEE-754 formatted numbers in half precision.
* It takes a total of **10 clock cycles** to implement the full subroutine, which is considered fast.
* If the number of full adders is not limited, it is possible to compress the overall subroutine - or to even make a single-instruction implementation.
However, it has the following limitations:
* It could not yield an **exact** representation of the product due to bits limitations;
* It uses multiple MUXes which could also be considered as increased hardware costs.









