// 64-bit version of the hardware icbrt algorithm.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

/* ----------------------------- icbrt1 ----------------------------- */

// Execution time is 3 + (12.5 + mul)*22 = 275 + 22*mul cycles (avg).

int icbrt1(unsigned long long x) {
   int s;
   unsigned long long y, b, bs;

   y = 0;
   for (s = 63; s >= 0; s = s - 3) {
      y = 2*y;
      b = 3*y*(y + 1) + 1;
      bs = b << s;
      if (x >= bs && b == (bs >> s)) {
         x = x - bs;
         y = y + 1;
      }
   }
   return y;
}

/* ---------------------------- icbrt1a ----------------------------- */

/* This is like icbrt1 except the case in which b overflows is treated
differently. Overflow can happen when s = 60. Here we write out the code
for the first two iterations of the loop, which simplifies considerably,
followed by the loop for x = 57 down to 0.
   The first two iterations have the following effect:
             if x >= 2**63, set x = x - 2**63; y = 2.
             else if x >= 2**60, set x = x - 2**60; y = 1.
             else y = 0.
   Execution time is 6 + (11 + mul)*20 = 226 + 20*mul cycles (avg). */

int icbrt1a(unsigned long long x) {
   int s;
   unsigned long long y, b;

   y = 0;
   if (x >= 0x1000000000000000LL) {
      if (x >= 0x8000000000000000LL) {
         x = x - 0x8000000000000000LL;
         y = 2;
      } else {
         x = x - 0x1000000000000000LL;
         y = 1;
      }
   }

   for (s = 57; s >= 0; s = s - 3) {
      y = 2*y;
      b = (3*y*(y + 1) + 1) << s;
      if (x >= b) {
         x = x - b;
         y = y + 1;
      }
   }
   return y;
}

/* ----------------------------- icbrt2 ----------------------------- */

// This is icbrt1 strength reduced to avoid a multiplication of variables.
// Execution time is 4 + 16*22 = 356 cycles (avg).

int icbrt2(unsigned long long x) {
   int s;
   unsigned long long y, b, bs, y2;

   y2 = 0;
   y = 0;
   for (s = 63; s >= 0; s = s - 3) {
      y2 = 4*y2;
      y = 2*y;
      b = 3*(y2 + y) + 1;
      bs = b << s;
      if (x >= bs && b == (bs >> s)) {
         x = x - bs;
         y2 = y2 + 2*y + 1;
         y = y + 1;
      }
   }
   return y;
}

int errors;
void error(long long x, int y) {
   errors = errors + 1;
   printf("Error for x = %016llx, got %d\n", x, y);
}

/* ------------------------------ main ------------------------------ */

int main() {
   int i, n;
   unsigned r;
   static unsigned long long test[] = {0,0, 1,1, 2,1, 3,1, 4,1, 5,1,
      6,1, 7,1, 8,2, 9,2, 10,2, 11,2, 12,2, 13,2, 14,2,
      15,2, 16,2, 17,2, 18,2, 19,2, 20,2, 21,2, 22,2, 23,2,
      24,2, 25,2, 26,2, 27,3, 28,3, 29,3, 30,3, 31,3,
      32,3, 33,3, 34,3, 35,3, 36,3, 37,3, 38,3, 39,3, 40,3,
      99,4, 100,4, 101,4, 32767,31, 32768,32, 32769,32,
      1073741823,1023, 1073741824,1024, 1073741825,1024,
      0x80000000,1290, 0xFFFFFFFF,1625,
      0x100000000LL,1625, 0x1FFFFFFFFLL,2047, 0x200000000LL, 2048,
      0x1FFFFFFFFFFFFFFFLL,1321122,
      0x2FFFFFFFFFFFFFFFLL,1512308,
      0x7FFFFFFFFFFFFFFFLL,2097151,
      0x8000000000000000LL,2097152,
      0x8765432108765432LL,2136787,
      0xFFFFEDE923933E3CLL,2642244,
      0xFFFFEDE923933E3DLL,2642245,
      0xFFFFFFFFFFFFFFFFLL,2642245,
      };

   n = sizeof(test)/8;

   printf("icbrt1:\n");
   for (i = 0; i < n; i += 2) {
      r = icbrt1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("icbrt1a:\n");
   for (i = 0; i < n; i += 2) {
      r = icbrt1a(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("icbrt2:\n");
   for (i = 0; i < n; i += 2) {
      r = icbrt2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/16);
   return 0;
}
