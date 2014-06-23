/* Variation of lamxy that eliminates the branch in the loop. */
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
/*
Given the "order" n of a Hilbert curve and a distance s along the curve,
this program computes the corresponding (x, y) coordinates.  The square
that the Hilbert curve traverses is of size 2**n by 2**n.
   The method is that given in [Lam&Shap], described by the following
table.  Here i = n-1 for the most significant bit of x and y, and i = 0
for the least significant bits.

              s[2i+1:2i]   x[i]  y[i]  x[i-1:0]  y[i-1:0]
              -----------|-------------------------------
                   00    |  0     0    y[i-1:0]  x[i-1:0]
                   01    |  0     1    x[i-1:0]  y[i-1:0]
                   10    |  1     1    x[i-1:0]  y[i-1:0]
                   11    |  1     0   ~y[i-1:0] ~x[i-1:0]

To use this table, start at the least significant two bits of s (i = 0).
If they are both 0 (first row), set the least significant bits of x and
y to 0 and 0 respectively, and interchange x and y.  The last two
columns designate an interchange of the bits of x and y to the right of
bit i, but on this first iteration they are null, so there is no
interchange to do.  If the least significant two bits of s are 10 (third
row), set the least significant bits of x and y to 1, and similarly for
the other rows.
   Then, consider the next least significant two bits of s, and select
the appropriate row of the table to determine the next bits of x and y,
and how to change the bits of x and y to the right of i.  Continue until
the most significant bits of x and y have been processed. */

// ------------------------------ cut ----------------------------------
void hil_xy_from_s(unsigned s, int n, unsigned *xp,
                                      unsigned *yp) {

   int i, sa, sb;
   unsigned x, y, swap, cmpl;

   for (i = 0; i < 2*n; i += 2) {
      sa = (s >> (i+1)) & 1;      // Get bit i+1 of s.
      sb = (s >> i) & 1;          // Get bit i of s.

      swap = (sa ^ sb) - 1;  // -1 if should swap, else 0.
      cmpl = -(sa & sb);     // -1 if should compl't, else 0.
      x = x ^ y;
      y = y ^ (x & swap) ^ cmpl;
      x = x ^ y;

      x = (x >> 1) | (sa << 31);  // Prepend sa to x and
      y = (y >> 1) | ((sa ^ sb) << 31); // (sa^sb) to y.
   }
   *xp = x >> (32 - n);           // Right-adjust x and y
   *yp = y >> (32 - n);           // and return them to
}                                 // the caller.
// ------------------------------ cut ----------------------------------

int main(int argc, char *argv[]) {
   unsigned n, N, s, x, y;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                        // N = 2**2n.
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   for (s = 0; s < N; s++) {
      hil_xy_from_s(s, n, &x, &y);          // Pass addresses of x and y.
      printf("%5d %5d %5d\n", s, x, y);
   }
   return 0;
}
