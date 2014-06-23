#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

/* Given the "order" n of a Hilbert curve and coordinates (x, y), this
program computes the coordinates of the next point on the curve.  See
the text for a description of this algorithm.  */

// --------------------------- hil_inc_xy ------------------------------

// Num ops: TBD

// ------------------------------ cut ----------------------------------
void hil_inc_xy(unsigned *xp, unsigned *yp, int n) {

   int i;
   unsigned x, y, state, dx, dy, row, dochange;

   x = *xp;
   y = *yp;
   state = 0;                   // Initialize.
   dx = -((1 << n) - 1);        // Init. -(2**n - 1).
   dy = 0;

   for (i = n-1; i >= 0; i--) {         // Do n times.
      row = 4*state | 2*((x >> i) & 1) | (y >> i) & 1;
      dochange = (0xBDDB >> row) & 1;
      if (dochange) {
         dx = ((0x16451659 >> 2*row) & 3) - 1;
         dy = ((0x51166516 >> 2*row) & 3) - 1;
      }
      state = (0x8FE65831 >> 2*row) & 3;
   }
   *xp = *xp + dx;
   *yp = *yp + dy;
}
// ------------------------------ cut ----------------------------------

// ------------------------------ main ---------------------------------

int main(int argc, char *argv[]) {
   int n;
   unsigned N, s, x = 0, y = 0;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                        // N = 2**2n.
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   printf("    s     x     y\n");
   for (s = 0; s < N + 4; s++) {        // Do 4 extra iterations.
      printf("%5d %5d %5d\n", s, x, y);
      hil_inc_xy(&x, &y, n);
   }
   return 0;
}
