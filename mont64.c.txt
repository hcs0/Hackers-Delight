/* Computes a*b mod m using Montgomery multiplication (MM). a, b, and m
are unsigned numbers with a, b < m < 2**64, and m odd. The code does
some 128-bit arithmetic.
   The variable r is fixed at 2**64, and its log base 2 at 64.
   Works with gcc on Windows and Linux. */

#include <stdlib.h>
#include <stdio.h>

typedef unsigned long long uint64;
typedef long long int64;

/* ---------------------------- mulul64 ----------------------------- */

/* Multiply unsigned long 64-bit routine, i.e., 64 * 64 ==> 128.
Parameters u and v are multiplied and the 128-bit product is placed in
(*whi, *wlo). It is Knuth's Algorithm M from [Knu2] section 4.3.1.
Derived from muldwu.c in the Hacker's Delight collection. */

void mulul64(uint64 u, uint64 v, uint64 *whi, uint64 *wlo)
   {
   uint64 u0, u1, v0, v1, k, t;
   uint64 w0, w1, w2;

   u1 = u >> 32; u0 = u & 0xFFFFFFFF;
   v1 = v >> 32; v0 = v & 0xFFFFFFFF;

   t = u0*v0;
   w0 = t & 0xFFFFFFFF;
   k = t >> 32;

   t = u1*v0 + k;
   w1 = t & 0xFFFFFFFF;
   w2 = t >> 32;

   t = u0*v1 + w1;
   k = t >> 32;

   *wlo = (t << 32) + w0;
   *whi = u1*v1 + w2 + k;

   return;
}

/* ---------------------------- modul64 ----------------------------- */

uint64 modul64(uint64 x, uint64 y, uint64 z) {

   /* Divides (x || y) by z, for 64-bit integers x, y,
   and z, giving the remainder (modulus) as the result.
   Must have x < z (to get a 64-bit result). This is
   checked for. */

   int64 i, t;

   printf("In modul64, x = %016llx, y = %016llx, z = %016llx\n", x, y, z);
   if (x >= z) {
      printf("Bad call to modul64, must have x < z.");
      exit(1);
   }
   for (i = 1; i <= 64; i++) {  // Do 64 times.
      t = (int64)x >> 63;       // All 1's if x(63) = 1.
      x = (x << 1) | (y >> 63); // Shift x || y left
      y = y << 1;               // one bit.
      if ((x | t) >= z) {
         x = x - z;
         y = y + 1;
      }
   }
   return x;                    // Quotient is y.
}

/* ---------------------------- montmul ----------------------------- */

uint64 montmul(uint64 abar, uint64 bbar, uint64 m,
               uint64 mprime) {

   uint64 thi, tlo, tm, tmmhi, tmmlo, uhi, ulo, ov;

   printf("\nmontmul, abar = %016llx, bbar   = %016llx\n", abar, bbar);
   printf("            m = %016llx, mprime = %016llx\n", m, mprime);

   /* t = abar*bbar. */

   mulul64(abar, bbar, &thi, &tlo);  // t = abar*bbar.
   printf("montmul, thi = %016llx, tlo = %016llx\n", thi, tlo);

   /* Now compute u = (t + ((t*mprime) & mask)*m) >> 64.
   The mask is fixed at 2**64-1. Because it is a 64-bit
   quantity, it suffices to compute the low-order 64
   bits of t*mprime, which means we can ignore thi. */

   tm = tlo*mprime;
   printf("montmul, tm = %016llx\n", tm);

   mulul64(tm, m, &tmmhi, &tmmlo);   // tmm = tm*m.
   printf("montmul, tmmhi = %016llx, tmmlo = %016llx\n", tmmhi, tmmlo);

   ulo = tlo + tmmlo;                // Add t to tmm
   uhi = thi + tmmhi;                // (128-bit add).
   if (ulo < tlo) uhi = uhi + 1;     // Allow for a carry.

   // The above addition can overflow. Detect that here.

   ov = (uhi < thi) | ((uhi == thi) & (ulo < tlo));
   printf("montmul, sum, uhi = %016llx, ulo = %016llx, ov = %lld\n", uhi, ulo, ov);

   ulo = uhi;                   // Shift u right
   uhi = 0;                     // 64 bit positions.
   printf("montmul, ulo = %016llx\n", ulo);

// if (ov > 0 || ulo >= m)      // If u >= m,
//    ulo = ulo - m;            // subtract m from u.
   ulo = ulo - (m & -(ov | (ulo >= m))); // Alternative
                                // with no branching.
   printf("montmul, final ulo = %016llx\n", ulo);

   if (ulo >= m)
      printf("ERROR in montmul, ulo = %016llx, m = %016llx\n", ulo, m);
   return ulo;
}

/* ---------------------------- xbinGCD ----------------------------- */

/* C program implementing the extended binary GCD algorithm. C.f.
http://www.ucl.ac.uk/~ucahcjm/combopt/ext_gcd_python_programs.pdf. This
is a modification of that routine in that we find s and t s.t.
    gcd(a, b) = s*a - t*b,
rather than the same expression except with a + sign.
   This routine has been greatly simplified to take advantage of the
facts that in the MM use, argument a is a power of 2, and b is odd. Thus
there are no common powers of 2 to eliminate in the beginning. The
parent routine has two loops. The first drives down argument a until it
is 1, modifying u and v in the process. The second loop modifies s and
t, but because a = 1 on entry to the second loop, it can be easily seen
that the second loop doesn't alter u or v. Hence the result we want is u
and v from the end of the first loop, and we can delete the second loop.
   The intermediate and final results are always > 0, so there is no
trouble with negative quantities. Must have a either 0 or a power of 2
<= 2**63. A value of 0 for a is treated as 2**64. b can be any 64-bit
value.
   Parameter a is half what it "should" be. In other words, this function
does not find u and v st. u*a - v*b = 1, but rather u*(2a) - v*b = 1. */

void xbinGCD(uint64 a, uint64 b, uint64 *pu, uint64 *pv)
   {
   uint64 alpha, beta, u, v;
   printf("Doing GCD(%llx, %llx)\n", a, b);

   u = 1; v = 0;
   alpha = a; beta = b;         // Note that alpha is
                                // even and beta is odd.

   /* The invariant maintained from here on is:
   a = u*2*alpha - v*beta. */

// printf("Before, a u v = %016llx %016llx %016llx\n", a, u, v);
   while (a > 0) {
      a = a >> 1;
      if ((u & 1) == 0) {             // Delete a common
         u = u >> 1; v = v >> 1;      // factor of 2 in
      }                               // u and v.
      else {
         /* We want to set u = (u + beta) >> 1, but
         that can overflow, so we use Dietz's method. */
         u = ((u ^ beta) >> 1) + (u & beta);
         v = (v >> 1) + alpha;
      }
//    printf("After,  a u v = %016llx %016llx %016llx\n", a, u, v);
   }

// printf("At end,    a u v = %016llx %016llx %016llx\n", a, u, v);
   *pu = u;
   *pv = v;
   return;
}

/* ------------------------------ main ------------------------------ */

int main(int argc, char* argv[]) {
   char *q;
   uint64 a, b, m, hr, rinv, mprime, p1hi, p1lo, p1, p, abar, bbar;
   uint64 phi, plo;

   if (argc != 4) {
      printf("Need exactly three arguments, decimal numbers.");
      return 1;
   }

   a = strtoull(argv[1], &q, 0);
   if (*q != 0 || a == 0xFFFFFFFFFFFFFFFFLL) {
      printf("Invalid first argument.");
      return 1;
   }

   b = strtoull(argv[2], &q, 0);
   if (*q != 0 || b == 0xFFFFFFFFFFFFFFFFLL) {
      printf("Invalid second argument.");
      return 1;
   }

   m = strtoull(argv[3], &q, 0);         // The modulus, we are computing a*b (mod m).
   if (*q != 0) {
      printf("Invalid third argument.");
      return 1;
   }

   if ((m & 1) == 0) {
      printf("The modulus (third argument) must be odd.");
      return 1;
   }

   if (a >= m || b >= m) {
      printf("The first two args must be less than the modulus (third argument).");
      return 1;
   }

   printf("a, b, m = %016llx %016llx %016llx\n", a, b, m);

   /* The simple calculation: This computes (a*b)**4 (mod m) correctly for all a,
   b, m < 2**64. */

   mulul64(a, b, &p1hi, &p1lo);         // Compute a*b (mod m).
   p1 = modul64(p1hi, p1lo, m);
   mulul64(p1, p1, &p1hi, &p1lo);       // Compute (a*b)**2 (mod m).
   p1 = modul64(p1hi, p1lo, m);
   mulul64(p1, p1, &p1hi, &p1lo);       // Compute (a*b)**4 (mod m).
   p1 = modul64(p1hi, p1lo, m);
   printf("p1 = %016llx\n", p1);

   /* The MM method uses a quantity r that is the smallest power of 2
   that is larger than m, and hence also larger than a and b. Here we
   deal with a variable hr that is just half of r. This is because r can
   be as large as 2**64, which doesn't fit in one 64-bit word. So we
   deal with hr, where 2**63 <= hr <= 1, and make the appropriate
   adjustments wherever it is used.
      We fix r at 2**64, and its log base 2 at 64. It doesn't hurt if
   they are too big, it's just that some quantities (e.g., mprime) come
   out larger than they would otherwise be. */

   hr = 0x8000000000000000LL;

   /* Now, for the MM method, first compute the quantities that are
   functions of only r and m, and hence are relatively constant. These
   quantities can be used repeatedly, without change, when raising a
   number to a large power modulo m.
      First use the extended GCD algorithm to compute two numbers rinv
   and mprime, such that

                         r*rinv - m*mprime = 1

   Reading this nodulo m, clearly r*rinv = 1 (mod m), i.e., rinv is the
   multiplicative inverse of r modulo m. It is needed to convert the
   result of MM back to a normal number. The other calculated number,
   mprime, is used in the MM algorithm. */

   xbinGCD(hr, m, &rinv, &mprime);      // xbinGCD, in effect, doubles hr.

   /* Do a partial check of the results. It is partial because the
   multiplications here give only the low-order half (64 bits) of the
   products. */

   printf("rinv = %016llx, mprime = %016llx\n", rinv, mprime);
   if (2*hr*rinv - m*mprime != 1) {
      printf("The Extended Euclidean algorithm failed.");
      return 1;
   }

   /* Compute abar = a*r(mod m) and bbar = b*r(mod m). That is, abar =
   (a << 64)%m, and bbar = (b << 64)%m. */

   abar = modul64(a, 0, m);
   bbar = modul64(b, 0, m);

   p = montmul(abar, bbar, m, mprime); /* Compute a*b (mod m). */
   p = montmul(p, p, m, mprime);       /* Compute (a*b)**2 (mod m). */
   p = montmul(p, p, m, mprime);       /* Compute (a*b)**4 (mod m). */
   printf("p before converting back = %016llx\n", p);

   /* Convert p back to a normal number by p = (p*rinv)%m. */

   mulul64(p, rinv, &phi, &plo);
   p = modul64(phi, plo, m);
   printf("p = %016llx\n", p);
   if (p != p1) printf("ERROR, p != p1.\n");
   else         printf("Correct (p = p1).\n");

   return 0;
}

