/* Tests when signed long division will overflow. The data types used
(long long) are oriented toward a 64-bit machine. */

#include <stdio.h>

#define llabs(x) ({unsigned long long t = (x) >> 63; ((x) ^ t) - t;})

int oflo1(long long u, long long v) {

   unsigned long long au, av, del;

   if (((v << 32) >> 32) != v) return 0;
   au = llabs(u);
   av = llabs(v);
   del = ((u ^ v) >> 63) & av;
   if (au < (av << 31) + del) return 1;
   return 0;
}

int oflo2(long long u, long long v) {

   long long vs;

   if (((v << 32) >> 32) != v) return 0;
   vs = v << 31;
   if (u >= 0)
      if (v > 0) {
         if (u < vs) goto easy_case;            // +/+.
      }
      else {
         if (u + vs + v < 0) goto easy_case;    // +/-.
      }
   else
      if (v > 0) {
         if (u + vs + v > 0) goto easy_case;    // -/+.
      }
      else {
         if (u > vs) goto easy_case;            // -/-.
      }
   return 0;

easy_case:
   return 1;
}

int main() {
   long long tab[] = {0, 1, 2, 3, 0x7ffffffe, 0x7fffffff,
      0x80000000, 0x80000001, 0x80000002, 0x80000003,
      0xfffffffe, 0xffffffff,
      0x100000000, 0x100000001, 0x100000002, 0x100000003,
      0x200000000, 0x200000001, 0x200000002, 0x200000003,
      0xfffffffe00000000, 0xfffffffe00000001, 0xfffffffe00000003,
      0xffffffff00000000, 0xffffffff00000001, 0xffffffff00000003,
      0xfffffffffffffffd, 0xfffffffffffffffe, 0xffffffffffffffff,
   };

   long long u, v, q;
   int n, i, j, k, ok, y, errs = 0;

   n = sizeof(tab)/sizeof(tab[0]);
   for (i = 0; i < n; i++) {
      u = tab[i];
      for (j = 1; j < n; j++) {
         v = tab[j];
         for (k = 0; k <= 3; k++) {     // Do all combinations of +, -.
            if (k & 1)  u = -u;
            if (k >= 2) v = -v;
            ok = 1;
            q = 0xdeadbeefdeadbeef;     // Initialize q.
            if (v << 32 >> 32 != v) ok = 0;
            else {
               q = u/v;
               if (q > (long long)0x7FFFFFFF
               || q < (long long)0xFFFFFFFF80000000) ok = 0;
            }

            /* Now we have the correct value of ok (means ok to
            use the machine's "long division" routine. Try the
            method under test to see if it gets the same result. */

            y = oflo1(u, v);
            if (y != ok) {
               printf("oflo1 failed for %016llx / %016llx = %016llx, got %d (should be %d).\n",
                  u, v, q, y, ok);
               errs = errs + 1;
            }

            y = oflo2(u, v);
            if (y != ok) {
               printf("oflo2 failed for %016llx / %016llx = %016llx, got %d (should be %d).\n",
                  u, v, q, y, ok);
               errs = errs + 1;
            }
         } // for k
      } // for j
   } // for i
   if (errs == 0) printf("All functions passed all %d tests.\n", n*(n-1)*4);
   else printf("Failed %d cases.\n", errs);
   return errs;
}
