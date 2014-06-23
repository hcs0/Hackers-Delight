/* This is similar to hamming.c but the main program does an exhaustive
test, for all 2**32 possible information words, corrupting just one bit
of either the information bits or the check bits (inner loop is done
39*2**32 = 167*10**9 times). Took 101 minutes on my Windows box. */

/* This program generates check bits for a Hamming
SEC-DED code with 32 information bits, and simulates
error creation, detection, and correction for 0-, 1-, and
2- bit errors. The errors may be in the data or the check
bits (or both).

Max line length is 57, to fit in hacker.book. */

#include <stdio.h>
#include <time.h>

// ----------------------------- parity --------------------------------

unsigned int parity(unsigned int x) {
   x = x ^ (x >> 1);
   x = x ^ (x >> 2);
   x = x ^ (x >> 4);
   x = x ^ (x >> 8);
   x = x ^ (x >> 16);
   return x & 1;
}

// --------------------------- checkbits -------------------------------

unsigned int checkbits(unsigned int u) {

   /* Computes the six parity check bits for the
   "information" bits given in the 32-bit word u. The
   check bits are p[5:0]. On sending, an overall parity
   bit will be prepended to p (by another process).

   Bit   Checks these bits of u
   p[0]  0, 1, 3, 5, ..., 31 (0 and the odd positions).
   p[1]  0, 2-3, 6-7, ..., 30-31 (0 and positions xxx1x).
   p[2]  0, 4-7, 12-15, 20-23, 28-31 (0 and posns xx1xx).
   p[3]  0, 8-15, 24-31 (0 and positions x1xxx).
   p[4]  0, 16-31 (0 and positions 1xxxx).
   p[5]  1-31 */

   unsigned int p0, p1, p2, p3, p4, p5, p6, p;
   unsigned int t1, t2, t3;

   // First calculate p[5:0] ignoring u[0].
   p0 = u ^ (u >> 2);
   p0 = p0 ^ (p0 >> 4);
   p0 = p0 ^ (p0 >> 8);
   p0 = p0 ^ (p0 >> 16);        // p0 is in posn 1.

   t1 = u ^ (u >> 1);
   p1 = t1 ^ (t1 >> 4);
   p1 = p1 ^ (p1 >> 8);
   p1 = p1 ^ (p1 >> 16);        // p1 is in posn 2.

   t2 = t1 ^ (t1 >> 2);
   p2 = t2 ^ (t2 >> 8);
   p2 = p2 ^ (p2 >> 16);        // p2 is in posn 4.

   t3 = t2 ^ (t2 >> 4);
   p3 = t3 ^ (t3 >> 16);        // p3 is in posn 8.

   p4 = t3 ^ (t3 >> 8);         // p4 is in posn 16.

   p5 = p4 ^ (p4 >> 16);        // p5 is in posn 0.

   p = ((p0>>1) & 1) | ((p1>>1) & 2) | ((p2>>2) & 4) |
       ((p3>>5) & 8) | ((p4>>12) & 16) | ((p5 & 1) << 5);

   p = p ^ (-(u & 1) & 0x3F);   // Now account for u[0].
   return p;
}

// ---------------------------- correct --------------------------------

int correct(unsigned int pr, unsigned int *ur) {

   /* This function looks at the received seven check
   bits and 32 information bits (pr and ur), and
   determines how many errors occurred (under the
   presumption that it must be 0, 1, or 2). It returns
   with 0, 1, or 2, meaning that no errors, one error, or
   two errors occurred. It corrects the information word
   received (ur) if there was one error in it. */

   unsigned int po, p, syn, b;

   po = parity(pr ^ *ur);       // Compute overall parity
                                // of the received data.
   p = checkbits(*ur);          // Calculate check bits
                                // for the received info.
   syn = p ^ (pr & 0x3F);       // Syndrome (exclusive of
                                // overall parity bit).
   if (po == 0) {
      if (syn == 0) return 0;   // If no errors, return 0.
      else return 2;            // Two errors, return 2.
   }
                                // One error occurred.
   if (((syn - 1) & syn) == 0)  // If syn has zero or one
      return 1;                 // bits set, then the
                                // error is in the check
                                // bits or the overall
                                // parity bit (no
                                // correction required).

   // One error, and syn bits 5:0 tell where it is in ur.

   b = syn - 31 - (syn >> 5); // Map syn to range 0 to 31.
// if (syn == 0x1f) b = 0;    // (These two lines equiv.
// else b = syn & 0x1f;       // to the one line above.)
   *ur = *ur ^ (1 << b);      // Correct the bit.
   return 1;
}

// ------------------------------ main ---------------------------------

int main(int argc, char *argv[]) {

   unsigned int us, ur, uc;     // Information words, sent, received,
                                // and corrected.
   unsigned int ps, pr;         // Check bits sent and received.
   int e, c;                    // Number of errors made and detected.
   int i;
   clock_t cloc;
   time_t start_time;

   cloc = clock();
   start_time = time(NULL);
   printf("----sent----   --received--          corrected\n");
   printf("ps     us      pr     ur      e   c      uc\n");

   us = 0xFFFFFFFF;                        // This loops through all
   do {                                    // 2**32 values of us
      us = us + 1;                         // (information bits sent).
      for (i = 0; i < 39; i++) {           // Bit i will be flipped.
         ps = checkbits(us);               // Compute the 6 check bits
         ps = ps | (parity(us ^ ps) << 6); // and prepend the overall
                                           // parity bit.

         /* Corrupt us or ps by flipping one bit, bit i, producing
         ur and pr (information and parity, resp., received). */

         if (i <= 31) {
            ur = us ^ (1 << i);
            pr = ps;
         }
         else {
            ur = us;
            pr = ps ^ (1 << (i - 32));
         }
         e = 1;                    // One bit was corrupted.

         uc = ur;
         c = correct(pr, &uc);     // Correct ur (1 error occurred).

         if ((us & 0xFFFFFF) == 0) {
            printf("%02x  %08x   %02x  %08x   ", ps, us, pr, ur); // Program
            printf("%d   %d   %08x\n", e, c, uc);                 // trace.
         }

         if (e != c) {
            printf("Program error, e = %d, c = %d\n", e, c);
            return 1;
         }

         if (e <= 1 && uc != us) {
            printf("Program error, e = %d, us = %08x, uc = %08x\n", e, us, uc);
            return 1;
         }
      }
   } while (us < 0xFFFFFFFF);   // End of us loop.

   printf("hamming_exhaustive_1 ended, no errors.\n");
   printf("Process time = %4.2f secs\n", (float)(clock() - cloc)/CLOCKS_PER_SEC);
   printf("Wall clock time = %d secs\n", (int)difftime(time(NULL), start_time));
   return 0;
}
