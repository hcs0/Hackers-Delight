// Programs for incrementing a reversed integer.
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

// Basic code.
unsigned increv1(unsigned x) {
   unsigned m;

   m = 0x80000000;
   x = x ^ m;
   if ((int)x >= 0) {
      do {
         m = m >> 1;
         x = x ^ m;
      } while (x < m);
   }
   return x;
}

// Using nlz.
unsigned increv2(unsigned x) {
   unsigned m, n, s;

// s = nlz(~x);                         // 6 inst'ns, unsigned shifts,
// return x ^ ~(0xFFFFFFFF >> (s+1));   // using equivalence, req's
                                        // mod 64 shifts.

// s = nlz(~x);                         // 6 inst'ns, unsigned shifts,
// return x ^ ~(0x7FFFFFFF >> s);       // using equivalence.

// m = 0x80000000 >> nlz(~x);           // 6 inst'ns, unsigned shifts,
// return x ^ (-m);                     // fails for x = 0xFFFFFFFF.

// s = nlz(~x);                         // 5 inst'ns, unsigned shifts.
// return ((x << s) + 0x80000000) >> s; // A winner.  Req's mod 64
                                        // shifts (and addis).

// s = nlz(~x);                         // 7 inst'ns, fails for x =
// return x ^ ((-1) << (31 - s));       // 0xFFFFFFFF.  Don't bother with it.

// return x + (3 << (31 - nlz(~x)));    // 7 Frisk inst'ns, fails for
//                                      // x = 0xFFFFFFFF.

// m = 0x80000000 >> nlz(~x);           // 7 Frisc inst'ns, fails for
// return x + m + m + m;                // x = 0xFFFFFFFF.

   return x ^ ((int)0x80000000 >> nlz(~x));  // 5 inst'ns, signed shift.
                                        // Req's mod 64 shifts.
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %08x\n", x, y);
}

int main() {
   int i, r, n;
   static int test[] = {0,0x80000000, 1,0x80000001,
      0x80000000,0x40000000, 0x40000000,0xC0000000,
      0xC0000000,0x20000000, 0x20000000,0xA0000000,
      0xA0000000,0x60000000, 0x60000000,0xE0000000,
      0xE0000000,0x10000000, 0x10000000,0x90000000,
      0x90000000,0x50000000, 0x50000000,0xD0000000,
      0xD0000000,0x30000000, 0x30000000,0xB0000000,
      0xB0000000,0x70000000, 0x70000000,0xF0000000,
      0xF0000000,0x08000000, 0x08000000,0x88000000,
      0x3FFFFFFF,0xBFFFFFFF, 0xBFFFFFFF,0x7FFFFFFF,
      0x7FFFFFFF,0xFFFFFFFF, 0xFFFFFFFF,0x00000000};

   n = sizeof(test)/4;

   printf("increv1:\n");
   for (i = 0; i < n; i += 2) {
      r = increv1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("increv2:\n");
   for (i = 0; i < n; i += 2) {
      r = increv2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
}
