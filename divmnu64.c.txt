/* This divides an n-word dividend by an m-word divisor, giving an
n-m+1-word quotient and m-word remainder. The bignums are in arrays of
words. Here a "word" is 32 bits. This routine is designed for a 64-bit
machine which has a 64/64 division instruction. */

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

void dumpit(char *msg, int n, unsigned v[]) {
   int i;
   printf(msg);
   for (i = n-1; i >= 0; i--) printf(" %08x", v[i]);
   printf("\n");
}

/* q[0], r[0], u[0], and v[0] contain the LEAST significant words.
(The sequence is in little-endian order).

This is a fairly precise implementation of Knuth's Algorithm D, for a
binary computer with base b = 2**32. The caller supplies:
   1. Space q for the quotient, m - n + 1 words (at least one).
   2. Space r for the remainder (optional), n words.
   3. The dividend u, m words, m >= 1.
   4. The divisor v, n words, n >= 2.
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

int divmnu(unsigned q[], unsigned r[],
     const unsigned u[], const unsigned v[],
     int m, int n) {

   const unsigned long long b = 4294967296LL; // Number base (2**32).
   unsigned *un, *vn;                         // Normalized form of u, v.
   unsigned long long qhat;                   // Estimated quotient digit.
   unsigned long long rhat;                   // A remainder.
   unsigned long long p;                      // Product of two digits.
   long long t, k;
   int s, i, j;

   if (m < n || n <= 0 || v[n-1] == 0)
      return 1;                         // Return if invalid param.

   if (n == 1) {                        // Take care of
      k = 0;                            // the case of a
      for (j = m - 1; j >= 0; j--) {    // single-digit
         q[j] = (k*b + u[j])/v[0];      // divisor here.
         k = (k*b + u[j]) - q[j]*v[0];
      }
      if (r != NULL) r[0] = k;
      return 0;
   }

   /* Normalize by shifting v left just enough so that its high-order
   bit is on, and shift u left the same amount. We may have to append a
   high-order digit on the dividend; we do that unconditionally. */

   s = nlz(v[n-1]);             // 0 <= s <= 31.
   vn = (unsigned *)alloca(4*n);
   for (i = n - 1; i > 0; i--)
      vn[i] = (v[i] << s) | ((unsigned long long)v[i-1] >> (32-s));
   vn[0] = v[0] << s;

   un = (unsigned *)alloca(4*(m + 1));
   un[m] = (unsigned long long)u[m-1] >> (32-s);
   for (i = m - 1; i > 0; i--)
      un[i] = (u[i] << s) | ((unsigned long long)u[i-1] >> (32-s));
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
         t = un[i+j] - k - (p & 0xFFFFFFFFLL);
         un[i+j] = t;
         k = (p >> 32) - (t >> 32);
      }
      t = un[j+n] - k;
      un[j+n] = t;

      q[j] = qhat;              // Store quotient digit.
      if (t < 0) {              // If we subtracted too
         q[j] = q[j] - 1;       // much, add back.
         k = 0;
         for (i = 0; i < n; i++) {
            t = (unsigned long long)un[i+j] + vn[i] + k;
            un[i+j] = t;
            k = t >> 32;
         }
         un[j+n] = un[j+n] + k;
      }
   } // End j.
   // If the caller wants the remainder, unnormalize
   // it and pass it back.
   if (r != NULL) {
      for (i = 0; i < n-1; i++)
         r[i] = (un[i] >> s) | ((unsigned long long)un[i+1] << (32-s));
      r[n-1] = un[n-1] >> s;
   }
   return 0;
}

int errors;

void check(unsigned q[], unsigned r[],
           unsigned u[], unsigned v[],
           int m, int n,
           unsigned cq[], unsigned cr[]) {
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
   static unsigned test[] = {
   // m, n, u...,          v...,          cq...,  cr....
      1, 1, 3,             0,             1,      1,            // Error, divide by 0.
      1, 2, 7,             1,3,           0,      7,0,          // Error, n > m.
      2, 2, 0,0,           1,0,           0,      0,0,          // Error, incorrect remainder cr.
      1, 1, 3,             2,             1,      1,
      1, 1, 3,             3,             1,      0,
      1, 1, 3,             4,             0,      3,
      1, 1, 0,             0xffffffff,    0,      0,
      1, 1, 0xffffffff,    1,             0xffffffff, 0,
      1, 1, 0xffffffff,    0xffffffff,    1,      0,
      1, 1, 0xffffffff,    3,             0x55555555, 0,
      2, 1, 0xffffffff,0xffffffff, 1,     0xffffffff,0xffffffff, 0,
      2, 1, 0xffffffff,0xffffffff, 0xffffffff,        1,1,    0,
      2, 1, 0xffffffff,0xfffffffe, 0xffffffff,        0xffffffff,0, 0xfffffffe,
      2, 1, 0x00005678,0x00001234, 0x00009abc,        0x1e1dba76,0, 0x6bd0,
      2, 2, 0,0,           0,1,           0,      0,0,
      2, 2, 0,7,           0,3,           2,      0,1,
      2, 2, 5,7,           0,3,           2,      5,1,
      2, 2, 0,6,           0,2,           3,      0,0,
      1, 1, 0x80000000,  0x40000001, 0x00000001, 0x3fffffff,
      2, 1, 0x00000000,0x80000000, 0x40000001, 0xfffffff8,0x00000001, 0x00000008,
      2, 2, 0x00000000,0x80000000, 0x00000001,0x40000000, 0x00000001, 0xffffffff,0x3fffffff,
      2, 2, 0x0000789a,0x0000bcde, 0x0000789a,0x0000bcde,          1,          0,0,
      2, 2, 0x0000789b,0x0000bcde, 0x0000789a,0x0000bcde,          1,          1,0,
      2, 2, 0x00007899,0x0000bcde, 0x0000789a,0x0000bcde,          0, 0x00007899,0x0000bcde,
      2, 2, 0x0000ffff,0x0000ffff, 0x0000ffff,0x0000ffff,          1,          0,0,
      2, 2, 0x0000ffff,0x0000ffff, 0x00000000,0x00000001, 0x0000ffff, 0x0000ffff,0,
      3, 2, 0x000089ab,0x00004567,0x00000123, 0x00000000,0x00000001,   0x00004567,0x00000123, 0x000089ab,0,
      3, 2, 0x00000000,0x0000fffe,0x00008000, 0x0000ffff,0x00008000,   0xffffffff,0x00000000, 0x0000ffff,0x00007fff, // Shows that first qhat can = b + 1.
      3, 3, 0x00000003,0x00000000,0x80000000, 0x00000001,0x00000000,0x20000000,   0x00000003, 0,0,0x20000000, // Adding back step req'd.
      3, 3, 0x00000003,0x00000000,0x00008000, 0x00000001,0x00000000,0x00002000,   0x00000003, 0,0,0x00002000, // Adding back step req'd.
      4, 3, 0,0,0x00008000,0x00007fff, 1,0,0x00008000,   0xfffe0000,0, 0x00020000,0xffffffff,0x00007fff,  // Add back req'd.
      4, 3, 0,0x0000fffe,0,0x00008000, 0x0000ffff,0,0x00008000, 0xffffffff,0, 0x0000ffff,0xffffffff,0x00007fff,  // Shows that mult-sub quantity cannot be treated as signed.
      4, 3, 0,0xfffffffe,0,0x80000000, 0x0000ffff,0,0x80000000, 0x00000000,1, 0x00000000,0xfffeffff,0x00000000,  // Shows that mult-sub quantity cannot be treated as signed.
      4, 3, 0,0xfffffffe,0,0x80000000, 0xffffffff,0,0x80000000, 0xffffffff,0, 0xffffffff,0xffffffff,0x7fffffff,  // Shows that mult-sub quantity cannot be treated as signed.
   };
   int i, n, m, ncases, f;
   unsigned q[10], r[10];
   unsigned *u, *v, *cq, *cr;

   printf("divmnu:\n");
   i = 0;
   ncases = 0;
   while (i < sizeof(test)/4) {
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
