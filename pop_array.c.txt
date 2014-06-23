/* Computing the number of 1-bits in an array. This is the code in Fig.
5-5 on page 73 of Hacker's Delight, first edition. There are better ways
to do this, given in newCode/pop_arrayHS.cc.
   Max line length is 57, to fit in hacker.book. */
#include <stdio.h>
inline int min(int x, int y) {return x < y ? x : y;}
inline int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x >> 8);
   x = x + (x >> 16);
   return x & 0x0000003F;
}

// ------------------------------ cut ----------------------------------
int pop_array(unsigned A[], int n) {

   int i, j, lim;
   unsigned s, s8, x;

   s = 0;
   for (i = 0; i < n; i = i + 31) {
      lim = min(n, i + 31);
      s8 = 0;
      for (j = i; j < lim; j++) {
         x = A[j];
         x = x - ((x >> 1) & 0x55555555);
         x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
         x = (x + (x >> 4)) & 0x0F0F0F0F;
         s8 = s8 + x;
      }
      x = (s8 & 0x00FF00FF) + ((s8 >> 8) & 0x00FF00FF);
      x = (x & 0x0000ffff) + (x >> 16);
      s = s + x;
   }
   return s;
}
// ---------------------------- end cut --------------------------------

int main(void) {
   unsigned A[10000];
   int i, n, s1, s2;

   n = sizeof(A)/sizeof(A[0]);
   for (i = 0; i < n; i++) {A[i] = 0xffffffff;}

   s1 = 0;
   for (i = 0; i < n; i++) {
      s1 = s1 + pop(A[i]);
   }
   printf("s1 = %d\n", s1);

   s2 = pop_array(A, n);
   printf("s2 = %d\n", s2);
   return s2 - s1;
}
