

Circuit Verse Link: https://circuitverse.org/users/277779/projects/ece-366-project-3-b2ba4608-7e27-4f99-a99e-b5f3c5fb3902  
Name: NS - ISA
Goal: Make sure we are able to make alphabet count work within a 8bit ISA
Approach go based off of previous projects and HW to get ideas of what to do


 
Instructions
Functionality
Encoding Format
MC Example
asm Example
init Rx, imm
Rx = imm; PC++ imm[0, 15]
00 xx iiii
00011111
init r1, 15
lw Rx, Ry
Rx = Mem[Ry]; PC++
0100 xx yy
01001000
lw r1, r0
sw Rx, Ry
Mem[Ry] = Rx; PC++
0101 xx yy
01011011
sw r2, r3
add Rx, Ry
Rx = Rx + Ry; PC++
0110 xx yy
01101001
add r2, r1
sub Rx, Ry
Rx = Rx - Ry; PC++
0111 xx yy
011111111
sub r3, r3
addi Rx, imm
Rx = Rx + imm; PC++
1000 xx ii
10000001
addi r0, 1
mul Rx, imm
Rx = Rx * imm; PC++
1001 xx ii
10010110
mul r1, 2
sltR0 Rx, Ry
R0 = 0 if Rx  < Ry            R0 = 1 (otherwise); PC++
1010 xx yy
10100011
sltR0 r0, r3
bezR0
if (R0 == 0) PC = PC - imm else PC++ imm: [0, 63]
11 iiiiii
11111111
bezR0 63
Special Rx, imm
Explained below
1011 x iii
10111111
special r1, 7
halt
stop
01010101
01010101
halt



Special function (op =1011, to replace srl and sll)
	Ex: if 16 bit val in r1
R1 = 1111 0000 1010 0011		

	Special r1 , 1 
	r1->: 0000 0000 0000 1010

	Special r1, 0
	r1-> 0000 0000 0000 0011
	
	
Machine code
1111 
0000 
1010 
0011
index
3
2
1
0


	It basically works by knowing the index of the bit code and then knowing the index it would move that byte over to the first 4 bits and zero out the rest

Things done to achieve goal: compress functions to make every bit count, and also use a specialized function to get rid of unnecessary functions
Limitations: is the amount of instructions you could have, you can only do so much with a 8 bit isa since if you don't use your instructions to the max your going  to run out of functions you can use. Also for the Multiplication we only made it so it can multiply up to the value of 3 since we wouldn’t need anything more than that

The main compromise we did was our branch instruction, since its our own one and it takes up a lot of instruction space, what we did was only make it go backwards, meaning it can’t jump to instructions ahead of it only behead 

Logic for ever function (in Control Unit)



a
b
c
d
mux4
mux3
mux2
mux1
mux0
decode
real1
real2
func1
func2
func3
mux8
mux7
mux6
mux5
init
0
0
x
x
1
0
0
0
0
0
1
0
0
0
0
0
0
0
0
sw
0
1
0
1
0
0
0
0
0
1
0
1
0
0
0
0
0
0
0
addi
1
0
0
0
0
0
1
0
0
0
1
0
0
0
1
0
0
0
0
lw
0
1
0
0
0
0
0
0
1
0
1
0
0
0
0
0
0
0
0
add
0
1
1
0
0
0
0
1
0
0
1
0
0
1
0
0
0
0
0
sub
0
1
1
1
0
0
0
1
0
0
1
0
0
1
1
0
0
0
0
multiply
1
0
0
1
0
0
1
0
0
0
1
0
1
1
0
0
0
0
0
sltR0
1
0
1
0
0
1
0
1
0
0
1
0
1
0
0
0
0
0
0
beZr0
1
1
x
x
0
1
0
0
0
1
0
0
0
1
1
0
0
1
1
Special
1
0
1
1
0
0
0
0
0
0
1
0
1
0
1
1
1
0
0


Mux4 = a’b’
Mux3 = ab’cd’+ab
Mux2 = ab’c’d’+ab’c’d
Mux1 = a’bcd’ + a’bcd+ab’cd’
Mux0 = a’bc’d’
De = a’bc’d + ab
Real1 = not(a’bc’d+ab)
Real2 = a’bc’d
Func1 = ab’c’d+ab’cd’+ab’cd
Func2 = a’bcd’ + a’bcd + ab’c’d + ab
Func3 = ab’c’d’ + a’bcd + + ab + ab’cd
Mux8 = ab’cd
Mux7 = ab’cd
Mux6 = ab
Mux5 = ab
Branch = ab


Toy Test Case

// r1 = 15
Init r1, 15

// Mem[0] = 15 (r1)
Sw r1, r0
// r2 = Mem[0]
Lw r2, r0

// r1 = 15 (r1) + 15 (r2)
Add r1, r2

// r1 = 30 (r1) - 15 (r2)
Sub r1, r2

// 18 (r1) = 15 (r1) + 3
Addi r1, 3

// r3 = 2
Init r3, 2

// 36 (r1) = 18 (r1) * 2 (r3) 
Mul r1, r3

// get the second 4 bits of 36 (in binary)
// which is 2 (r1) = spec(36 (r1), 1)
Special r1, 1

// r0 = 1
Init r0, 1

// if r1 =< 1 (r0), make r0 = 0
// otherwise r0 = 1
// since r1 = 2, r0 = 1
sltR0, r1, r0 

// if r0 != 0 loop back to beginning
// because r0 is not =< 0, means PC = PC++
bezR0 11

// make r0 =0
Init r0, 0

// since r0 =< 0 loops for between this instruction and the previous 
Bezr0 1

// stop
halt

Hex code for Toy Test Case:

1F
54
48
66
76
87
32
97
B9
01
A4
CB
00
CD
55


Instruction for the Alphabet Count
___________________________

Init r3, 1
Init r2, 2
Init r1, 0

// M[0] = r3
Sw r3,  r1

// M[1] = r2
Addi r1, 1
Sw r2, r1	

// r1 = 2
Addi r1, 1
__________________________

Loop

// r2 = 2 
Init r2, 2

// r1 = r1 - r2 (r1 = 2 - 2 = 0)
Sub  r1, r2

// r2 = M[curr - 2]
Lw r2, r1

// r1 = r1 + 1 (r1 = 0 +1)
Addi r1, 1

// r3 = M[curr -1]
Lw r3, r1

// r2 = r2 - 2*r3
Mul r3, 2 	****
Sub r2, r3

// r1 = M[curr]
Addi r1, 1

// M[2] = r2
Sw r2, r1

// if r1 < 15 loop
Init r0, 15
slt r1, r0
Addi r1, 1
BeZR 12  
// reset registers
Init r0, 0
Init r1, 0
Init r2, 0
Init r3, 0

Loop2: 

// load the first 4 bits of the word (of whatever register r2 is holding
Lw r1, r2
Special r1, (0)

// if 4 bits aren’t a character set r0 = 0, if character r0 =1
Init r0 ,10
Slt r1, r0

// add to the count
Add r3, r0

// repeat above but next 4 bits (next byte) of the word
Lw r1, r2	
Special r1, (1)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (2)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (3)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (4)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (5)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (6)
Init r0 ,10
Slt, r1, 10
Add r3, r0

Lw r1, r2
Special r1, (7)
Init r0 ,10
Slt, r1, r0
Add r3, r0


// make r0 = 16
Init r0, 15
Add r0, 1
// add current index(value of r2) to 16 to get the correct memory data register
Add r0, r2
00001111
10000001
01100010

// store it M[curr index + 16] = r3 ( character count) 
Sw r3, r0
01011100

// if its not the end of the index set r0 = 1
Init r0, 15
Slt r2, r0

// add to the index
Addi r2, 1

// reset count
Init r3, 0 

// if r0 != 0 loop back
beZRO  48
halt
DONE

Machine Code (in hex)

Part1:

31
22
10
5D
85
59
85

Part2:

22
76
49
85
4D
9E
7B
85
59
0f
A4
85
Cc

Part3:

00
10
20
30
***********
46
B8
0A
A4 
6C

46
B9
0A
A4
6C

46
Ba
0A
A4
6C

46
Bb
0A
A4
6C

46
Bc
0A
A4
6C

46
Bd
0A
A4
6C

46
Be
0A
A4
6C

46
Bf
0A
A4
6C

************************

0f
81
62
5C
0f
a8
89
30
f0
55
