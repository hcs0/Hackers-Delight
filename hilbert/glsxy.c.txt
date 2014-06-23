/* Rewrite of lamxy.cc by GLS.  Straight-line code, constant running
time of about 63 basic RISC instructions.  (plus prolog & epilog
overhead).  */

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

/* This is the standard for comparison: */

void hil_xy_from_s(unsigned s, int n, unsigned *xp,
                                      unsigned *yp) {

   int i, sa, sb;
   unsigned x, y, temp;

   for (i = 0; i < 2*n; i += 2) {
      sa = (s >> (i+1)) & 1;      // Get bit i+1 of s.
      sb = (s >> i) & 1;          // Get bit i of s.

      if ((sa ^ sb) == 0) {       // If sa,sb = 00 or 11,
         temp = x;                // swap x and y,
         x = y^(-sa);             // and if sa = 1,
         y = temp^(-sa);          // complement them.
      }
      x = (x >> 1) | (sa << 31);  // Prepend sa to x and
      y = (y >> 1) | ((sa ^ sb) << 31); // (sa^sb) to y.
   }
   *xp = x >> (32 - n);           // Right-adjust x and y
   *yp = y >> (32 - n);           // and return them to
}                                 // the caller.

// ------------------------------ cut ----------------------------------
void hil_xy_from_s3(unsigned s, int n, unsigned *xp,
                                      unsigned *yp) {
   unsigned comp, swap, cs, t, sr;

   s = s | (0x55555555 << 2*n); // Pad s on left with 01
   sr = (s >> 1) & 0x55555555;  // (no change) groups.
   cs = ((s & 0x55555555) + sr) // Compute complement &
        ^ 0x55555555;           // swap info in two-bit
                                // groups.
   // Parallel prefix xor op to propagate both complement
   // and swap info together from left to right (there is
   // no step "cs ^= cs >> 1", so in effect it computes
   // two independent parallel prefix operations on two
   // interleaved sets of sixteen bits).

   cs = cs ^ (cs >> 2);
   cs = cs ^ (cs >> 4);
   cs = cs ^ (cs >> 8);
   cs = cs ^ (cs >> 16);
   swap = cs & 0x55555555;      // Separate the swap and
   comp = (cs >> 1) & 0x55555555;  // complement bits.

   t = (s & swap) ^ comp;       // Calculate x and y in
   s = s ^ sr ^ t ^ (t << 1);   // the odd & even bit
                                // positions, resp.
   s = s & ((1 << 2*n) - 1);    // Clear out any junk
                                // on the left (unpad).

   // Now "unshuffle" to separate the x and y bits.

   t = (s ^ (s >> 1)) & 0x22222222; s = s ^ t ^ (t << 1);
   t = (s ^ (s >> 2)) & 0x0C0C0C0C; s = s ^ t ^ (t << 2);
   t = (s ^ (s >> 4)) & 0x00F000F0; s = s ^ t ^ (t << 4);
   t = (s ^ (s >> 8)) & 0x0000FF00; s = s ^ t ^ (t << 8);

   *xp = s >> 16;               // Assign the two halves
   *yp = s & 0xFFFF;            // of t to x and y.
}
// ------------------------------ cut ----------------------------------

int main(int argc, char *argv[]) {
   unsigned n, N, s, x1, y1, x3, y3;

   if (argc <= 1) {
      printf("Need one parameter, the order of the curve.\n");
      exit(1);
   }
   n = atoi(argv[1]);
   N = 1 << 2*n;                        // N = 2**2n.
   printf("(x, y) coordinates along the Hilbert curve of order %d\n", n);
   for (s = 0; s < N; s++) {
      hil_xy_from_s (s, n, &x1, &y1);          // Pass addresses of x and y.
      hil_xy_from_s3(s, n, &x3, &y3);          // Pass addresses of x and y.
      printf("%5d %5d %5d\n", s, x3, y3);
      if (x1 != x3 || y1 != y3)
         printf("********** Error at s = %d **********\n", s);
   }
   return 0;
}
