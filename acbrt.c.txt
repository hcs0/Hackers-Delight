// Fast approximate cube root of a float.
// Compile with g++ (for anonymous unions).
#include <math.h>
#include <stdio.h>
#include <stdlib.h>

/* ----------------------------- acbrt ------------------------------ */

/* This is a novel and fast routine for the approximate cube root of
an IEEE float (single precision). It is derived from a similar program
for the approximate square root.
   The relative error ranges from 0 to +0.00103.
   Caution:
Result for -0 is NaN.
Result for 0 is 1.24e-13.
For denorms it is either within tolerance or gives a result < 2.1e-13.
Gives the correct result (inf) for x = inf.
Gives the correct result (NaN) for x = NaN. */

float acbrt(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = ix/4 + ix/16;           // Approximate divide by 3.
   ix = ix + ix/16;
   ix = ix + ix/256;
   ix = 0x2a5137a0 + ix;        // Initial guess.
   x = 0.33333333f*(2.0f*x + x0/(x*x));  // Newton step.
   return x;
}

/* ----------------------------- acbrt1 ----------------------------- */

/* This is acbrt with an additional step of the Newton iteration, for
increased accuracy.
   The relative error ranges from 0 to +0.00000116. */

float acbrt1(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = ix/4 + ix/16;           // Approximate divide by 3.
   ix = ix + ix/16;
   ix = ix + ix/256;
   ix = 0x2a5137a0 + ix;        // Initial guess.
   x = 0.33333333f*(2.0f*x + x0/(x*x));  // Newton step.
   x = 0.33333333f*(2.0f*x + x0/(x*x));  // Newton step again.
   return x;
}

/* ----------------------------- acbrt2 ----------------------------- */

/* This is a very approximate but very fast version of acbrt. It is just
two integer instructions (shift right and divide), plus instructions
to load the constant.
   The constant 0x2a51067f balances the relative error at +-0.0316. */

float acbrt2(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = 0x2a51067f + ix/3;      // Initial guess.
   return x;
}

/* ---------------------------- acbrt2a ----------------------------- */

/* This is a very approximate but very fast version of acbrt. It is just
eight integer instructions (shift rights and adds), plus instructions
to load the constant.
   1/3 is approximated as 1/4 + 1/16 + 1/64 + 1/256 + ... + 1/65536.
   The constant 0x2a511cd0 balances the relative error at +-0.0321. */

float acbrt2a(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = ix/4 + ix/16;           // Approximate divide by 3.
   ix = ix + ix/16;
   ix = ix + ix/256;
   ix = 0x2a511cd0 + ix;        // Initial guess.
   return x;
}

/* ---------------------------- acbrt2b ----------------------------- */

/* This is a very approximate but very fast version of acbrt. It is just
six integer instructions (shift rights and adds), plus instructions
to load the constant.
   1/3 is approximated as 1/4 + 1/16 + 1/64 + 1/256.
   The constant 0x2a6497f8 balances the relative error at +-0.151. */

float acbrt2b(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as int.
   ix = ix/4 + ix/16;           // Approximate divide by 3.
   ix = ix + ix/16;
   ix = 0x2a6497f8 + ix;        // Initial guess.
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
   yt = cbrtf(x);               // From the math library.
   ya = acbrt(x);               // Change to acbrt1 or acbrt2 etc. to
                                // test the other routines.
   printf("x = %g %08x\nTrue cbrt = %g, approx = %g, rel error = %.7f\n",
      x, ix, yt, ya, (ya - yt)/yt);
   return 0;
}
