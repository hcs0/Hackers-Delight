// Fast approximate reciprocal square root of a float.
// Max line length is 57, to fit in hacker.book.
// Compile with g++ (for anonymous unions).
#include <math.h>
#include <stdio.h>
#include <stdlib.h>

/* ----------------------------- rsqrt ------------------------------ */

/* This is a novel and fast routine for the reciprocal square root of an
IEEE float (single precision). It was communicated to me by Mike Morton,
and has been analyzed substantially by Chris Lomont. Later (12/1/06)
Peter Capek pointed it out to me. See:

http://www.lomont.org/Math/Papers/2003/InvSqrt.pdf
http://playstation2-linux.com/download/p2lsd/fastrsqrt.pdf
http://www.beyond3d.com/content/articles/8/

The author of this has been researched but seems to be lost in history.
However, Gary Tarolli worked on it and helped to make it more widely
known, probably while he was at SGI. Gary says it goes back to 1995 or
earlier. */

float rsqrt(float x0) {
   union {int ix; float x;};
// union {int ihalf; float xhalf;}; // For alternative halving step.

   x = x0;                      // x can be viewed as int.
// ihalf = ix - 0x00800000;     // Alternative to line below, for x not a denorm.
   float xhalf = 0.5f*x;
// ix = 0x5f3759df - (ix >> 1); // Initial guess (traditional),
//                                 but slightly better:
   ix = 0x5f375a82 - (ix >> 1); // Initial guess.
   x = x*(1.5f - xhalf*x*x);    // Newton step.
// x = x*(1.5008908 - xhalf*x*x);  // Newton step for a balanced error.
   return x;
}

/* Notes: For more accuracy, repeat the Newton step (just duplicate the
line). The routine always gets the result too low. According to Chris
Lomont, the relative error is at most -0.00175228 (I get -0.00175204).
Therefore, to cut its relative error in half, making it approximately
plus or minus 0.000876, change the 1.5f in the Newton step to 1.500876f
(1.5008908 works best for me, rel err is +-0.0008911).
   Chris says that changing the hex constant to 0x5f375a86 reduces the
maximum relative error slightly, to 0.00175124. (I get 0.00175128. But
the best I can do is use 5f375a82, which gives rel err = 0 to
-0.00175123). However, using that value seems to usually give a slightly
larger relative error, according to Chris.
   If the alternative code is used to do the multiplication by 0.5 with
an integer subtract, the result is
        0x00800000 <= x <= 2.34e-38: inaccurate, but > 6*10e18
        0 < x < 0x00800000: NaN
        x = 0: +inf
   The routine can be adapted to IEEE double precision. */

/* ----------------------------- rsqrt1 ----------------------------- */

/* This is rsqrt with an additional step of the Newton iteration, for
increased accuracy. The constant 0x5f37599e makes the relative error
range from 0 to -0.00000463.
   You can't balance the error by adjusting the constant. */

float rsqrt1(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as an int.
   float xhalf = 0.5f*x;
   ix = 0x5f37599e - (ix >> 1); // Initial guess.
   x = x*(1.5f - xhalf*x*x);    // Newton step.
   x = x*(1.5f - xhalf*x*x);    // Newton step again.
   return x;
}

/* ----------------------------- rsqrt2 ----------------------------- */

/* This is a very approximate but very fast version of rsqrt. It is just
two integer instructions (shift right and subtract), plus instructions
to load the constant.
   The constant 0x5f37642f balances the relative error at +-0.034213.
   The constant 0x5f30c7f0 makes the relative error range from 0 to
-0.061322.
   The constant 0x5f400000 makes the relative error range from 0 to
+0.088662. */

float rsqrt2(float x0) {
   union {int ix; float x;};

   x = x0;                      // x can be viewed as an int.
   ix = 0x5f37642f - (ix >> 1); // Initial guess.
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
   yt = 1.0/sqrt(x);
   ya = rsqrt(x);               // Change to rsqrt1 or rsqrt2 to test
                                // the other routines.
   printf("x = %g %08x\nTrue recip sqrt = %g, approx = %g, rel error = %.7f\n",
      x, ix, yt, ya, (ya - yt)/yt);
   return 0;
}
