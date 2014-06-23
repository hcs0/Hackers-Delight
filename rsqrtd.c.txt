// Fast approximate reciprocal square root of a double.
// Compile with g++ (for anonymous unions).
#include <math.h>
#include <stdio.h>
#include <stdlib.h>

/* ----------------------------- rsqrtd ----------------------------- */

/* This is a version of rsqrt2 (see file rsqrt.c) for double-precision.
   The constant 0x5fe6ec85e80...0 balances the relative error at +-0.034213.
   The constant 0x5fe618fdf80...0 makes the relative error range from 0 to
-0.061330.
   The constant 0x5fe80...0 makes the relative error range from 0 to
+0.088662. */

double rsqrtd(double x0) {
   union {long long ix; double x;};

   x = x0;
   ix = 0x5fe6ec85e8000000LL - (ix >> 1);
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
   yt = 1.0/sqrt(x);
   ya = rsqrtd(x);

   printf("x = %g %016llx\nTrue recip sqrt = %g, approx = %g, rel error = %.7f\n",
      x, ix, yt, ya, (ya - yt)/yt);
   return 0;
}
