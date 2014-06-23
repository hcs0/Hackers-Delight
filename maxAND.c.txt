/* Given a, b, c, and d, this program computes the max value of x & y,
where a <= x <= b and c <= y <= d (unsigned numbers).
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

int nlz(unsigned x) {
   int n;

   if (x == 0) return(32);
   n = 0;
   if (x <= 0x0000FFFF) {n = n +16; x = x <<16;}
   if (x <= 0x00FFFFFF) {n = n + 8; x = x << 8;}
   if (x <= 0x0FFFFFFF) {n = n + 4; x = x << 4;}
   if (x <= 0x3FFFFFFF) {n = n + 2; x = x << 2;}
   if (x <= 0x7FFFFFFF) {n = n + 1;}
   return n;
}

unsigned minOR(unsigned a, unsigned b,
               unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (~a & c & m) {
         temp = (a | m) & -m;
         if (temp <= b) {a = temp; break;}
      }
      else if (a & ~c & m) {
         temp = (c | m) & -m;
         if (temp <= d) {c = temp; break;}
      }
      m = m >> 1;
   }
   return a | c;
}

// ------------------------------ cut ----------------------------------
unsigned maxAND(unsigned a, unsigned b,
                unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (b & ~d & m) {
         temp = (b & ~m) | (m - 1);
         if (temp >= a) {b = temp; break;}
      }
      else if (~b & d & m) {
         temp = (d & ~m) | (m - 1);
         if (temp >= c) {d = temp; break;}
      }
      m = m >> 1;
   }
   return b & d;
}
// ------------------------------ cut ----------------------------------
/* Speedups: "b & ~d" and "~b & d" move out of the loop.
A better starting value of m is
   m = 0x80000000 >> nlz(b ^ d);
(best to have mod 32 shifts for case b ^ d = 0).
Or, use one of the methods for computing flp2(x) in sect. 3-2.
*/

unsigned brute(unsigned a, unsigned b, unsigned c, unsigned d) {

   unsigned i, j, rmax;

   rmax = 0;                    // Init to 0.
   for (i = a; i <= b; i++) {
      for (j = c; j <= d; j++) {
         if ((i & j) > rmax) rmax = i & j;
      }
   }
   return rmax;
}

int main() {
   unsigned n, nn, a, b, c, d, rmax, r1, r2;

   n = 5;                       // Size of problem.
   nn = 1 << n;                 // 2**n.

   for (a = 0; a < nn; a++) {
      for (b = a; b < nn; b++) {
         for (c = 0; c < nn; c++) {
            for (d = c; d < nn; d++) {
               rmax = brute(a, b, c, d);        // Correct result.
               r1 = maxAND(a, b, c, d);
               r2 = ~minOR(~b, ~a, ~d, ~c);     // Algebraic method.
/*             printf("maxAND(%04x %04x %04x %04x) = %04x\n", a, b, c, d, r1); */
               if (r1 != rmax || r2 != rmax) {
                  printf("ERROR, %04x <= x <= %04x, %04x <= y <= %04x\n"
                         "r1 = %04x, r2 = %04x, rmax = %04x\n", a, b, c, d, r1, r2, rmax);
                  return 1;
               }
            }
         }
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
