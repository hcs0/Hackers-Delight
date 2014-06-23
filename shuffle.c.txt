// Programs for computing the perfect shuffle of the bits in a word.
// Some line lengths exceed 57, must use a smaller font.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

// Basic perfect outer shuffle (42 Brisc instructions).

unsigned shuffle1(unsigned x) {
// ------------------------------ cut ----------------------------------
   x = (x & 0x0000FF00) << 8 | (x >> 8) & 0x0000FF00 | x & 0xFF0000FF;
   x = (x & 0x00F000F0) << 4 | (x >> 4) & 0x00F000F0 | x & 0xF00FF00F;
   x = (x & 0x0C0C0C0C) << 2 | (x >> 2) & 0x0C0C0C0C | x & 0xC3C3C3C3;
   x = (x & 0x22222222) << 1 | (x >> 1) & 0x22222222 | x & 0x99999999;
// ---------------------------- end cut --------------------------------
   return x;
}

// Perfect outer shuffle coded as swaps [GLS] (30 Brisc instructions).

unsigned shuffle2(unsigned x) {
   unsigned t;
// ------------------------------ cut ----------------------------------
   t = (x ^ (x >> 8)) & 0x0000FF00;  x = x ^ t ^ (t << 8);
   t = (x ^ (x >> 4)) & 0x00F000F0;  x = x ^ t ^ (t << 4);
   t = (x ^ (x >> 2)) & 0x0C0C0C0C;  x = x ^ t ^ (t << 2);
   t = (x ^ (x >> 1)) & 0x22222222;  x = x ^ t ^ (t << 1);
// ---------------------------- end cut --------------------------------
   return x;
}

// Perfect outer unshuffle coded as swaps [GLS] (30 Brisk instructions).

unsigned unshuffle(unsigned x) {
   unsigned t;
// ------------------------------ cut ----------------------------------
   t = (x ^ (x >> 1)) & 0x22222222;  x = x ^ t ^ (t << 1);
   t = (x ^ (x >> 2)) & 0x0C0C0C0C;  x = x ^ t ^ (t << 2);
   t = (x ^ (x >> 4)) & 0x00F000F0;  x = x ^ t ^ (t << 4);
   t = (x ^ (x >> 8)) & 0x0000FF00;  x = x ^ t ^ (t << 8);
// ---------------------------- end cut --------------------------------
   return x;
}

/* Half shuffle (i.e., move the bits from the right halfword to the even
positions) derived as a simplification of shuffle2 (22 or 23 Brisk
instructions, 12 cycles.  Not in the book. */

unsigned halfshuffle1(unsigned x) {
   unsigned t;
   x = x & 0x0000FFFF;          // (If required.)
   t = x & 0x0000FF00;  x = x ^ t ^ (t << 8);
   t = x & 0x00F000F0;  x = x ^ t ^ (t << 4);
   t = x & 0x0C0C0C0C;  x = x ^ t ^ (t << 2);
   t = x & 0x22222222;  x = x ^ t ^ (t << 1);
   return x;
}

/* Half shuffle coded from scratch (19 Brisk instructions, 12 cycles). */

unsigned halfshuffle2(unsigned x) {
// ------------------------------ cut ----------------------------------
   x = ((x & 0xFF00) << 8) | (x & 0x00FF);
   x = ((x << 4) | x) & 0x0F0F0F0F;
   x = ((x << 2) | x) & 0x33333333;
   x = ((x << 1) | x) & 0x55555555;
// ---------------------------- end cut --------------------------------
   return x;
}

/* Half unshuffle (i.e., move the bits from the even positions to the
right halfword) derived as a simplification of unshuffle (26 or 29
instructions, 17 or 19 cycles).  Not in the book. */

unsigned halfunshuffle1(unsigned x) {
   unsigned t;
   x = x & 0x55555555;          // (If required.)
   t = (x >> 1) & 0x22222222;  x = x ^ t ^ (t << 1);
   t = (x >> 2) & 0x0C0C0C0C;  x = x ^ t ^ (t << 2);
   t = (x >> 4) & 0x00F000F0;  x = x ^ t ^ (t << 4);
   t = (x >> 8) & 0x0000FF00;  x = x ^ t ^ (t << 8);
   return x;
}

/* Half unshuffle coded from scratch (18 or 21 Brisk instructions,
12 or 15 cycles. */

unsigned halfunshuffle2(unsigned x) {
// ------------------------------ cut ----------------------------------
   x = x & 0x55555555;          // (If required.)
   x = ((x >> 1) | x) & 0x33333333;
   x = ((x >> 2) | x) & 0x0F0F0F0F;
   x = ((x >> 4) | x) & 0x00FF00FF;
   x = ((x >> 8) | x) & 0x0000FFFF;
// ---------------------------- end cut --------------------------------
   return x;
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %08x\n", x, y);
}

int main() {
   int i, n;
   unsigned r;
   static unsigned test[] = {0,0, 1,1, 2,4, 4,16, 8,64, 16,256,
      32,1024, 64,4096, 128,16384, 256,65536, 512,262144,
      1024,1048576, 2048,4194304, 4096,16777216, 8192,67108864,
      0x4000,0x10000000, 0x8000,0x40000000, 0x10000,2,
      0x20000,8, 0x40000,32, 0x80000,128, 0x100000,512,
      0x200000,2048, 0x400000,8192, 0x800000,32768, 0x1000000,131072,
      0x2000000,0x80000, 0x4000000,0x200000, 0x8000000,0x800000,
      0x10000000,0x2000000, 0x20000000,0x8000000, 0x80000000,0x80000000,
      0x0000FFFF,0x55555555, 0xFFFF0000,0xAAAAAAAA,
      0x77777777,0x3F3F3F3F, 0xFFFFFFFF,0xFFFFFFFF};

   n = sizeof(test)/4;

   printf("shuffle1:\n");
   for (i = 0; i < n; i += 2) {
      r = shuffle1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("shuffle2:\n");
   for (i = 0; i < n; i += 2) {
      r = shuffle2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("unshuffle:\n");
   for (i = 0; i < n; i += 2) {
      r = unshuffle(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("halfshuffle1:\n");
   for (i = 0; i < n; i += 2) {
      r = halfshuffle1(test[i]);
      if (r != (test[i+1] & 0x55555555)) error(test[i], r);}

   printf("halfshuffle2:\n");
   for (i = 0; i < n; i += 2) {
      r = halfshuffle2(test[i]);
      if (r != (test[i+1] & 0x55555555)) error(test[i], r);}

   printf("halfunshuffle1:\n");
   for (i = 0; i < n; i += 2) {
      r = halfunshuffle1(test[i+1]);
      if (r != (test[i] & 0x0000ffff)) error(test[i+1], r);}

   printf("halfunshuffle2:\n");
   for (i = 0; i < n; i += 2) {
      r = halfunshuffle2(test[i+1]);
      if (r != (test[i] & 0x0000ffff)) error(test[i+1], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
}
