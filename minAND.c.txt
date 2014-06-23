/* Given a, b, c, and d, this program computes the min value of x & y,
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

unsigned maxOR(unsigned a, unsigned b,
               unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (b & d & m) {
         temp = (b - m) | (m - 1);
         if (temp >= a) {b = temp; break;}
         temp = (d - m) | (m - 1);
         if (temp >= c) {d = temp; break;}
      }
      m = m >> 1;
   }
   return b | d;
}

// ------------------------------ cut ----------------------------------
unsigned minAND(unsigned a, unsigned b,
                unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (~a & ~c & m) {
         temp = (a | m) & -m;
         if (temp <= b) {a = temp; break;}
         temp = (c | m) & -m;
         if (temp <= d) {c = temp; break;}
      }
      m = m >> 1;
   }
   return a & c;
}
// ------------------------------ cut ----------------------------------
/* Speedups: "~a & ~c" moves out of loop as ~(a | c).
A better starting value of m is
   m = 0x80000000 >> nlz(~(a | c));
(best to have mod 32 shifts for case ~(a | c) = 0).
Or, use one of the methods for computing flp2(x) in sect. 3-2.
*/

unsigned brute(unsigned a, unsigned b, unsigned c, unsigned d) {

   unsigned i, j, rmin;

   rmin = 0xffffffff;           // Init to infinity.
   for (i = a; i <= b; i++) {
      for (j = c; j <= d; j++) {
         if ((i & j) < rmin) rmin = i & j;
      }
   }
   return rmin;
}

int main() {
   unsigned n, nn, a, b, c, d, rmin, r1, r2;

   n = 5;                       // Size of problem.
   nn = 1 << n;                 // 2**n.

   for (a = 0; a < nn; a++) {
      for (b = a; b < nn; b++) {
         for (c = 0; c < nn; c++) {
            for (d = c; d < nn; d++) {
               rmin = brute(a, b, c, d);        // Correct result.
               r1 = minAND(a, b, c, d);
               r2 = ~maxOR(~b, ~a, ~d, ~c);     // Algebraic method.
/*             printf("minAND(%04x %04x %04x %04x) = %04x\n", a, b, c, d, r1); */
               if (r1 != rmin || r2 != rmin) {
                  printf("ERROR, %04x <= x <= %04x, %04x <= y <= %04x\n"
                         "r1 = %04x, r2 = %04x, rmin = %04x\n", a, b, c, d, r1, r2, rmin);
                  return 1;
               }
               if (r1 > (a & c))   printf("minAND too big.\n");
            }
         }
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
