// Programs for computing the integer square root.
// Max line length is 57, to fit in hacker.book.
// This routine evaluates exhaustively the first integer sqrt routine.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

int ndiv;

int isqrt1(unsigned x) {
   unsigned x1;
   int s, g0, g1;

   ndiv = 0;
   if (x <= 1) return x;
   s = 1;
   x1 = x - 1;
   if (x1 > 65535) {s = s + 8; x1 = x1 >> 16;}
   if (x1 > 255)   {s = s + 4; x1 = x1 >> 8;}
   if (x1 > 15)    {s = s + 2; x1 = x1 >> 4;}
   if (x1 > 3)     {s = s + 1;}

   g0 = 1 << s;                // g0 = 2**s.
   g1 = (g0 + (x >> s)) >> 1;  // g1 = (g0 + x/g0)/2.

   while (g1 < g0) {           // Do while approximations
      g0 = g1;                 // strictly decrease.
      g1 = (g0 + (x/g0)) >> 1;
      ndiv = ndiv + 1;
   }
   return g0;
}

int isqrt2(unsigned x) {
   int s, g0, g1;

   ndiv = 0;
   if (x <= 4224)
      if (x <= 24)
         if (x <= 3) return (x + 3) >> 2;
         else if (x <= 8) return 2;
         else return (x >> 4) + 3;
      else if (x <= 288)
         if (x <= 80) s = 3; else s = 4;
      else if (x <= 1088) s = 5; else s = 6;
   else if (x <= 1025*1025 - 1)
      if (x <= 257*257 - 1)
         if (x <= 129*129 - 1) s = 7; else s = 8;
      else if (x <= 513*513 - 1) s = 9; else s = 10;
   else if (x <= 4097*4097 - 1)
      if (x <= 2049*2049 - 1) s = 11; else s = 12;
   else if (x <= 16385*16385 - 1)
      if (x <= 8193*8193 - 1) s = 13; else s = 14;
   else if (x <= 32769*32769 - 1) s = 15; else s = 16;
   g0 = 1 << s;                // g0 = 2**s.
   g1 = (g0 + (x >> s)) >> 1;  // g1 = (g0 + x/g0)/2.

   while (g1 < g0) {           // Do while approximations
      g0 = g1;                 // strictly decrease.
      g1 = (g0 + (x/g0)) >> 1;
      ndiv = ndiv + 1;
   }
   return g0;
}

int isqrt3(unsigned x) {
   unsigned a, b, m;            // Limits and midpoint.

   ndiv = 0;                    // Loop counter, not division counter.
   a = 1;
   b = (x >> 5) + 8;            // See text.
   if (b > 65535) b = 65535;
   do {
      m = (a + b) >> 1;
      if (m*m > x) b = m - 1;
      else         a = m + 1;
      ndiv = ndiv + 1;
   } while (b >= a);
   return a - 1;
}


int main() {
   unsigned i, r, n;
   int max_ndiv = 0;
   long long tot_ndiv = 0;

   n = 9999;

   printf("isqrt3:\n");         // Set to isqrt1, isqrt2, or isqrt3.
   for (i = 0; i <= n; i++) {
      r = isqrt3(i);            // Here also.
      tot_ndiv = tot_ndiv + ndiv;
      if (ndiv > max_ndiv) max_ndiv = ndiv;
   }

   printf("max = %d, avg = %9.4f\n", max_ndiv, double(tot_ndiv)/double(n+1));
}
