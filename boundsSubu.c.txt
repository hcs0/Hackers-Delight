/* Given a, b, c, and d, this program computes the min and max values
of x - y, where a <= x <= b and c <= y <= d (unsigned numbers).
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

void boundsSubu(unsigned a, unsigned b, unsigned c,
                unsigned d, unsigned *ps, unsigned *pt) {

   unsigned s, t;

// ------------------------------ cut ----------------------------------
   s = a - d;
   t = b - c;
   if (s > a && t <= b) {
      s = 0;
      t = 0xFFFFFFFF;}
// ------------------------------ cut ----------------------------------
   *ps = s;
   *pt = t;
   return;
}

// ------------------------------ main ---------------------------------

int main() {
   static struct {unsigned a, b, c, d, e, f;} u[] =
   //      a           b           c           d           e          f
   { {0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000, 0x00000000},
     {0x00000004, 0x00000006, 0x00000002, 0x00000003, 0x00000001, 0x00000004},
     {0x00000002, 0x00000003, 0x00000004, 0x00000006, 0xfffffffc, 0xffffffff},
     {0x00000002, 0x00000003, 0x00000000, 0x00000000, 0x00000002, 0x00000003},
     {0x00000002, 0x00000003, 0x00000000, 0xffffffff, 0x00000000, 0xffffffff},
     {0x00000004, 0x00000006, 0x00000003, 0x00000005, 0x00000000, 0xffffffff},
     {0x80000000, 0xa0000000, 0x90000000, 0x90000005, 0x00000000, 0xffffffff},
     {0x80000000, 0x80000000, 0x80000000, 0x80000000, 0x00000000, 0x00000000},
     {0x80000000, 0x80000001, 0x00000004, 0x00000006, 0x7ffffffa, 0x7ffffffd},
     {0x00000000, 0x00000000, 0xffffffff, 0xffffffff, 0x00000001, 0x00000001},
     {0x00000000, 0x00000001, 0xffffffff, 0xffffffff, 0x00000001, 0x00000002},
     {0xffffffff, 0xffffffff, 0xffffffff, 0xffffffff, 0x00000000, 0x00000000},
   };

   unsigned s, t;

   for (int i = 0; i < (int)(sizeof(u)/sizeof(u[0])); i++) {
      boundsSubu(u[i].a, u[i].b, u[i].c, u[i].d, &s, &t);
      if (s != u[i].e) {
         printf("Error in computing mindiff, i = %d\n", i);
         exit(1);
      }
      if (t != u[i].f) {
         printf("Error in computing maxdiff, i = %d\n", i);
         exit(2);
      }
   }
   printf("Passed all tests.\n");
   return 0;
}
