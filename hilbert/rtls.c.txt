#include <stdio.h>
#include <stdlib.h>
/* Right-to-left algorithm for computing s from (x, y). */
// Not in hacker.book.
unsigned s_from_xy(unsigned x, unsigned y, int n) {

   int i, xi, yi;
   unsigned s, temp;

   s = 0;                         // Initialize.
   for (i = 0; i < n; i++) {
      xi = (x >> i) & 1;          // Get bit i of x.
      yi = (y >> i) & 1;          // Get bit i of y.

      if (yi == 0) {
         if (xi == 0)
            s = s ^ ((s & 0x55555555) << 1);
         else
            s = s ^ ((~s & 0x55555555) << 1);
      }
                                  // Prepend two bits to s.
      s = (s >> 2) | (xi << 31) | ((xi ^ yi) << 30);
   }
   return s >> (32 - 2*n);
}

void main(int argc, char *argv[]) {
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
         printf("%5d %5d %5d\n", x, y, s_from_xy(x, y, n));
   return 0;
}
