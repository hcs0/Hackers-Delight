/* Given a, b, c, and d, this program computes the min and max values
of x + y, where a <= x <= b and c <= y <= d (signed numbers).
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

void boundsAdds(int a, int b, int c,
                int d, int *ps, int *pt) {

   int s, t, u, v;

// ------------------------------ cut ----------------------------------
   s = a + c;
   t = b + d;
   u = a & c & ~s & ~(b & d & ~t);
   v = ((a ^ c) | ~(a ^ s)) & (~b & ~d & t);
   if ((u | v) < 0) {
      s = 0x80000000;
      t = 0x7FFFFFFF;}
// ------------------------------ cut ----------------------------------
   *ps = s;
   *pt = t;
   return;
}

// ------------------------------ main ---------------------------------

int main() {
   static struct {int a, b, c, d, e, f;} u[] =
   //      a           b           c           d           e          f
   { {0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000},
     {         3,          6,          2,          7,          5,         13},
     {        -6,         -3,         -7,         -2,        -13,         -5},
     {0xbffffffe, 0xbfffffff, 0xbfffffff, 0xc0000000, 0x7ffffffd, 0x7fffffff},
     {0xc0000000, 0xc0000000, 0xbfffffff, 0xc0000000, 0x80000000, 0x7fffffff},
     {0xc0000000,          3, 0xbfffffff,          4, 0x80000000, 0x7fffffff},
     {0xc0000000, 0x7f000000, 0xbfffffff, 0x01000000, 0x80000000, 0x7fffffff},
     {0x00000000, 0x7f000000, 0x80000000, 0x00ffffff, 0x80000000, 0x7fffffff},
     {0x00000001, 0x7f000000, 0x80000000, 0x00fffffe, 0x80000001, 0x7ffffffe},
     {0x00000001, 0x7f000000, 0x80000000, 0x01000000, 0x80000000, 0x7fffffff},
     {0x01000000, 0x01000000, 0x7f000000, 0x7f000000, 0x80000000, 0x80000000},
     {0x7fffffff, 0x7fffffff, 0x7fffffff, 0x7fffffff, 0xfffffffe, 0xfffffffe},
   };

   int s, t;

   for (int i = 0; i < (int)(sizeof(u)/sizeof(u[0])); i++) {
      boundsAdds(u[i].a, u[i].b, u[i].c, u[i].d, &s, &t);
      if (s != u[i].e) {
         printf("Error in computing minsum, i = %d\n", i);
         exit(1);
      }
      if (t != u[i].f) {
         printf("Error in computing maxsum, i = %d\n", i);
         exit(2);
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
