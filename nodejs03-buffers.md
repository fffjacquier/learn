## Buffers

Why are Buffers so important?

Buffers connect NodeJs to all the binary machine code so we can deal with files and network requests, it gives us access to the low level thing.

### Binary numbers (base 2 numbers)

1 bit can contain 0 or 1
1 byte = 8 bits grouped together

00001011 is a number
1011 is, starting from the right:
= 1 x 2⁰ + 1 x 2¹ + 0 x 2² + 1 x 2³
= 1 + 2 + 0 + 8 = 11

11001011
the last 1 is the last significant Digit or bit (LSD / LSB)
the first 1 is the most significant bit or digit (MSB / MSD)

### Hexadecimal numbers (base 16 numbers)

Conversion is simple between binary and hex
Every four bits there is one hexadecimal character

(0x)456 in base 16
0x is an indicator that this is hexadecimal (convention)
= 6 x 16⁰ = 6
= 5 x 16¹ = 80
= 4 x 16² = 1024
= 1110

hex numbers: case insensitive
0 1 2 3 4 5 6 7 8 9 A B C D E F (for 10 to 15)

¹²³⁴⁵⁶⁷⁸⁹⁰
fa3c:
= 12 x 16⁰ = 12
= 3 x 16¹ = 48
= 10 x 16² = 2560
= 15 x 16³ = 61440
= 64060

Compare 0xffffff vs 16777215 vs 24chars 1 in binary
only 6 chars for hex, so it's easy to deal with

0101 0101 0111 1101 0101 1111 0000 0001
yuo can use a table hex/bin to simplify this number

0 0000
1 0001
2 0010
3 0011
4 0100
5 0101
6 0110
7 0111
8 1000
9 1001
A 1010
B 1011
C 1100
D 1101
E 1110
F 1111
so previous number is 557D 5F01

indicators: #, %, &#x

### charsets and encoding

Character sets are standards

**Unicode** is a standard for representing and encoding characters in most writing systems worldwide

Defines 149 813 chars in version 15.1
for ex. the char M is 115

**ASCII** defines 128 chars (lowercase, uppercase, numbers, punct, symbols and some control char lile newline or delete) -> it's only for english language

> man ascii (to see the list of ascii chars)

you can see that numbers are also defined (hexa set: 1 is 31)
