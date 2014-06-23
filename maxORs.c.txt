/* Given a, b, c, and d, this program computes the max value of x | y,
where a <= x <= b and c <= y <= d (signed numbers).
   Not explicitly in hacker.book. */

#include <stdio.h>

unsigned maxOR(unsigned a, unsigned b,
               unsigned c, unsigned d) {
   unsigned m, temp;

   m = 0x80000000;
   while (m != 0) {
      if (b & d & m) {
         temp = (b - m) | (m - 1);
         if (temp >= a) {b = temp; break;}
         temp = (d - m) | (m - 1);
         if (temp >= c) {d = temp; break;}
      }
      m = m >> 1;
   }
   return b | d;
}

int brute(int a, int b, int c, int d) {

   int i, j, rmax;

   rmax = 0x80000000;           // Init. to max neg. no.
   for (i = a; i <= b; i++) {
      for (j = c; j <= d; j++) {
         if ((i | j) > rmax) rmax = i | j;
      }
   }
   return rmax;
}

int main() {
   int n, nmax, nmin, a, b, c, d, s, rmax, r1;

   n = 5;                       // Size of problem.
   nmax = 1 << (n-1);           // 2**(n-1).
   nmin = -nmax;
   s = 0x80000000;

   for (a = nmin; a < nmax; a++) {
      for (b = a; b < nmax; b++) {
         for (c = nmin; c < nmax; c++) {
            for (d = c; d < nmax; d++) {
               rmax = brute(a, b, c, d);        // Correct result.

               if      (a< 0 && b< 0 && c< 0 && d< 0) r1 = maxOR(a, b, c, d);
               else if (a< 0 && b< 0 && c< 0 && d>=0) r1 = -1;
               else if (a< 0 && b< 0 && c>=0 && d>=0) r1 = maxOR(a, b, c, d);
               else if (a< 0 && b>=0 && c< 0 && d< 0) r1 = -1;
               else if (a< 0 && b>=0 && c< 0 && d>=0) r1 = maxOR(0, b, 0, d);
               else if (a< 0 && b>=0 && c>=0 && d>=0) r1 = maxOR(0, b, c, d);
               else if (a>=0 && b>=0 && c< 0 && d< 0) r1 = maxOR(a, b, c, d);
               else if (a>=0 && b>=0 && c< 0 && d>=0) r1 = maxOR(a, b, 0, d);
               else if (a>=0 && b>=0 && c>=0 && d>=0) r1 = maxOR(a, b, c, d);
               else {printf("What???\n"); return 1;}

/*             printf("minORs(%08x %08x %08x %08x) = %08x\n", a, b, c, d, r1); */
               if (r1 != rmax) {
                  printf("ERROR, %08x <= x <= %08x, %08x <= y <= %08x\n"
                         "r1 = %08x, rmax = %08x\n", a, b, c, d, r1, rmax);
                  return 1;
               }
            }
         }
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
