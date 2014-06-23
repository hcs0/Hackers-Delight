/* Given a, b, c, and d, this program computes the min value of x | y,
where a <= x <= b and c <= y <= d (signed numbers).
   Not explicitly in hacker.book. */

#include <stdio.h>

#define min(x, y) ((x) < (y) ? (x) : (y))

unsigned minOR(unsigned a, unsigned b,
               unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (~a & c & m) {
         temp = (a | m) & -m;
         if (temp <= b) {a = temp; break;}
      }
      else if (a & ~c & m) {
         temp = (c | m) & -m;
         if (temp <= d) {c = temp; break;}
      }
      m = m >> 1;
   }
   return a | c;
}

int brute(int a, int b, int c, int d) {

   int i, j, rmin;

   rmin = 0x7FFFFFFF;           // Init to "infinity."
   for (i = a; i <= b; i++) {
      for (j = c; j <= d; j++) {
         if ((i | j) < rmin) rmin = i | j;
      }
   }
   return rmin;
}

int main() {
   int n, nmax, nmin, a, b, c, d, s, rmin, r1;

   n = 5;                       // Size of problem.
   nmax = 1 << (n-1);           // 2**(n-1).
   nmin = -nmax;
   s = 0x80000000;

   for (a = nmin; a < nmax; a++) {
      for (b = a; b < nmax; b++) {
         for (c = nmin; c < nmax; c++) {
            for (d = c; d < nmax; d++) {
               rmin = brute(a, b, c, d);        // Correct result.

               if      (a< 0 && b< 0 && c< 0 && d< 0) r1 = minOR(a, b, c, d);
               else if (a< 0 && b< 0 && c< 0 && d>=0) r1 = a;
               else if (a< 0 && b< 0 && c>=0 && d>=0) r1 = minOR(a, b, c, d); // a | c doesn't work.
               else if (a< 0 && b>=0 && c< 0 && d< 0) r1 = c;
               else if (a< 0 && b>=0 && c< 0 && d>=0) r1 = min(a, c);
               else if (a< 0 && b>=0 && c>=0 && d>=0) r1 = minOR(a, 0xFFFFFFFF, c, d); // Or, minOR(a+s, b+s, c, d) - s.
               else if (a>=0 && b>=0 && c< 0 && d< 0) r1 = minOR(a, b, c, d);
               else if (a>=0 && b>=0 && c< 0 && d>=0) r1 = minOR(a, b, c, 0xFFFFFFFF); // Or, minOR(a, b, c+s, d+s) - s.
               else if (a>=0 && b>=0 && c>=0 && d>=0) r1 = minOR(a, b, c, d);
               else {printf("What???\n"); return 1;}

/*             printf("minORs(%08x %08x %08x %08x) = %08x\n", a, b, c, d, r1); */
               if (r1 != rmin) {
                  printf("ERROR, %08x <= x <= %08x, %08x <= y <= %08x\n"
                         "r1 = %08x, rmin = %08x\n", a, b, c, d, r1, rmin);
                  return 1;
               }
            }
         }
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
