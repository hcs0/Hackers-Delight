// Computes the magic number for signed division.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>
struct ms {int M;          // Magic number
          int s;};         // and shift amount.

int main() {
   struct ms magic(int);
   struct ms mag;

   mag = magic(3);
   printf("M = %08x s = %d\n", mag.M, mag.s);
   if (mag.M + mag.s != 0x55555556 + 0) return 1;
   mag = magic(7);
   printf("M = %08x s = %d\n", mag.M, mag.s);
   if (mag.M + mag.s != 0x92492493 + 2) return 2;
   return 0;
}

struct ms magic(int d) {   // Must have 2 <= d <= 2**31-1
                           // or   -2**31 <= d <= -2.
   int p;
   unsigned ad, anc, delta, q1, r1, q2, r2, t;
   const unsigned two31 = 0x80000000;     // 2**31.
   struct ms mag;

   ad = abs(d);
   t = two31 + ((unsigned)d >> 31);
   anc = t - 1 - t%ad;     // Absolute value of nc.
   p = 31;                 // Init. p.
   q1 = two31/anc;         // Init. q1 = 2**p/|nc|.
   r1 = two31 - q1*anc;    // Init. r1 = rem(2**p, |nc|).
   q2 = two31/ad;          // Init. q2 = 2**p/|d|.
   r2 = two31 - q2*ad;     // Init. r2 = rem(2**p, |d|).
   do {
      p = p + 1;
      q1 = 2*q1;           // Update q1 = 2**p/|nc|.
      r1 = 2*r1;           // Update r1 = rem(2**p, |nc|).
      if (r1 >= anc) {     // (Must be an unsigned
         q1 = q1 + 1;      // comparison here).
         r1 = r1 - anc;}
      q2 = 2*q2;           // Update q2 = 2**p/|d|.
      r2 = 2*r2;           // Update r2 = rem(2**p, |d|).
      if (r2 >= ad) {      // (Must be an unsigned
         q2 = q2 + 1;      // comparison here).
         r2 = r2 - ad;}
      delta = ad - r2;
   } while (q1 < delta || (q1 == delta && r1 == 0));

   mag.M = q2 + 1;
   if (d < 0) mag.M = -mag.M; // Magic number and
   mag.s = p - 32;            // shift amount to return.
   return mag;
}
