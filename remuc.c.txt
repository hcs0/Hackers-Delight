/* Computes remu(n, d) for d = 3, 5, 7, 9 (remainder of unsigned
division). This material is not in HD, but may be in the second edition.
Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

//                  METHODS BASED ON SUMMING DIGITS

/* If you have an efficiently implemented pop instruction, the code
below may be useful. It is 18 ops. 3 of the ops are to generate the
constants (and hence can move out of a loop).
   This code is of historical interest only, as the function immediately
following is much better. */

int remu3a(unsigned n) {
   n = pop(n & 0x55555555) - pop(n & 0xAAAAAAAA);
   n = n - (n >> 31);
   n = pop(n & 0x55555555) - pop(n & 0xAAAAAAAA);
   return n + (((int)n >> 31) & 3);
}

/* Below is a substantially improved version of the above, by Paolo
Bonzini. 11 ops, two of which are to generate the large constant. */

int remu3b(unsigned n) {
   n = pop(n ^ 0xAAAAAAAA) + 23;    // Now 23 <= n <= 55.
   n = pop(n ^ 0x2A) - 3;           // Now -3 <= n <= 2.
   return n + (((int)n >> 31) & 3); // (Signed shift).
}

/* Using pop with a table lookup. Uses Bonzini's identity. Four ops, two
of which are to generate the large constant, plus an indexed load. */

int remu3c(unsigned n) {

   static char table[33] = {2, 0,1,2, 0,1,2, 0,1,2,
          0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
          0,1,2, 0,1};

   n = pop(n ^ 0xAAAAAAAA);
   return table[n];
}

/* The code below follows the technique of Figure 5-2. 16 ops plus an
indexed load. Just mention this in the book, don't show it, because the
subsequent function (remu3e) is better. */

int remu3d(unsigned n) {

   static char table[49] = {0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0};

   n = (n & 0x33333333) + ((n >> 2) & 0x33333333);
   n = (n + (n >> 4)) & 0x0F0F0F0F;
   n = n + (n >> 8);
   n = n + (n >> 16);
   return table[n & 0x3F];
}

/* The code below is 19 elementary ops. If a table of size 62 bytes is
used, it is 9 ops + an indexed load. If a table of size 766 bytes is
used, it is 6 ops + an indexed load. */

int remu3e(unsigned n) {
   n = (n >> 16) + (n & 0xFFFF);        // Max 0x1FFFE.
   n = (n >>  8) + (n & 0x00FF);        // Max 0x2FD.
   n = (n >>  4) + (n & 0x000F);        // Max 0x3D.
   n = (n >>  2) + (n & 0x0003);        // Max 0x11.
   n = (n >>  2) + (n & 0x0003);        // Max 0x6.
   return (0x0924 >> (n << 1)) & 3;
}

/* The code below is 9 ops plus an indexed load. */

int remu3f(unsigned n) {
   static char table[62] = {0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1};

   n = (n >> 16) + (n & 0xFFFF);        // Max 0x1FFFE.
   n = (n >>  8) + (n & 0x00FF);        // Max 0x2FD.
   n = (n >>  4) + (n & 0x000F);        // Max 0x3D.
   return table[n];
}

/* The code below is 21 elementary ops assuming the multiplication by 3
is expanded into a shift and add. */

int remu5a(unsigned n) {
   n = (n >> 16) + (n & 0xFFFF);          // Max 0x1FFFE.
   n = (n >>  8) + (n & 0x00FF);          // Max 0x2FD.
   n = (n >>  4) + (n & 0x000F);          // Max 0x3D.
   n = (n>>4) - ((n>>2) & 3) + (n & 3);   // -3 to 6.
   return (01043210432 >> 3*(n + 3)) & 7; // Octal const.
}

/* The code below is 9 elementary ops + an indexed load. Don't bother to
put this in the book, it's very similar to rem3e. */

int remu5b(unsigned n) {

   static char table[62] = {0,1,2,3,4, 0,1,2,3,4,
      0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4,
      0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4,
      0,1,2,3,4, 0,1,2,3,4, 0,1};

   n = (n >> 16) + (n & 0xFFFF);        // Max 0x1FFFE.
   n = (n >>  8) + (n & 0x00FF);        // Max 0x2FD.
   n = (n >>  4) + (n & 0x000F);        // Max 0x3D.
   return table[n];
}

/* The code below is 9 elementary ops + an indexed load. */

int remu7(unsigned n) {

   static char table[75] = {0,1,2,3,4,5,6, 0,1,2,3,4,5,6,
             0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4,5,6,
             0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4,5,6,
             0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4};

   n = (n >> 15) + (n & 0x7FFF);        // Max 0x27FFE.
   n = (n >>  9) + (n & 0x001FF);       // Max 0x33D.
   n = (n >>  6) + (n & 0x0003F);       // Max 0x4A.
   return table[n];
}

/* The code below is 9 elementary ops + an indexed load. Don't bother
putting this in the book, because remu9b is better (smaller table). */

int remu9a(unsigned n) {

   static char table[128] = {0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1};

   n = (n >> 18) + (n & 0x3FFFF);       // Max 0x43FFE.
   n = (n >> 12) + (n & 0x00FFF);       // Max 0x1041.
   n = (n >>  6) + (n & 0x0003F);       // Max 0x7F.
   return table[n];
}

/* The code below is 9 elementary ops + an indexed load.
6 ops if a table of size 831 (decimal) is used. */

int remu9b(unsigned n) {

   int r;
   static char table[75] = {0,1,2,3,4,5,6,7,8,
         0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
         0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
         0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
         0,1,2,3,4,5,6,7,8, 0,1,2};

   r = (n & 0x7FFF) - (n >> 15);     // FFFE0001 to 7FFF.
   r = (r & 0x01FF) - (r >>  9);     // FFFFFFC1 to 2FF.
   r = (r & 0x003F) + (r >>  6);     // 0 to 4A.
   return table[r];
}

//                  METHODS BASED ON MULTIPLICATION

/* The code below is 10 ops, including a multiply. It is based on
                     n mod 3 = floor((4/3)n) mod 4.
The terms n >> 1 and n >> 3 get more accuracy and cause the estimate
to be a little high. 8 ops. */

int remu3m(unsigned n) {
   return (0x55555555*n + (n >> 1) - (n >> 3)) >> 30;
}

/* The code below is the same as remu3m except the multiplication by
0x55555555 is expanded. 13 elementary ops. */

int remu3n(unsigned n) {
   unsigned r;

   r = n + (n << 2);
   r = r + (r << 4);
   r = r + (r << 8);
   r = r + (r << 16);
   r = r + (n >> 1);
   r = r - (n >> 3);
   return r >> 30;
}

/* Based on floor((8/5)n) mod 8 = 0, 1, 3, 4, 6, 7 for n = 0, 1, 2, 3,
4, 5 resp. Hence must translate. Mapping 2 to 2 and 5 to 4 reduces the
precision necessary. Without the term n >> 3, first error is at n =
0x60000005 (gets 0, shd be 1). 11 ops including a multiply. */

int remu5m(unsigned n) {
   n = (0x33333333*n + (n >> 3)) >> 29;
   return (0x04432210 >> (n << 2)) & 7;
}

/* Based on floor((8/7)n) mod 8 = 0, 1, 2, 3, 4, 5, 6, 7 for n = 0, 1,
2, 3, 4, 5, 6, 7 resp. Hence must translate, just mapping 7 to 0.
Without the accuracy terms, first error is at n = 0x08000007 (gets 0,
shd be 1). With just the term n >> 1, first error is at n = 0x40000007.
11 ops including a multiply. */

int remu7m(unsigned n) {
   n = (0x24924924*n + (n >> 1) + (n >> 4)) >> 29;
   return n & ((int)(n - 7) >> 31);
}

/* Based on an approximation of floor((16/9)n) mod 16. Must translate 2
to 1, 3 to 2, etc. The table was arrived at by experimentation. Without
the term n >> 1, first failure is at n = 9. However, if that term is
omitted and the multiplier is changed to 0x1C71C71D, the first failure
is at n = 0x1999999F. 6 ops including a multiply, plus an indexed load.
All entries in the table are used. */

int remu9m(unsigned n) {
   static char table[16] = {0, 1, 1, 2, 2, 3, 3, 4,
                            5, 5, 6, 6, 7, 7, 8, 8};

   n = (0x1C71C71C*n + (n >> 1)) >> 28;
   return table[n];
}

/* Based on an approximation of floor((16/10)n) mod 16. Must translate
3 to 2, 4 to 3, etc. Without the term n >> 3, first error is at n =
0x40000005. With both accuracy terms deleted, first error is at
0x0AAAAAAE (this can be increased by changes to the table).
8 ops including a multiply, plus an indexed load. */

int remu10m(unsigned n) {
   static char table[16] = {0, 1, 2, 2, 3, 3, 4, 5,
                            5, 6, 7, 7, 8, 8, 9, 0};

   n = (0x19999999*n + (n >> 1) + (n >> 3)) >> 28;
   return table[n];
}

/* The mysterious code below is by Joe Keane, given in newsgroup
sci.math.num-analysis, 1995/07/09. Norbert Joffa pointed it out to me.
Peter Montgomery has allegedly proved it correct. Here we verify it
exhaustively.
   This function is used in the popcount function at the top of HD p. 84.
   12 elementary ops, probably better than the methods above. */

int remu63k(unsigned n) {
   unsigned t;

   t = (((n >> 12) + n) >> 10) + (n << 2);
   t = ((t >> 6) + t + 3) & 0xFF;
   return (t - (t >> 6)) >> 2;
}

/* Let us see how the methods suggested in the new "Integer Division by
Constants" section that is on www.HackersDelight.org do.
   The technique of "Remainder by Summing Digits" ("casting out 63's")
does not work out all that well. It is 9 elementary instructions plus an
indexed load into a 256-byte table.
   The technique of "Remainder by Multiplication and Shift Right" is not
too bad. It is based on

                   n mod 63 = floor((64/63)n) mod 64,

but because of accuracy problems, when n is a multiple of 63, the result
of the above calculation is 63. Hence the result must be altered by
changing 63 to 0.
   11 ops, including the multiply and two to load the constant.
   This might be competitive with Keane's algorithm on a machine with a
very fast unsigned integer multiply, if the instructions to load the
constant move out of a loop and/or if it can do the two shifts in
parallel with the multiply. */

int remu63m(unsigned n) {
   n = (0x04104104*n + (n >> 4) + (n >> 10)) >> 26;
   return n & ((n - 63) >> 6);   // Change 63 to 0.
}

/* The code below is the same as remu63m except the multiplication by
0x04104104 is expanded. 15 elementary ops. */

int remu63n(unsigned n) {
   unsigned r;

   r = (n << 2) + (n << 8);      // r = 0x104*n.
   r = r + (r << 12);            // r = 0x104104*n.
   r = r + (n << 26);            // r = 0x04104104*n.
   r = r + (n >> 4) + (n >> 10); // Need more accuracy.
   r = r >> 26;                  // Now 0 <= r <= 63.
   return r & ((r - 63) >> 6);   // Change 63 to 0.
}

int errors;
void error(int n, int r) {
   errors = errors + 1;
   printf("Error for n = %08x, got %d\n", n, r);
}

int main() {
   int n, r;
   const int low = 0xFF000000,  // Set these equal for an exhaustive
            high = 0x00FFFFFF;  // test (takes a long time).

   printf("remu3a:\n");
   n = low; do {
      r = remu3a(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3b:\n");
   n = low; do {
      r = remu3b(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3c:\n");
   n = low; do {
      r = remu3c(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3d:\n");
   n = low; do {
      r = remu3d(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3e:\n");
   n = low; do {
      r = remu3e(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3f:\n");
   n = low; do {
      r = remu3f(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu5a:\n");
   n = low; do {
      r = remu5a(n);
      if (r != (unsigned)n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu5b:\n");
   n = low; do {
      r = remu5b(n);
      if (r != (unsigned)n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu7:\n");
   n = low; do {
      r = remu7(n);
      if (r != (unsigned)n%7) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu9a:\n");
   n = low; do {
      r = remu9a(n);
      if (r != (unsigned)n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu9b:\n");
   n = low; do {
      r = remu9b(n);
      if (r != (unsigned)n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3m:\n");
   n = low; do {
      r = remu3m(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu3n:\n");
   n = low; do {
      r = remu3n(n);
      if (r != (unsigned)n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu5m:\n");
   n = low; do {
      r = remu5m(n);
      if (r != (unsigned)n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu7m:\n");
   n = low; do {
      r = remu7m(n);
      if (r != (unsigned)n%7) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu9m:\n");
   n = low; do {
      r = remu9m(n);
      if (r != (unsigned)n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu10m:\n");
   n = low; do {
      r = remu10m(n);
      if (r != (unsigned)n%10) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu63k:\n");        // Keane's algorithm:
   n = low; do {
      r = remu63k(n);
      if (r != (unsigned)n%63) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu63m:\n");
   n = low; do {
      r = remu63m(n);
      if (r != (unsigned)n%63) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("remu63n:\n");
   n = low; do {
      r = remu63n(n);
      if (r != (unsigned)n%63) {error(n, r); return 1;}
      n = n + 1;
   } while (n != high);

   if (errors == 0)
      printf("Passed all tests.\n");
}
