// Fast approximate square root of a float.
// Compile with g++ (for anonymous unions).
#include <math.h>
#include <stdio.h>
#include <stdlib.h>

/* ----------------------------- asqrt ------------------------------ */

/* This is a novel and fast routine for the approximate square root of
an IEEE float (single precision). It is derived from a similar program
for the approximate reciprocal square root.
   The relative error ranges from 0 to +0.0006011.
   Caution:
Result for -0 is -1.35039e+19 (unreasonable).
Result for 0 is 3.96845e-20.
For denorms it is either within tolerance or gives a result < 1.0e-19.
Gives the correct result (inf) for x = inf.
Gives the correct result (NaN) for x = NaN. */

float asqrt(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = 0x1fbb67a8 + (ix >> 1); // Initial guess.
   x = 0.5f*(x + x0/x);         // Newton step.
   return x;
}

/* ----------------------------- asqrt1 ----------------------------- */

/* This is asqrt with an additional step of the Newton iteration, for
increased accuracy.
   The relative error ranges from 0 to +0.00000023. */

float asqrt1(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = 0x1fbb3f80 + (ix >> 1); // Initial guess.
   x = 0.5f*(x + x0/x);         // Newton step.
   x = 0.5f*(x + x0/x);         // Newton step again.
   return x;
}

/* ----------------------------- asqrt2 ----------------------------- */

/* This is a very approximate but very fast version of asqrt. It is just
two integer instructions (shift right and add), plus instructions
to load the constant.
   The constant 0x1fbb4f2e balances the relative error at +-0.0347474. */

float asqrt2(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = 0x1fbb4f2e + (ix >> 1); // Initial guess.
   return x;
}

/* ------------------------------ main ------------------------------ */

int main(int argc, char *argv[]) {
   float yt, ya;
   union {int ix; float x;};

   if (argc != 2) {
      printf("Need exactly one argument, a floating-point number.\n");
      return 1;
   }

   x = atof(argv[1]);
   yt = sqrt(x);
   ya = asqrt1(x);               // Change to asqrt1 or asqrt2 to test
                                // the other routines.
   printf("x = %g %08x\nTrue sqrt = %g, approx = %g, rel error = %.7f\n",
      x, ix, yt, ya, (ya - yt)/yt);
   return 0;
}
