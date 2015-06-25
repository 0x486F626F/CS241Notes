## Bits, Bytes, and Words

* Range of 2's Complement with $k$ bits $[-2^{k-1},2^{k-1}-1]$
* Range of unsigned int with $k$ bits $[0,2^k-1]$

e.g. $-25$ to an 8-bit 2's complement
```
00011001
11100110
00000001
--------
11100111
```

## Assembly Programs

input $\$1=1$ output $\$3=n!$

```assembly
add     $2, $1, $0
lis     $4
.word   1
add     $3, $4, $0
loop:
mult    $3, $2
mflo    $3
sub     $2, $2, $4
bne     $2, $4, loop
jr      $31
```

```assembly
factorial:
sw      $2, -4($30)
sw      $4, -8($30)
lis     $2
.word   8
sub     $30, $30, $2
add     $2, $1, $0
lis     $4
.word   1
add     $3, $4, $0
loop:
mult    $3, $2
mflo    $3
sub     $2, $2, $4
bne     $2, $4, loop
lis     $2
.word   8
add     $30, $30, $2
lw      $4, -8($30)
lw      $2, -4($30)
jr      $31
```

Hex notation for "beq \$20, \$13, -4"
```
0001 0010 1000 1101 1111 1111 1111 1100
124DFFFC
```

File A
```
.import abc
.word   0x18
.word   abc
```
File B
```
.export abc
.export def
.word   def
abc:def:
```
MERL
```
.word   0x10000002
.word   file len
.word   code len
.word   0x18
.word   0x18
.word   0x18
abc:def:
0x01
0x10
0x01
0x14
.export abc
.export def
```