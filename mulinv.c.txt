#include <stdio.h>
unsigned mulinv(unsigned d) {           // d must be odd.
   unsigned x1, v1, x2, v2, x3, v3, q;

   x1 = 0xFFFFFFFF;     v1 = -d;
   x2 = 1;              v2 = d;
   while (v2 > 1) {
      q = v1/v2;
      x3 = x1 - q*x2;   v3 = v1 - q*v2;
      x1 = x2;          v1 = v2;
      x2 = x3;          v2 = v3;
   }
   return x2;
}

unsigned mulinv2(unsigned d) {          // d must be odd.
   unsigned xn, t;

   xn = d;
loop: t = d*xn;
      if (t == 1) return xn;
      xn = xn*(2 - t);
      goto loop;
}

int errors;
void error(int u, int r) {
   errors = errors + 1;
   printf("Error for u = %08x, got %08x (%d dec)\n", u, r, r);
}

int main() {
   int i, r, n;
   static int test[] = {1,1, 3,0xAAAAAAAB, 5,0xCCCCCCCD,
        7,0xB6DB6DB7,  9,0x38E38E39, 11,0xBA2E8BA3,
       13,0xC4EC4EC5, 15,0xEEEEEEEF, 25,0xC28F5c29,
      125,0x26E978D5, -1,0xFFFFFFFF, -3,0x55555555,
       -5,0x33333333, -7,0x49249249};

   n = sizeof(test)/4;

   printf("mulinv:\n");
   for (i = 0; i < n; i += 2) {
      r = mulinv(test[i]);
      if (r != test[i+1]) error(test[i], r);
   }

   printf("mulinv2:\n");
   for (i = 0; i < n; i += 2) {
      r = mulinv2(test[i]);
      if (r != test[i+1]) error(test[i], r);
   }
   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);
}
