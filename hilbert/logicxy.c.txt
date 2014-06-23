#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
/*
Simulates the left-to-right logic circuit for converting path length s
to (x, y) coordinates in the order n Hilbert curve.

                  If       Then set     And set swap and
              s[2i+1:2i]   x[i]  y[i]   complement controls
              -----------|------------|-------------------
                   00    |  0     0   | swap = ~swap
                   01    |  0     1   | No change
                   10    |  1     1   | No change
                   11    |  1     0   | swap = ~swap, cmpl = ~cmpl

To use this table, initially have swap = cmpl = 0, which means do not
swap and do not complement.  Start with the most significant two bits of
s (i = n).  If they are both 0 (first row), set the most significant
bits of x and y to 0, and invert the "swap" control.  Next, test the
next two bits of s. Suppose they are 01.  Then, since swap = 1, set
x[n-1] = 1 and x[y-1] = 0 and do not change the swap and complement
controls.
   Continue until the least significant bits of s have been processed. */

/* This code is not in the book.  It is not really written for
efficiency, just to make sure the logic works.  */

void logic_xy_from_s(unsigned s, int n, unsigned *xp,
                                      unsigned *yp) {

   int i, sa, sb, swap, cmpl, xi, yi, temp;
   unsigned x, y;

   x = y = 0;                     // Initialize result.
   swap = cmpl = 0;               // Initialize controls.
   for (i = n - 1; i >= 0; i--) {
      sa = (s >> (2*i+1)) & 1;    // Get bit 2i+1 of s.
      sb = (s >> 2*i) & 1;        // Get bit 2i of s.

      xi = sa;                    // Set basic (xi, yi).
      yi = sa ^ sb;
      if (swap) {                 // Swap and/or
         temp = xi;               // complement
         xi = yi;                 // xi and yi.
         yi = temp;
      }
      if (cmpl) {
         xi = 1 - xi;
         yi = 1 - yi;
      }

      x = (x << 1) | xi;          // Append xi and yi
      y = (y << 1) | yi;          // to x and y.

      if ((sa ^ sb) == 0) {       // Update controls.
         swap = 1 - swap;
         if (sa)
            cmpl = 1 - cmpl;
      }
   }
   *xp = x;                       // Return (x, y) to
   *yp = y;                       // caller.
   return;
}
// ------------------------------ cut ----------------------------------

int main(int argc, char *argv[]) {
   int n, N;
   unsigned s, x, y;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                        // N = 2**2n.
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   for (s = 0; s < N; s++) {
      logic_xy_from_s(s, n, &x, &y);          // Pass addresses of x and y.
      printf("%5d %5d %5d\n", s, x, y);
   }
   return 0;
}
