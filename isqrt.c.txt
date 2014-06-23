// Programs for computing the integer square root.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

int nlz(unsigned x) {
   int nbits(unsigned x);

   x = x | (x >> 1);
   x = x | (x >> 2);
   x = x | (x >> 4);
   x = x | (x >> 8);
   x = x | (x >>16);
   return nbits(~x);
}

int nbits(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

// ------------------------------ cut1 ---------------------------------
int isqrt1(unsigned x) {
   unsigned x1;
   int s, g0, g1;

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
   }
   return g0;
}
// ------------------------------ cut1 ---------------------------------

// ------------------------------ cut2 ---------------------------------
int isqrt2(unsigned x) {
   int s, g0, g1;

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

   // Continue as in the previous program.
// ------------------------------ cut2 ---------------------------------

   g1 = (g0 + (x >> s)) >> 1;  // g1 = (g0 + x/g0)/2.

   while (g1 < g0) {           // Do while approximations
      g0 = g1;                 // strictly decrease.
      g1 = (g0 + (x/g0)) >> 1;
   }
   return g0;
}

// ------------------------------ cut3 ---------------------------------
int isqrt3(unsigned x) {
   unsigned a, b, m;            // Limits and midpoint.

   a = 1;
   b = (x >> 5) + 8;            // See text.
   if (b > 65535) b = 65535;
   do {
      m = (a + b) >> 1;
      if (m*m > x) b = m - 1;
      else         a = m + 1;
   } while (b >= a);
   return a - 1;
}
// ------------------------------ cut3 ---------------------------------

// Hardware algorithm [GLS]
// ------------------------------ cut4 ---------------------------------
int isqrt4(unsigned x) {
   unsigned m, y, b;

   m = 0x40000000;
   y = 0;
   while(m != 0) {              // Do 16 times.
      b = y | m;
      y = y >> 1;
      if (x >= b) {
         x = x - b;
         y = y | m;
      }
      m = m >> 2;
   }
   return y;
}
// ------------------------------ cut4 ---------------------------------

/* A 64-bit version of the above is easy to obtain. Change the
declaration of the function from int to unsigned int, change the
parameter x and the three automatic variables from unsigned to unsigned
long long, and change the first assignment to m to "m =
0x4000000000000000LL;". The while-loop will be iterated 32 times. */

/* Modification of isqrt4 that avoids the branch in the loop. */

int isqrt5(unsigned x) {
   unsigned m, y, b;
   int t;

   m = 0x40000000;
   y = 0;
   while(m != 0) {              // Do 16 times.
      b = y | m;
      y = y >> 1;
      t = (int)(x | ~(x - b)) >> 31; // -1 if x >= b, else 0.
      x = x - (b & t);
      y = y | (m & t);
      m = m >> 2;
   }
   return y;
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, r, n;
   static unsigned test[] = {0,0, 1,1, 2,1, 3,1, 4,2, 5,2,
      6,2, 7,2, 8,2, 9,3, 10,3, 11,3, 12,3, 13,3, 14,3,
      15,3, 16,4, 17,4, 18,4, 19,4, 20,4, 21,4, 22,4, 23,4,
      24,4, 25,5, 26,5, 27,5, 28,5, 29,5, 30,5, 31,5,
      32,5, 33,5, 34,5, 35,5, 36,6, 37,6, 38,6, 39,6, 40,6,
      99,9, 100,10, 101,10, 289,17, 65535,255, 65536,256,
      65537,256, 1073741823,32767, 1073741824,32768,
      1073741825,32768, 0x80000000,46340, 0xFFFFFFFF,65535};

   n = sizeof(test)/4;

   printf("isqrt1:\n");
   for (i = 0; i < n; i += 2) {
      r = isqrt1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("isqrt2:\n");
   for (i = 0; i < n; i += 2) {
      r = isqrt2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("isqrt3:\n");
   for (i = 0; i < n; i += 2) {
      r = isqrt3(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("isqrt4:\n");
   for (i = 0; i < n; i += 2) {
      r = isqrt4(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("isqrt5:\n");
   for (i = 0; i < n; i += 2) {
      r = isqrt5(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
   return errors;
}
