#include <stdio.h>
#include <stdlib.h>
/*
Given the "order" n of a Hilbert curve and coordinates x and y, this
program computes the length s of the curve from the origin to (x, y).
The square that the Hilbert curve traverses is of size 2**n by 2**n.
   The method is that given in [Lam&Shap], described by the following
table.  Here i = n-1 for the most significant bit of x and y, and i = 0
for the least significant bits.

                    x[i]  y[i] | s[2i+1:2i]   x   y
                    -----------|-------------------
                     0     0   |     00       y   x
                     0     1   |     01       x   y
                     1     0   |     11      ~y  ~x
                     1     1   |     10       x   y

To use this table, start at the most significant bits of x and y
(i = n - 1).  If they are both 0 (first row), set the most significant
two bits of s to 00 and interchange x and y.  (Actually, it is only
necessary to interchange the remaining bits of x and y.)  If the most
significant bits of x and y are 10 (third row), output 11, interchange x
and y, and complement x and y.
   Then, consider the next most significant bits of x and y (which may
have been changed by this process), and select the appropriate row of
the table to determine the next two bits of s, and how to change x and
y.  Continue until the least significant bits of x and y have been
processed. */

// ------------------------------ cut ----------------------------------
unsigned hil_s_from_xy(unsigned x, unsigned y, int n) {

   int i, xi, yi;
   unsigned s, temp;

   s = 0;                         // Initialize.
   for (i = n - 1; i >= 0; i--) {
      xi = (x >> i) & 1;          // Get bit i of x.
      yi = (y >> i) & 1;          // Get bit i of y.

      if (yi == 0) {
         temp = x;                // Swap x and y and,
         x = y^(-xi);             // if xi = 1,
         y = temp^(-xi);          // complement them.
      }
      s = 4*s + 2*xi + (xi^yi);   // Append two bits to s.
   }
   return s;
}
// ------------------------------ cut ----------------------------------

int main(int argc, char *argv[]) {
   int n, N, x, y;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << n;                          // N = 2**n.
   printf("    x     y     s, order %d Hilbert curve.\n", n);
   for (x = 0; x < N; x++)
      for (y = 0; y < N; y++)
         printf("%5d %5d %5d\n", x, y, hil_s_from_xy(x, y, n));
   return 0;
}
