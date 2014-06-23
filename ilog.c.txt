// Program for computing integer log functions.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

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

// Integer log base 10, simple table search.
int ilog10a(unsigned x) {
   int i;
   static unsigned table[11] = {0, 9, 99, 999, 9999,
      99999, 999999, 9999999, 99999999, 999999999,
      0xFFFFFFFF};

   for (i = -1; ; i++) {
      if (x <= table[i+1]) return i;
   }
}

// Integer log base 10, repeated multiplication by 10.
int ilog10b(unsigned x) {
   int p, i;

   p = 1;
   for (i = -1; i <= 8; i++) {
      if (x < p) return i;
      p = 10*p;
   }
   return i;
}

// Integer log base 10, modified binary search.
int ilog10c(unsigned x) {
   if (x > 99)
      if (x < 1000000)
         if (x < 10000)
            return 3 + ((int)(x - 1000) >> 31);
         // return 3 - ((x - 1000) >> 31);              // Alternative.
         // return 2 + ((999 - x) >> 31);               // Alternative.
         // return 2 + ((x + 2147482648) >> 31);        // Alternative.
         else
            return 5 + ((int)(x - 100000) >> 31);
      else
         if (x < 100000000)
            return 7 + ((int)(x - 10000000) >> 31);
         else
            return 9 + ((int)((x-1000000000)&~x) >> 31);
         // return 8 + (((x + 1147483648) | x) >> 31);  // Alternative.
   else
      if (x > 9) return 1;
      else       return ((int)(x - 1) >> 31);
   // return ((int)(x - 1) >> 31) | ((unsigned)(9 - x) >> 31);  // Alt.
   // return (x > 9) + (x > 0) - 1;                             // Alt.
}

// Integer log base 10 from log base 2, double table lookup.
int ilog10d(unsigned x) {
   int y;
   static unsigned char table1[33] = {9, 9, 9, 8, 8, 8,
      7, 7, 7, 6, 6, 6, 6, 5, 5, 5, 4, 4, 4, 3, 3, 3, 3,
      2, 2, 2, 1, 1, 1, 0, 0, 0, 0};
   static unsigned table2[10] = {1, 10, 100, 1000, 10000,
      100000, 1000000, 10000000, 100000000, 1000000000};

   y = table1[nlz(x)];
   if (x < table2[y]) y = y - 1;
   return y;
}

// Integer log base 10 from log base 2, double table lookup, branch free.
int ilog10e(unsigned x) {
   int y;
   static unsigned char table1[33] = {10, 9, 9, 8, 8, 8,
      7, 7, 7, 6, 6, 6, 6, 5, 5, 5, 4, 4, 4, 3, 3, 3, 3,
      2, 2, 2, 1, 1, 1, 0, 0, 0, 0};
   static unsigned table2[11] = {1, 10, 100, 1000, 10000,
      100000, 1000000, 10000000, 100000000, 1000000000,
      0};

   y = table1[nlz(x)];
   y = y - ((x - table2[y]) >> 31);
   return y;
}

// Integer log base 10 from log base 2, one table lookup.
int ilog10f(unsigned x) {
   int y;
   static unsigned table2[10] = {0, 9, 99, 999, 9999,
      99999, 999999, 9999999, 99999999, 999999999};

   y = (9*(31 - nlz(x))) >> 5;
   if (x > table2[y+1]) y = y + 1;
   return y;
}

// Integer log base 10 from log base 2, one table lookup, branch free.
int ilog10g(unsigned x) {
   int y;
   static unsigned table2[11] = {0, 9, 99, 999, 9999,
      99999, 999999, 9999999, 99999999, 999999999,
      0xFFFFFFFF};

   y = (19*(31 - nlz(x))) >> 6;
   y = y + ((table2[y+1] - x) >> 31);
   return y;
}

int errors;
void error(int x, int r) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, r);
}

int main() {
   int i, r, n;
   static int test[] = {0,-1, 1,0, 2,0, 3,0, 4,0, 5,0, 6,0,
      7,0, 8,0, 9,0, 10,1, 11,1, 12,1, 13,1, 14,1, 15,1,
      99,1, 100,2, 101,2, 999,2, 1000,3, 1001,3,
      9999,3, 10000,4, 10001,4, 99999,4, 100000,5, 100001,5,
      999999,5, 1000000,6, 1000001,6, 9999999,6, 10000000,7, 10000001,7,
      99999999,7, 100000000,8, 100000001,8, 0xFFFFFFFE,9, 0xFFFFFFFF,9};

   n = sizeof(test)/4;

   printf("ilog10a:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10a(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10b:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10b(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10c:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10c(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10d:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10d(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10e:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10e(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10f:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10f(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("ilog10g:\n");
   for (i = 0; i < n; i += 2) {
      r = ilog10g(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/3);
}
