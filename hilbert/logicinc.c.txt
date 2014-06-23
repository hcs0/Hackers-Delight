#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
/*
Simulates the left-to-right logic circuit for determining whether to
increment or decrement x or y.

This code is not in the book.  It is not written for efficiency, just to
ensure that the logic works.


(X, Y) = (xi, yi) swapped if S = 1 and complemented if C = 1.
I      = 1/0: increment/decrement.
W      = 1/0: change x/y.
S      = 1: Swap x and y.
C      = 1: Complement x and y.

Initial conditions: I(n) = W(n) = S(n) = C(n) = 0.
*/

void logic_inc_from_xy(unsigned x, unsigned y, int n,
                       int *Ip, int *Wp) {

   int i, xi, yi, S, C, I, W;
   unsigned X, Y;

   S = C = 0;                   // Initialize controls.
   I = W = 0;                   // Initialize result.
   for (i = n - 1; i >= 0; i--) {
      xi = (x >> i) & 1;        // Get bit i of x.
      yi = (y >> i) & 1;        // Get bit i of y.

      X = (S & yi | !S & xi) ^ C;
      Y = (S & xi | !S & yi) ^ C;
      I = !C & !X | C & X & Y | I & X & !Y;
      W = !S & !X & Y | S & !(X ^ Y) | W & X & !Y;
      S = !(S ^ Y);
      C = C ^ (X & !Y);
/*    printf("i = %d, xi = %d, yi = %d, I = %d, W = %d, S = %d, C = %d\n", */
/*       i, xi, yi, I, W, S, C); */
   }
   *Ip = I;                     // Return I and W
   *Wp = W;                     // to caller.
   return;
}
// ------------------------------ cut ----------------------------------

int main(int argc, char *argv[]) {
   int n, N, k, I, W;
   unsigned  x, y;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                // N = 2**2n.
   x = y = 0;
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   for (k = 0; k < N; k++) {
      printf("%5d %5d %5d\n", k, x, y);
      logic_inc_from_xy(x, y, n, &I, &W);
      I = 2*I - 1;              // 1 ==> 1, 0 ==> -1.
      if (W) x = x + I;
      else   y = y + I;
   }
   return 0;
}
