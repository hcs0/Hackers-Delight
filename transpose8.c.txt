// Programs for computing the transpose of an 8x8 bit matrix.
// Max line length is 57 (sometimes), to fit in hacker.book.
// This has been tested on both AIX/xlc and Windows/gcc, and
// so is believed to be independent of endian mode and whether
// char is by default signed or unsigned.
#include <stdio.h>
#include <string.h>

/* All the functions herein are for transposing an 8x8 bit matrix, that
is presumed to be a block of a larger matrix of size m x n bytes. Brief
description of the functions:

transpose8vn:    Very naive method, directly places one bit at a time.
transpose8b64:   Basic shifting method that directly places a few bits at a time.
transpose8b64c:  Compact version of 8b64; uses a for-loop.
transpose8bS64:  Like 8b64, but uses GLS's bit swapping device.
transpose8r64:   Basic recursive method (the main point of this comparison study).
transpose8rS64:  Like 8r64, but uses GLS's bit swapping device.
transpose8b32, ..., 8rS32: Above four coded for a 32-bit machine.
transpose8rSr32: Like 8rS32 but done in reverse (coarse to fine granularity). */


/* -------------------------- transpose8vn -------------------------- */

/* This is the very naive method, that directly places one bit at a
time. This may be too naive to include in the book (i.e., maybe we
should take this out if there should be another edition).
   This is equally suitable (or unsuitable, if you wish) for a 32- or a
64-bit machine.
   Instruction counts for the calculation part:
   62 ANDs
   56 shifts
   56 ORs (or ADDs or XORs)
   --
  174 total (very naive method, placing one bit at a time) */

void transpose8vn(unsigned char A[8], int m, int n, unsigned char B[8]) {
   unsigned char a0, a1, a2, a3, a4, a5, a6, a7,
                 b0, b1, b2, b3, b4, b5, b6, b7;

   // Load the array into eight one-byte variables.

   a0 = A[0];  a1 = A[m];  a2 = A[2*m];  a3 = A[3*m];
   a4 = A[4*m];  a5 = A[5*m];  a6 = A[6*m];  a7 = A[7*m];

   b0 = (a0 & 128)    | (a1 & 128)/2  | (a2 & 128)/4  | (a3 & 128)/8 |
        (a4 & 128)/16 | (a5 & 128)/32 | (a6 & 128)/64 | (a7      )/128;
   b1 = (a0 &  64)*2  | (a1 &  64)    | (a2 &  64)/2  | (a3 &  64)/4 |
        (a4 &  64)/8  | (a5 &  64)/16 | (a6 &  64)/32 | (a7 &  64)/64;
   b2 = (a0 &  32)*4  | (a1 &  32)*2  | (a2 &  32)    | (a3 &  32)/2 |
        (a4 &  32)/4  | (a5 &  32)/8  | (a6 &  32)/16 | (a7 &  32)/32;
   b3 = (a0 &  16)*8  | (a1 &  16)*4  | (a2 &  16)*2  | (a3 &  16)   |
        (a4 &  16)/2  | (a5 &  16)/4  | (a6 &  16)/8  | (a7 &  16)/16;
   b4 = (a0 &   8)*16 | (a1 &   8)*8  | (a2 &   8)*4  | (a3 &   8)*2 |
        (a4 &   8)    | (a5 &   8)/2  | (a6 &   8)/4  | (a7 &   8)/8;
   b5 = (a0 &   4)*32 | (a1 &   4)*16 | (a2 &   4)*8  | (a3 &   4)*4 |
        (a4 &   4)*2  | (a5 &   4)    | (a6 &   4)/2  | (a7 &   4)/4;
   b6 = (a0 &   2)*64 | (a1 &   2)*32 | (a2 &   2)*16 | (a3 &   2)*8 |
        (a4 &   2)*4  | (a5 &   2)*2  | (a6 &   2)    | (a7 &   2)/2;
   b7 = (a0      )*128| (a1 &   1)*64 | (a2 &   1)*32 | (a3 &   1)*16|
        (a4 &   1)*8  | (a5 &   1)*4  | (a6 &   1)*2  | (a7 &   1);

   B[0] = b0;    B[n] = b1;    B[2*n] = b2;  B[3*n] = b3;
   B[4*n] = b4;  B[5*n] = b5;  B[6*n] = b6;  B[7*n] = b7;
}

/* The above executes in 174 instructions (just the calculation part).
62 ANDs, 56 shifts, and 56 ORs. */


/* ------------------------- transpose8b64 -------------------------- */

/* transpose8b64 directly places the bits in the target array. It uses
64-bit quantities, which makes it easy to understand. It can be easily
translated for execution on a 32-bit machine, either by hand or by
letting the compiler do it, if your compiler supports 64-bit integers on
a 32-bit machine.
   It is based on the observation that the bits in the 64-bit doubleword
all move either 0, 7, 14, 21, 28, 35, 42, or 49 positions to the left or
right. This is illustrated by the diagram below. Each digit, letter, $,
or period represents a single bit in a 64-bit word. A dash represents a
0, resulting from the shift instructions.
   Looking at the Input and Output lines, the bit denoted by the
character '0' does not move, '1' moves 7 positions to the right, '2'
moves 14 positions to the right, etc.
   Note: Rotate shifts do not help. That is, they do not move any
additional bits to their output positions.

Input x: 01234567 89abcdef ghijklmn opqrstuv wxyzABCD EFGHIJKL MNOPQRST UVWXYZ$.
Output:  08gowEMU 19hpxFNV 2aiqyGOW 3bjrzHPX 4cksAIQY 5dltBJRZ 6emuCKS$ 7fnvDLT.

x:       01234567 89abcdef ghijklmn opqrstuv wxyzABCD EFGHIJKL MNOPQRST UVWXYZ$.
x <<  7: 789abcde fghijklm nopqrstu vwxyzABC DEFGHIJK LMNOPQRS TUVWXYZ$ .-------
x << 14: efghijkl mnopqrst uvwxyzAB CDEFGHIJ KLMNOPQR STUVWXYZ $.------ --------
x << 21: lmnopqrs tuvwxyzA BCDEFGHI JKLMNOPQ RSTUVWXY Z$.----- -------- --------
x << 28: stuvwxyz ABCDEFGH IJKLMNOP QRSTUVWX YZ$.---- -------- -------- --------
x << 35: zABCDEFG HIJKLMNO PQRSTUVW XYZ$.--- -------- -------- -------- --------
x << 42: GHIJKLMN OPQRSTUV WXYZ$.-- -------- -------- -------- -------- --------
x << 49: NOPQRSTU VWXYZ$.- --------- ------- -------- -------- -------- --------
x >>  7: -------0 12345678 9abcdefg hijklmno pqrstuvw xyzABCDE FGHIJKLM NOPQRSTU
x >> 14: -------- ------01 23456789 abcdefgh ijklmnop qrstuvwx yzABCDEF GHIJKLMN
x >> 21: -------- -------- -----012 3456789a bcdefghi jklmnopq rstuvwxy zABCDEFG
x >> 28: -------- -------- -------- ----0123 456789ab cdefghij klmnopqr stuvwxyz
x >> 35: -------- -------- -------- -------- ---01234 56789abc defghijk lmnopqrs
x >> 42: -------- -------- -------- ----------------- -0123456 789abcde fghijklm
x >> 49: -------- -------- -------- ----------------- -------- 01234567 89abcdef

The function below positions some of the bits with an expression of the
form (x & mask) << s, and some with an expression of the form (x >> s) &
mask. This is reduces the number of distinct masks that are required.
   Instruction counts for the calculation part, for a 64-bit machine:
   14 shifts
   15 ANDs
   14 ORs (or ADDs or XORs)
    9 Mask generation (4 for the first and 1 for each subsequent
      one, except the two smallest masks can be immediate fields).
   --
   52 total (64-bit machine, direct placement) */

void transpose8b64(unsigned char A[8], int m, int n,
                   unsigned char B[8]) {
   unsigned long long x, y;
   int i;

   for (i = 0; i <= 7; i++)     // Load 8 bytes from the
      x = x << 8 | A[m*i];      // input array and pack
                                // them into x.

   y =  x & 0x8040201008040201LL        |
       (x & 0x0080402010080402LL) <<  7 |
       (x & 0x0000804020100804LL) << 14 |
       (x & 0x0000008040201008LL) << 21 |
       (x & 0x0000000080402010LL) << 28 |
       (x & 0x0000000000804020LL) << 35 |
       (x & 0x0000000000008040LL) << 42 |
       (x & 0x0000000000000080LL) << 49 |
       (x >>  7) & 0x0080402010080402LL |
       (x >> 14) & 0x0000804020100804LL |
       (x >> 21) & 0x0000008040201008LL |
       (x >> 28) & 0x0000000080402010LL |
       (x >> 35) & 0x0000000000804020LL |
       (x >> 42) & 0x0000000000008040LL |
       (x >> 49) & 0x0000000000000080LL;

   for (i = 7; i >= 0; i--) {   // Store result into
      B[n*i] = y; y = y >> 8;}  // output array B.
}


/* ------------------------- transpose8b64c ------------------------- */

/* This is a compact version of transpose8b64, ca. 75 instructions for
the calculation part. */

void transpose8b64c(unsigned char A[8], int m, int n,
                    unsigned char B[8]) {
   unsigned long long x, y, mask;
   int i, s;

   for (i = 0; i <= 7; i++)     // Load 8 bytes from the
      x = x << 8 | A[m*i];      // input array and pack
                                // them into x.
   mask = 0x8040201008040201LL;
   y = x & mask;

   for (s = 7; s <= 49; s = s + 7) {
      mask = mask >> 8;
      y = y | ((x & mask) << s) | ((x >> s) & mask);
   }

   for (i = 7; i >= 0; i--) {   // Store result into
      B[n*i] = y; y = y >> 8;}  // output array B.
}

/* ------------------------- transpose8bS64 ------------------------- */

/* This is transpose8b64 but using the GLS method of bit field swapping.
   Instruction counts for the calculation part:
    7 ANDs
   21 XORs
   14 shifts
    8 Mask generation (many can be generated from earlier masks)
   --
   50 total (direct placement method for a 64-bit machine, using GLS's bit swapping) */

void transpose8bS64(unsigned char A[8], int m, int n,
                    unsigned char B[8]) {
   unsigned long long x, t;
   int i;

   for (i = 0; i <= 7; i++)     // Load 8 bytes from the
      x = x << 8 | A[m*i];      // input array and pack
                                // them into x.

   t = (x ^ (x >>  7)) & 0x0080402010080402LL;
   x = x ^ t ^ (t <<  7);
   t = (x ^ (x >> 14)) & 0x0000804020100804LL;
   x = x ^ t ^ (t << 14);
   t = (x ^ (x >> 21)) & 0x0000008040201008LL;
   x = x ^ t ^ (t << 21);
   t = (x ^ (x >> 28)) & 0x0000000080402010LL;
   x = x ^ t ^ (t << 28);
   t = (x ^ (x >> 35)) & 0x0000000000804020LL;
   x = x ^ t ^ (t << 35);
   t = (x ^ (x >> 42)) & 0x0000000000008040LL;
   x = x ^ t ^ (t << 42);
   t = (x ^ (x >> 49)) & 0x0000000000000080LL;
   x = x ^ t ^ (t << 49);

   for (i = 7; i >= 0; i--) {   // Store result into
      B[n*i] = x; x = x >> 8;}  // output array B.
}


/* ------------------------- transpose8r64 -------------------------- */

/* transpose8r64 is the basic recursive method for a 64-bit machine.
This function positions some of the bits with an expression of the
form (x & mask) << s, and some with an expression of the form (x >> s) &
mask. This is reduces the number of distinct masks that are required.
   Instruction counts for the calculation part, for a 64-bit machine:
    6 shifts
    9 ANDs
    6 ORs (or ADDs or XORs)
   17 Mask generation
   --
   38 total (64-bit machine, basic recursive method) */

void transpose8r64(unsigned char A[8], int m, int n,
                   unsigned char B[8]) {
   unsigned long long x;
   int i;

   for (i = 0; i <= 7; i++)     // Load 8 bytes from the
      x = x << 8 | A[m*i];      // input array and pack
                                // them into x.

   x =  x & 0xAA55AA55AA55AA55LL        |
       (x & 0x00AA00AA00AA00AALL) <<  7 |
       (x >> 7) & 0x00AA00AA00AA00AALL;
   x =  x & 0xCCCC3333CCCC3333LL        |
       (x & 0x0000CCCC0000CCCCLL) << 14 |
       (x >> 14) & 0x0000CCCC0000CCCCLL;
   x =  x & 0xF0F0F0F00F0F0F0FLL        |
       (x & 0x00000000F0F0F0F0LL) << 28 |
       (x >> 28) & 0x00000000F0F0F0F0LL;

   for (i = 7; i >= 0; i--) {   // Store result into
      B[n*i] = x; x = x >> 8;}  // output array B.
}


/* ------------------------- transpose8rS64 -------------------------- */

/* This is transpose8r64 but using the GLS method of bit field swapping.
   Instruction counts for the calculation part, for a 64-bit machine:
    6 shifts
    3 ANDs
    9 XORs
    8 Mask generation
   --
   26 total (64-bit machine, recursive method with GLS bit swapping) */

void transpose8rS64(unsigned char A[8], int m, int n,
                    unsigned char B[8]) {
   unsigned long long x, t;
   int i;

   for (i = 0; i <= 7; i++)     // Load 8 bytes from the
      x = x << 8 | A[m*i];      // input array and pack
                                // them into x.

   t = (x ^ (x >> 7)) & 0x00AA00AA00AA00AALL;
   x = x ^ t ^ (t << 7);
   t = (x ^ (x >> 14)) & 0x0000CCCC0000CCCCLL;
   x = x ^ t ^ (t << 14);
   t = (x ^ (x >> 28)) & 0x00000000F0F0F0F0LL;
   x = x ^ t ^ (t << 28);

   for (i = 7; i >= 0; i--) {   // Store result into
      B[n*i] = x; x = x >> 8;}  // output array B.
}


/* ------------------------- transpose8b32 -------------------------- */

/* This is transpose8b64 adapted to a 32-bit machine more-or-less
mechanically. Because of the double-length shifts, some of the terms of
the from (x & mask) << n were changed to (x << n) & mask'. Then, for
consistency, all were changed to that form.
   Instruction counts for the calculation part, for a 32-bit machine:
   26 shifts
   22 ANDs
   26 ORs (or ADDs or XORs)
   10 Mask generation (many can be generated from earlier masks)
   --
   84 total (32-bit machine, direct placement) */

void transpose8b32(unsigned char A[8], int m, int n,
                   unsigned char B[8]) {
   unsigned int xh, xl, yh, yl;         // x high, low, etc.

   // Load the array and pack it into xh and xl.

   xh = (A[0]<<24)   | (A[m]<<16)   | (A[2*m]<<8) | A[3*m];
   xl = (A[4*m]<<24) | (A[5*m]<<16) | (A[6*m]<<8) | A[7*m];

   yh = xh                   & 0x80402010 |
       (xh <<  7 | xl >> 25) & 0x40201008 |
       (xh << 14 | xl >> 18) & 0x20100804 |
       (xh << 21 | xl >> 11) & 0x10080402 |
       (xh << 28 | xl >>  4) & 0x08040201 |
       (           xl <<  3) & 0x04020100 |
       (           xl << 10) & 0x02010000 |
       (           xl << 17) & 0x01000000 |
       (xh >>  7           ) & 0x00804020 |
       (xh >> 14           ) & 0x00008040 |
       (xh >> 21           ) & 0x00000080;

   yl = xl                   & 0x08040201 |
       (xl <<  7           ) & 0x04020100 |
       (xl << 14           ) & 0x02010000 |
       (xl << 21           ) & 0x01000000 |
       (xh << 25 | xl >>  7) & 0x10080402 |
       (xh << 18 | xl >> 14) & 0x20100804 |
       (xh << 11 | xl >> 21) & 0x40201008 |
       (xh <<  4           ) & 0x80402010 |
       (xh >>  3           ) & 0x00804020 |
       (xh >> 10           ) & 0x00008040 |
       (xh >> 17           ) & 0x00000080;

   B[0]=yh>>24;    B[n]=yh>>16;    B[2*n]=yh>>8;  B[3*n]=yh;
   B[4*n]=yl>>24;  B[5*n]=yl>>16;  B[6*n]=yl>>8;  B[7*n]=yl;
}


/* ------------------------- transpose8bS32 ------------------------- */

/* This is transpose8b32 but using the GLS method of bit field swapping.
   Instruction counts for the calculation part, for a 32-bit machine:
   27 shifts
   10 ANDs
    6 ORs (or ADDs or XORs)
   31 XORs
    7 Mask generation (many can be generated from earlier masks)
   --
   81 total (32-bit machine, direct placement with GLS bit swapping) */

void transpose8bS32(unsigned char A[8], int m, int n,
                    unsigned char B[8]) {
   unsigned int xh, xl, th, tl;         // x high, low, etc.

   // Load the array and pack it into xh and xl.

   xh = (A[0]<<24)   | (A[m]<<16)   | (A[2*m]<<8) | A[3*m];
   xl = (A[4*m]<<24) | (A[5*m]<<16) | (A[6*m]<<8) | A[7*m];

   th = (xh ^ (xh >> 7)) & 0x00804020;
   tl = (xl ^ (xl >> 7 | xh << 25)) & 0x10080402;
   xh = xh ^ th ^ (th << 7 | tl >> 25);
   xl = xl ^ tl ^ (tl << 7);

   th = (xh ^ (xh >> 14)) & 0x00008040;
   tl = (xl ^ (xl >> 14 | xh << 18)) & 0x20100804;
   xh = xh ^ th ^ (th << 14 | tl >> 18);
   xl = xl ^ tl ^ (tl << 14);

   th = (xh ^ (xh >> 21)) & 0x00000080;
   tl = (xl ^ (xl >> 21 | xh << 11)) & 0x40201008;
   xh = xh ^ th ^ (th << 21 | tl >> 11);
   xl = xl ^ tl ^ (tl << 21);

   tl = (xl ^ (xh << 4)) & 0x80402010;
   xh = xh ^ (tl >> 4);
   xl = xl ^ tl ^ (tl << 28);

   tl = (xl ^ (xh >> 3)) & 0x00804020;
   xh = xh ^ (tl << 3);
   xl = xl ^ tl;

   tl = (xl ^ (xh >> 10)) & 0x00008040;
   xh = xh ^ (tl << 10);
   xl = xl ^ tl;

   tl = (xl ^ (xh >> 17)) & 0x00000080;
   xh = xh ^ (tl << 17);
   xl = xl ^ tl;

   B[0]=xh>>24;    B[n]=xh>>16;    B[2*n]=xh>>8;  B[3*n]=xh;
   B[4*n]=xl>>24;  B[5*n]=xl>>16;  B[6*n]=xl>>8;  B[7*n]=xl;
}


/* ------------------------- transpose8r32 -------------------------- */

/* Next is the basic "recursive" method. Decided not to include this in
HD. It's too similar to transpose8rS32, which is a little better (probably).
   Instruction counts for the calculation part:
   16 ANDs
   10 shifts
   10 ORs
    9 mask generation
   --
   45 total (recursive method, direct placement at each step) */

void transpose8r32(unsigned char A[8], int m, int n,
                   unsigned char B[8]) {
   unsigned x, y, t;

   // Load the array and pack it into x and y.

   x = (A[0]<<24)   | (A[m]<<16)   | (A[2*m]<<8) | A[3*m];
   y = (A[4*m]<<24) | (A[5*m]<<16) | (A[6*m]<<8) | A[7*m];

   x = (x & 0xAA55AA55) | ((x & 0x00AA00AA) << 7) |
       ((x >> 7) & 0x00AA00AA);
   y = (y & 0xAA55AA55) | ((y & 0x00AA00AA) << 7) |
       ((y >> 7) & 0x00AA00AA);

   x = (x & 0xCCCC3333) | ((x & 0x0000CCCC) << 14) |
       ((x >> 14) & 0x0000CCCC);
   y = (y & 0xCCCC3333) | ((y & 0x0000CCCC) << 14) |
       ((y >> 14) & 0x0000CCCC);

   t = (x & 0xF0F0F0F0) | ((y >> 4) & 0x0F0F0F0F);
   y = ((x << 4) & 0xF0F0F0F0) | (y & 0x0F0F0F0F);
   x = t;

   B[0]=x>>24;    B[n]=x>>16;    B[2*n]=x>>8;  B[3*n]=x;
   B[4*n]=y>>24;  B[5*n]=y>>16;  B[6*n]=y>>8;  B[7*n]=y;
}


/* ------------------------- transpose8rS32 ------------------------- */

/* This is transpose8r32 but using the GLS method of bit field swapping.
   Instruction counts for the calculation part:
    8 ANDs
   12 XORs
   10 shifts
    2 ORs
    5 mask generation
   --
   37 total (recursive method using GLS's bit swapping) */

void transpose8rS32(unsigned char A[8], int m, int n,
                 unsigned char B[8]) {
   unsigned x, y, t;

   // Load the array and pack it into x and y.

   x = (A[0]<<24)   | (A[m]<<16)   | (A[2*m]<<8) | A[3*m];
   y = (A[4*m]<<24) | (A[5*m]<<16) | (A[6*m]<<8) | A[7*m];

   t = (x ^ (x >> 7)) & 0x00AA00AA;  x = x ^ t ^ (t << 7);
   t = (y ^ (y >> 7)) & 0x00AA00AA;  y = y ^ t ^ (t << 7);

   t = (x ^ (x >>14)) & 0x0000CCCC;  x = x ^ t ^ (t <<14);
   t = (y ^ (y >>14)) & 0x0000CCCC;  y = y ^ t ^ (t <<14);

   t = (x & 0xF0F0F0F0) | ((y >> 4) & 0x0F0F0F0F);
   y = ((x << 4) & 0xF0F0F0F0) | (y & 0x0F0F0F0F);
   x = t;

   B[0]=x>>24;    B[n]=x>>16;    B[2*n]=x>>8;  B[3*n]=x;
   B[4*n]=y>>24;  B[5*n]=y>>16;  B[6*n]=y>>8;  B[7*n]=y;
}


/* ------------------------ transpose8rSr32 ------------------------- */

/* This is transpose8rS32 done "backwards" (coarse to fine granularity).
Why? Just to show that this works.
   Instruction counts for the calculation part:
    8 ANDs
   12 XORs
   10 shifts
    2 ORs
    5 mask generation
   --
   37 total (recursive method in reverse, using GLS's bit swapping) */

void transpose8rSr32(unsigned char A[8], int m, int n,
                 unsigned char B[8]) {
   unsigned x, y, t;

   // Load the array and pack it into x and y.

   x = (A[0]<<24)   | (A[m]<<16)   | (A[2*m]<<8) | A[3*m];
   y = (A[4*m]<<24) | (A[5*m]<<16) | (A[6*m]<<8) | A[7*m];

   t = (x & 0xF0F0F0F0) | ((y >> 4) & 0x0F0F0F0F);
   y = ((x << 4) & 0xF0F0F0F0) | (y & 0x0F0F0F0F);
   x = t;

   t = (x ^ (x >>14)) & 0x0000CCCC;  x = x ^ t ^ (t <<14);
   t = (y ^ (y >>14)) & 0x0000CCCC;  y = y ^ t ^ (t <<14);

   t = (x ^ (x >> 7)) & 0x00AA00AA;  x = x ^ t ^ (t << 7);
   t = (y ^ (y >> 7)) & 0x00AA00AA;  y = y ^ t ^ (t << 7);

   B[0]=x>>24;    B[n]=x>>16;    B[2*n]=x>>8;  B[3*n]=x;
   B[4*n]=y>>24;  B[5*n]=y>>16;  B[6*n]=y>>8;  B[7*n]=y;
}

// ----------------------------- error ---------------------------------

int errors;
void error(unsigned char T[8], unsigned char R[8]) {
   errors = errors + 1;
   printf("Error for test = %02x%02x%02x%02x %02x%02x%02x%02x, got "
      "%02x%02x%02x%02x %02x%02x%02x%02x\n",
      T[0],T[1],T[2],T[3],T[4],T[5],T[6],T[7],
      R[0],R[1],R[2],R[3],R[4],R[5],R[6],R[7]);

}

// ------------------------------ main ---------------------------------

int main() {
   int i, n;
   static unsigned char test[][8] = {
      {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00}, {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00},
      {0x80,0x40,0x20,0x10,0x08,0x04,0x02,0x01}, {0x80,0x40,0x20,0x10,0x08,0x04,0x02,0x01},
      {0x00,0x00,0x00,0x01,0x00,0x00,0x00,0x00}, {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x10},
      {0x00,0x00,0x00,0x02,0x00,0x00,0x00,0x00}, {0x00,0x00,0x00,0x00,0x00,0x00,0x10,0x00},
      {0x00,0x00,0x00,0x04,0x00,0x00,0x00,0x00}, {0x00,0x00,0x00,0x00,0x00,0x10,0x00,0x00},
      {0x00,0x00,0x00,0x08,0x00,0x00,0x00,0x00}, {0x00,0x00,0x00,0x00,0x10,0x00,0x00,0x00},
      {0x55,0x55,0x55,0x55,0x55,0x55,0x55,0x55}, {0x00,0xFF,0x00,0xFF,0x00,0xFF,0x00,0xFF},
      {0xAA,0xAA,0xAA,0xAA,0xAA,0xAA,0xAA,0xAA}, {0xFF,0x00,0xFF,0x00,0xFF,0x00,0xFF,0x00},
      {0x7F,0x3F,0x1F,0x0F,0x07,0x03,0x01,0x00}, {0x00,0x80,0xC0,0xE0,0xF0,0xF8,0xFC,0xFE},
      {0xFF,0xFF,0xFF,0xFF,0x00,0x00,0x00,0x00}, {0xF0,0xF0,0xF0,0xF0,0xF0,0xF0,0xF0,0xF0},
      {0x00,0x00,0x00,0x00,0xFF,0xFF,0xFF,0xFF}, {0x0F,0x0F,0x0F,0x0F,0x0F,0x0F,0x0F,0x0F},
      {0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF}, {0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF},
   };

   unsigned char result[8];
   n = sizeof(test)/8;

   printf("transpose8vn:\n");
   for (i = 0; i < n; i += 2) {
      transpose8vn(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8vn(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8b64:\n");
   for (i = 0; i < n; i += 2) {
      transpose8b64(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8b64(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8b64c:\n");
   for (i = 0; i < n; i += 2) {
      transpose8b64c(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8b64c(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8bS64:\n");
   for (i = 0; i < n; i += 2) {
      transpose8bS64(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8bS64(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8r64:\n");
   for (i = 0; i < n; i += 2) {
      transpose8r64(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8r64(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8rS64:\n");
   for (i = 0; i < n; i += 2) {
      transpose8rS64(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8rS64(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8b32:\n");
   for (i = 0; i < n; i += 2) {
      transpose8b32(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8b32(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8bS32:\n");
   for (i = 0; i < n; i += 2) {
      transpose8bS32(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8bS32(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8r32:\n");
   for (i = 0; i < n; i += 2) {
      transpose8r32(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8r32(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8rS32:\n");
   for (i = 0; i < n; i += 2) {
      transpose8rS32(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8rS32(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   printf("transpose8rSr32:\n");
   for (i = 0; i < n; i += 2) {
      transpose8rSr32(test[i], 1, 1, result);
      if (memcmp((char *)test[i+1], (char *)result, 8) != 0)
         error(test[i], result);

      transpose8rSr32(test[i+1], 1, 1, result);
      if (memcmp((char *)test[i], (char *)result, 8) != 0)
         error(test[i+1], result);
   }

   if (errors == 0)
      printf("Passed all %d cases and their reversals.\n", n/2);
   return 0;
}

/* Summary of instruction counts and operation counts (instruction
counts less those for mask generation) for all the transpose8 functions
above except for transpose8b64c:

        Instructions          Operations
           Machine              Machine
       64-bit  32-bit       64-bit  32-bit
8vn      174     174          174     174      Very naive method
8b        54      84           43      74      Basic direct placement
8bS       50      81           42      74      Above with GLS bit swapping
8r        38      45           21      36      Basic recursive
8rS       26      37           18      32      Above with GLS bit swapping
8rSr              37                   32      8rS in reverse order
*/
