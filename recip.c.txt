// Fast approximate reciprocal of a normal double > 0.
// Compile with g++ (for anonymous unions).
#include <stdio.h>
#include <stdlib.h>

/* ----------------------------- recip ------------------------------ */

/* This is derived from rsqrtd.c, a reciprocal square root routine. It
   simply computes rsqrtd(x) and then squares it with a floating-pt multiply.
   Max relative error ~ 7.0%.
   The relative error for x and 4.0*x is the same.
   Needs work for x = 0, negative, NaN, subnormals, and infinity.
*/

double recip(double x0) {
   union {long long ix; double x;};

   x = x0;
   ix = 0x5fe6ec85e8000000LL - (ix >> 1);
   return x*x;                  // Floating-pt multiply.
}

/* ----------------------------- recip2 ----------------------------- */

/* This is derived from
   if x = 2**(e - 1023)(1 + f), 0 <= f < 1, then
   1/x = 2**(1023 - e)(1/(1 + f)) = 2**(2046 - (e - 1023))(1 - f + f**2 ...).
   or 1/x ~= 0x7FE00000_00000000 - x (integer subtraction).
   The relative error is from 0 to +12.5% with the constant 0x7FE0...0.
   It can be reduced to ~ +/-5.05% by using a different constant (see below).
   This would require 4 Newton iterations to get to +/-0.5 ULP.
   The Newton iteration is x[n+1] = x[n]*(2.0 - a*x[n]).
   The relative error for x and 2.0*x is the same.
   Needs work for x = 0, negative, NaN, subnormals, infinity,
   and very large numbers (> 4e307).
      Michel Hack has derived analytically the exact value of the "magic
   constant." It is the integer (2046 - (5 - sqrt(24)))*2**52 =
   9213909883965677848 = 0x7FDE6238DA3C2118. */

double recip2(double x0) {
   union {long long ix; double x;};

   x = x0;
// ix = 0x7FE0000000000000LL - ix;      // 0 <= error <= 12.5%.
   ix = 0x7FDE6238DA3C2118LL - ix;      // -5.05102% < error < 5.05103%.
   return x;
}

/* ------------------------------ main ------------------------------ */

int main(int argc, char *argv[]) {
   double yt, ya;
   union {long long ix; double x;};

   if (argc != 2) {
      printf("Need exactly one argument, a floating-point number.\n");
      return 1;
   }

   x = atof(argv[1]);
   yt = 1.0/x;                  // True result.
   ya = recip2(x);              // Our approximation.

   printf("x = %g %016llx\nTrue recip = %g, approx = %g, rel error = %.7f\n",
      x, ix, yt, ya, (ya - yt)/yt);
   return 0;
}
