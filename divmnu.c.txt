// Computes the n-m+1-halfword quotient and m-halfword remainder
// of n halfwords divided by m halfwords, unsigned.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

#define max(x, y) ((x) > (y) ? (x) : (y))

int nlz(unsigned x) {
   int n;

   if (x == 0) return(32);
   n = 0;
   if (x <= 0x0000FFFF) {n = n +16; x = x <<16;}
   if (x <= 0x00FFFFFF) {n = n + 8; x = x << 8;}
   if (x <= 0x0FFFFFFF) {n = n + 4; x = x << 4;}
   if (x <= 0x3FFFFFFF) {n = n + 2; x = x << 2;}
   if (x <= 0x7FFFFFFF) {n = n + 1;}
   return n;
}

void dumpit(char *msg, int n, unsigned short v[]) {
   int i;
   printf(msg);
   for (i = n-1; i >= 0; i--) printf(" %04x", v[i]);
   printf("\n");
}

/* q[0], r[0], u[0], and v[0] contain the LEAST significant halfwords.
(The sequence is in little-endian order).

This first version is a fairly precise implementation of Knuth's
Algorithm D, for a binary computer with base b = 2**16.  The caller
supplies
   1. Space q for the quotient, m - n + 1 halfwords (at least one).
   2. Space r for the remainder (optional), n halfwords.
   3. The dividend u, m halfwords, m >= 1.
   4. The divisor v, n halfwords, n >= 2.
The most significant digit of the divisor, v[n-1], must be nonzero.  The
dividend u may have leading zeros; this just makes the algorithm take
longer and makes the quotient contain more leading zeros.  A value of
NULL may be given for the address of the remainder to signify that the
caller does not want the remainder.
   The program does not alter the input parameters u and v.
   The quotient and remainder returned may have leading zeros.  The
function itself returns a value of 0 for success and 1 for invalid
parameters (e.g., division by 0).
   For now, we must have m >= n.  Knuth's Algorithm D also requires
that the dividend be at least as long as the divisor.  (In his terms,
m >= 0 (unstated).  Therefore m+n >= n.) */

int divmnu(unsigned short q[], unsigned short r[],
     const unsigned short u[], const unsigned short v[],
     int m, int n) {

   const unsigned b = 65536; // Number base (16 bits).
   unsigned short *un, *vn;  // Normalized form of u, v.
   unsigned qhat;            // Estimated quotient digit.
   unsigned rhat;            // A remainder.
   unsigned p;               // Product of two digits.
   int s, i, j, t, k;

   if (m < n || n <= 0 || v[n-1] == 0)
      return 1;              // Return if invalid param.

   if (n == 1) {                        // Take care of
      k = 0;                            // the case of a
      for (j = m - 1; j >= 0; j--) {    // single-digit
         q[j] = (k*b + u[j])/v[0];      // divisor here.
         k = (k*b + u[j]) - q[j]*v[0];
      }
      if (r != NULL) r[0] = k;
      return 0;
   }

   // Normalize by shifting v left just enough so that
   // its high-order bit is on, and shift u left the
   // same amount.  We may have to append a high-order
   // digit on the dividend; we do that unconditionally.

   s = nlz(v[n-1]) - 16;        // 0 <= s <= 15.
   vn = (unsigned short *)alloca(2*n);
   for (i = n - 1; i > 0; i--)
      vn[i] = (v[i] << s) | (v[i-1] >> 16-s);
   vn[0] = v[0] << s;

   un = (unsigned short *)alloca(2*(m + 1));
   un[m] = u[m-1] >> 16-s;
   for (i = m - 1; i > 0; i--)
      un[i] = (u[i] << s) | (u[i-1] >> 16-s);
   un[0] = u[0] << s;

   for (j = m - n; j >= 0; j--) {       // Main loop.
      // Compute estimate qhat of q[j].
      qhat = (un[j+n]*b + un[j+n-1])/vn[n-1];
      rhat = (un[j+n]*b + un[j+n-1]) - qhat*vn[n-1];
again:
      if (qhat >= b || qhat*vn[n-2] > b*rhat + un[j+n-2])
      { qhat = qhat - 1;
        rhat = rhat + vn[n-1];
        if (rhat < b) goto again;
      }

      // Multiply and subtract.
      k = 0;
      for (i = 0; i < n; i++) {
         p = qhat*vn[i];
         t = un[i+j] - k - (p & 0xFFFF);
         un[i+j] = t;
         k = (p >> 16) - (t >> 16);
      }
      t = un[j+n] - k;
      un[j+n] = t;

      q[j] = qhat;              // Store quotient digit.
      if (t < 0) {              // If we subtracted too
         q[j] = q[j] - 1;       // much, add back.
         k = 0;
         for (i = 0; i < n; i++) {
            t = un[i+j] + vn[i] + k;
            un[i+j] = t;
            k = t >> 16;
         }
         un[j+n] = un[j+n] + k;
      }
   } // End j.
   // If the caller wants the remainder, unnormalize
   // it and pass it back.
   if (r != NULL) {
      for (i = 0; i < n; i++)
         r[i] = (un[i] >> s) | (un[i+1] << 16-s);
   }
   return 0;
}

int errors;

void check(unsigned short q[], unsigned short r[],
           unsigned short u[], unsigned short v[],
           int m, int n,
           unsigned short cq[], unsigned short cr[]) {
   int i, szq;

   szq = max(m - n + 1, 1);
   for (i = 0; i < szq; i++) {
      if (q[i] != cq[i]) {
         errors = errors + 1;
         dumpit("Error, dividend u =", m, u);
         dumpit("       divisor  v =", n, v);
         dumpit("For quotient,  got:", m-n+1, q);
         dumpit("        Should get:", m-n+1, cq);
         return;
      }
   }
   for (i = 0; i < n; i++) {
      if (r[i] != cr[i]) {
         errors = errors + 1;
         dumpit("Error, dividend u =", m, u);
         dumpit("       divisor  v =", n, v);
         dumpit("For remainder, got:", n, r);
         dumpit("        Should get:", n, cr);
         return;
      }
   }
   return;
}

int main() {
   static unsigned short test[] = {
   // m, n, u...,          v...,          cq...,  cr....
      1, 1, 3,             0,             1,      1,
      1, 2, 7,             1,3,           0,      7,0,
      2, 2, 0,0,           1,0,           0,      0,0,
      1, 1, 3,             2,             1,      1,
      1, 1, 3,             3,             1,      0,
      1, 1, 3,             4,             0,      3,
      1, 1, 0,             0xffff,        0,      0,
      1, 1, 0xffff,        1,             0xffff, 0,
      1, 1, 0xffff,        0xffff,        1,      0,
      1, 1, 0xffff,        3,             0x5555, 0,
      2, 1, 0xffff,0xffff, 1,             0xffff,0xffff, 0,
      2, 1, 0xffff,0xffff, 0xffff,        1,1,    0,
      2, 1, 0xffff,0xfffe, 0xffff,        0xffff,0, 0xfffe,
      2, 1, 0x5678,0x1234, 0x9abc,        0x1e1e,0, 0x2c70,
      2, 2, 0,0,           0,1,           0,      0,0,
      2, 2, 0,7,           0,3,           2,      0,1,
      2, 2, 5,7,           0,3,           2,      5,1,
      2, 2, 0,6,           0,2,           3,      0,0,
      2, 2, 0x0001,0x8000, 0x7000,0x4000, 0x0001, 0x9001,0x3fff,
      2, 2, 0x789a,0xbcde, 0x789a,0xbcde, 1,      0,0,
      2, 2, 0x789b,0xbcde, 0x789a,0xbcde, 1,      1,0,
      2, 2, 0x7899,0xbcde, 0x789a,0xbcde, 0,      0x7899,0xbcde,
      2, 2, 0xffff,0xffff, 0xffff,0xffff, 1,      0,0,
      2, 2, 0xffff,0xffff, 0x0000,0x0001, 0xffff, 0xffff,0,
      3, 2, 0x89ab,0x4567,0x0123, 0x0000,0x0001, 0x4567,0x0123, 0x89ab,0,
      3, 2, 0x0000,0xfffe,0x8000, 0xffff,0x8000, 0xffff,0x0000, 0xffff,0x7fff, // Shows that first qhat can = b + 1.
      3, 3, 0x0003,0x0000,0x8000, 0x0001,0x0000,0x2000, 0x0003, 0,0,0x2000, // Adding back step req'd.
      4, 3, 0,0,0x8000,0x7fff, 1,0,0x8000, 0xfffe,0, 2,0xffff,0x7fff,  // Add back req'd.
      4, 3, 0,0xfffe,0,0x8000, 0xffff,0,0x8000, 0xffff,0, 0xffff,0xffff,0x7fff,  // Shows that mult-sub quantity cannot be treated as signed.
   };
   int i, n, m, ncases, f;
   unsigned short q[10], r[10];
   unsigned short *u, *v, *cq, *cr;

   printf("divmnu:\n");
   i = 0;
   ncases = 0;
   while (i < sizeof(test)/2) {
      m = test[i];
      n = test[i+1];
      u = &test[i+2];
      v = &test[i+2+m];
      cq = &test[i+2+m+n];
      cr = &test[i+2+m+n+max(m-n+1, 1)];

      f = divmnu(q, r, u, v, m, n);
      if (f) {
         dumpit("Error return code for dividend u =", m, u);
         dumpit("                      divisor  v =", n, v);
         errors = errors + 1;
      }
      else
         check(q, r, u, v, m, n, cq, cr);
      i = i + 2 + m + n + max(m-n+1, 1) + n;
      ncases = ncases + 1;
   }

   printf("%d errors out of %d cases; there should be 3.\n", errors, ncases);
   return 0;
}
