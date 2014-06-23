/* This program generates check bits for a Hamming
SEC-DED code with 32 information bits, and simulates
error creation, detection, and correction for 0-, 1-, and
2- bit errors. The errors may be in the data or the check
bits (or both).

Max line length is 57, to fit in hacker.book. */

#include <stdio.h>
#include <stdlib.h>     // For rand().

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

// ---------------------------- perturb --------------------------------

int perturb(unsigned int *p, unsigned int *u) {

   /* This generates all the possible 39-bit quantities with 0, 1, or 2
   bits set, and alters the corresponding 0, 1, or 2 bits of p and u,
   treating them as a concatenation of p and u (39 bits long).
      The error bit words are generated in the order (illustrated for a
   5-bit quantitity):

   00011, 00101, 01001, 10001, 00001, 00110, 01010, 10010, 00010,
   01100, 10100, 00100, 11000, 01000, 10000, 00000. */

   static unsigned long long mask = (1LL << 39) - 1;
   static unsigned long long x = 1, y = 2;
   unsigned long long errorBits;
   int num;

   errorBits = x | y;

   if (errorBits == 0) num = 0;         // Set num = number
   else if (x == 0 || y == 0) num = 1;  // of 1-bits in
   else num = 2;                        // errorBits.

   *u = *u ^ (unsigned)errorBits;       // Apply the
   *p = *p ^ (errorBits >> 32);         // error bits.

   if (y != 0) y = (y << 1) & mask;
   else {x = (x << 1) & mask; y = (x << 1) & mask;}
   return num;
}

// ---------------------------- correct --------------------------------

int correct(unsigned int pr, unsigned int *ur) {

   /* This function looks at the received seven check
   bits and 32 information bits (pr and ur) and
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

   printf("----sent----   --received--          corrected\n");
   printf("ps     us      pr     ur      e   c      uc\n");
   for (i = 0; i < (39*40)/2 + 1; i++) {
      us = rand() * 3;          // Generate random information bits
                                // (rand() always has the msb = 0).
      ps = checkbits(us);               // Compute their 6 check bits
      ps = ps | (parity(us ^ ps) << 6); // and prepend the overall
                                        // parity bit.
      ur = us;                  // Set up the received data.
      pr = ps;
      e = perturb(&pr, &ur);    // Alter 0, 1, or 2 bits of pr and ur.
      uc = ur;
      c = correct(pr, &uc);     // Correct ur if 1 error occurred.

      printf("%02x  %08x   %02x  %08x   ", ps, us, pr, ur); // Program
      printf("%d   %d   %08x\n", e, c, uc);                 // trace.

      if (e != c) {
         printf("Program error, e = %d, c = %d\n", e, c);
         return 1;
      }

      if (e <= 1 && uc != us) {
         printf("Program error, e = %d, us = %08x, uc = %08x\n", e, us, uc);
         return 1;
      }
   }

   return 0;
}
