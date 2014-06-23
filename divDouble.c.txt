/* This file contains a program for doing 64/64 ==> 64 division, on a
machine that does not have that instruction but that does have
instructions for "long division" (64/32 ==> 32). Code for unsigned
division is given first, followed by a simple program for doing the
signed version by using the unsigned version.
   These programs are useful in implementing "long long" (64-bit)
arithmetic on a machine that has the long division instruction. It will
work on 64- and 32-bit machines, provided the compiler implements long
long's (64-bit integers). It is desirable that the machine have the
Count Leading Zeros instruction.
   In the GNU world, these programs are known as __divdi3 and __udivdi3,
and similar names are used here.
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

/* ----------------------------- nlz64 ------------------------------ */

int nlz64(unsigned long long x) {
   int n;

   if (x == 0) return(64);
   n = 0;
   if (x <= 0x00000000FFFFFFFF) {n = n + 32; x = x << 32;}
   if (x <= 0x0000FFFFFFFFFFFF) {n = n + 16; x = x << 16;}
   if (x <= 0x00FFFFFFFFFFFFFF) {n = n +  8; x = x <<  8;}
   if (x <= 0x0FFFFFFFFFFFFFFF) {n = n +  4; x = x <<  4;}
   if (x <= 0x3FFFFFFFFFFFFFFF) {n = n +  2; x = x <<  2;}
   if (x <= 0x7FFFFFFFFFFFFFFF) {n = n +  1;}
   return n;
}

// ---------------------------- udivdi3 --------------------------------

   /* The variables u0, u1, etc. take on only 32-bit values, but they
   are declared long long to avoid some compiler warning messages and to
   avoid some unnecessary EXTRs that the compiler would put in, to
   convert long longs to ints.

   First the procedure takes care of the case in which the divisor is a
   32-bit quantity. There are two subcases: (1) If the left half of the
   dividend is less than the divisor, one execution of DIVU is all that
   is required (overflow is not possible). (2) Otherwise it does two
   divisions, using the grade school method, with variables used as
   suggested below.

       q1 q0
    ________
   v)  u1 u0
     q1*v
     ____
        k u0   */

/* These macros must be used with arguments of the appropriate type
(unsigned long long for DIVU and long long for DIVS. They are
simulations of the presumed machines ops. I.e., they look at only the
low-order 32 bits of the divisor, they return garbage if the division
overflows, and they return garbage in the high-order half of the
quotient doubleword.
   In practice, these would be replaced with uses of the machine's DIVU
and DIVS instructions (e.g., by using the GNU "asm" facility). */

#define DIVU(u, v) ({unsigned long long __v = (v) & 0xFFFFFFFF; \
   long long __q = (u)/(v); \
   __q > 0xFFFFFFFF ? 0xdeaddeadbeefbeef : \
   __q | 0xdeadbeef00000000;})
#define DIVS(u, v) ({long long __v = (v) << 32 >> 32; \
   long long __q = (u)/(__v); \
   __q < (long long)0xFFFFFFFF80000000ll || \
   __q > (long long)0x000000007FFFFFFFll ? \
   0xfadefadedeafdeaf : __q | 0xfadedeaf00000000;})

// ------------------------------ cut ----------------------------------

unsigned long long udivdi3(unsigned long long u,
                           unsigned long long v) {

   unsigned long long u0, u1, v1, q0, q1, k, n;

   if (v >> 32 == 0) {          // If v < 2**32:
      if (u >> 32 < v)          // If u/v cannot overflow,
         return DIVU(u, v)      // just do one division.
            & 0xFFFFFFFF;
      else {                    // If u/v would overflow:
         u1 = u >> 32;          // Break u up into two
         u0 = u & 0xFFFFFFFF;   // halves.
         q1 = DIVU(u1, v)       // First quotient digit.
            & 0xFFFFFFFF;
         k = u1 - q1*v;         // First remainder, < v.
         q0 = DIVU((k << 32) + u0, v) // 2nd quot. digit.
            & 0xFFFFFFFF;
         return (q1 << 32) + q0;
      }
   }
                                // Here v >= 2**32.
   n = nlz64(v);                // 0 <= n <= 31.
   v1 = (v << n) >> 32;         // Normalize the divisor
                                // so its MSB is 1.
   u1 = u >> 1;                 // To ensure no overflow.
   q1 = DIVU(u1, v1)            // Get quotient from
       & 0xFFFFFFFF;            // divide unsigned insn.
   q0 = (q1 << n) >> 31;        // Undo normalization and
                                // division of u by 2.
   if (q0 != 0)                 // Make q0 correct or
      q0 = q0 - 1;              // too small by 1.
   if ((u - q0*v) >= v)
      q0 = q0 + 1;              // Now q0 is correct.
   return q0;
}

// ---------------------------- end cut --------------------------------

/* The next ca. 140 lines is a proof that the udivdi3
code is correct for the case in which v >= 2**32 (the
hard case).

We'll need these basics (integer variables):

   floor(floor(a/b)/d) = floor(a/(bd)).               (1)
   b*floor(a/b) = a - rem(a, b).                      (2)

From the first line of this section of the procedure,

0 <= n <= 31.

In computing v1, the left shift clearly cannot overflow.
Therefore

v1 = floor(v/2**(32-n)).

u1 = floor(u/2).

In computing q1, u1 and v1 are in range for the DIVU
instruction and it cannot overflow. Hence

q1 = floor(u1/v1).

In the first computation of q0, the left shift cannot
overflow because q1 < 2**32 (because the maximum value of
u1 is 2**63 - 1 and the minimum value of v1 is 2**31).
Therefore

q0 = floor(q1/2**(31-n)).

Now, for the main part of the proof, we want to show that

           floor(u/v) <= q0 <= floor(u/v) + 1,

which is to say, the first computation of q0 is the
desired result or is that plus 1.

Using (1) twice gives

     q0 = floor(u/((2**(32-n))v1))

        = floor(u/((2**(32-n))floor(v/2**(32-n)))).

Using (2) gives

     q0 = floor(u/(v - rem(v, 2**(32-n))).

Using algebra to get this in the form u/v + something:

               (u     u(rem(v, 2**(32-n)))  )
     q0 = floor(- + ------------------------).
               (v   v(v - rem(v, 2**(32-n))))

This is of the form

                  floor(u/v + delta),

and we will now show that delta < 1.

delta is largest when rem(v, 2**(32-n)) is as large as
possible and, given that, when v is as small as possible.
The maximum value of rem(v, 2**(32-n)) is 2**(32-n) - 1.
Because v >= 2**(63-n), the smallest value of v having
that remainder is

               2**(63-n) + 2**(32-n) - 1.

Thus

                       u(2**(32-n) - 1)
     delta <= -------------------------------------
              (2**(63-n) + 2**(32-n) - 1)*2**(63-n)

              u(2**(32-n) - 1)
           <  ----------------
               (2**(63-n))**2

By inspection, for n in its range of 0 to 31,

                    delta < u/2**64.

Since u is at most 2**64 - 1, delta < 1. Because
q0 = floor(u/v + delta) and delta < 1 (and obviously
delta >= 0),

           floor(u/v) <= q0 <= floor(u/v) + 1.

To correct this by subtracting 1 when necessary, we would
like to code

   if (u < q0*v) q0 = q0 - 1;

(i.e., if the remainder u - q0*v is negative, subtract 1
from q0). However, this doesn't quite work because q0*v
can overflow (e.g., for u = 2**64 - 1 and v = 2**32 + 3).
Instead, we subtract 1 from q0, so that it is either
correct or too SMALL by 1. Then, q0*v will not overflow.
We must avoid subtracting 1 if q0 = 0 (if q0 = 0, it is
already the correct quotient).

Then the final correction is:

   if ((u - q0*v) >= v) q0 = q0 + 1;

To see that this is a valid computation, we already noted
that q0*v does not overflow. It is easy to show that

   0 <= u - q0*v < 2*v.

If v is very large (>= 2**63), can the subtraction
overflow? No, because u < 2**64 and q0*v >= 0.

Incidentally, there are alternatives to the lines

   if (q0 != 0)                 // Make q0 correct or
      q0 = q0 - 1;              // too small by 1.

that may be preferable on some machines. One is to
replace them with

   if (q0 == 0) return 0;

Another is to place at the beginning of this section of
the procedure, or in fact at the beginning of the whole
procedure, the line

   if (u < v) return 0;         // Avoid a problem later.

These alternatives are preferable if branches are not
costly. The way it is coded above is good if the
machine's comparison instructions produce a 0/1 integer
result. Then, the compiler can change it to, in effect,

   q0 = q0 - (q0 != 0);

(or you can code it that way if your compiler doesn't do
this optimization). This is just a compare and subtract
on such machines. */

// ------------------------- End Gory Proof ----------------------------

// ----------------------------- divdi3 --------------------------------

/* This routine presumes that smallish cases (those which can be done in
one execution of DIVS) are common. If this is not the case, the test for
this case should be deleted.
   Note that the test for when DIVS can be used is not entirely
accurate. For example, DIVS is not used if v = 0xFFFFFFFF8000000,
whereas if could be (if u is sufficiently small in magnitude). */

// ------------------------------ cut ----------------------------------

#define llabs(x) \
({unsigned long long t = (x) >> 63; ((x) ^ t) - t;})

long long divdi3(long long u, long long v) {

   unsigned long long au, av;
   long long q, t;

   au = llabs(u);
   av = llabs(v);
   if (av >> 31 == 0) {         // If |v| < 2**31 and
// if (v << 32 >> 32 == v) {    // If v is in range and
      if (au < av << 31) {      // |u|/|v| cannot
         q = DIVS(u, v);        // overflow, use DIVS.
         return (q << 32) >> 32;
      }
   }
   q = au/av;                   // Invoke udivdi3.
   t = (u ^ v) >> 63;           // If u, v have different
   return (q ^ t) - t;          // signs, negate q.
}

// ---------------------------- end cut --------------------------------

// ------------------------------ main ---------------------------------

int main()
{
   /* First test unsigned division. */

   /* The entries in this table are used in all combinations. */

   static unsigned long long tabu[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
      10, 11, 12, 13, 14, 15, 16, 1000, 2003,
      32765, 32766, 32767, 32768, 32769, 32760,
      65533, 65534, 65535, 65536, 65537, 65538,
      0x7ffffffe, 0x7fffffff, 0x80000000, 0x80000001,
      0x7000000000000000, 0x7000000080000000, 0x7000000080000001,
      0x7fffffffffffffff, 0x7fffffff8fffffff, 0x7fffffff8ffffff1,
      0x7fffffff00000000, 0x7fffffff80000000, 0x7fffffff00000001,
      0x8000000000000000, 0x8000000080000000, 0x8000000080000001,
      0xc000000000000000, 0xc000000080000000, 0xc000000080000001,
      0xfffffffffffffffd, 0xfffffffffffffffe, 0xffffffffffffffff,
      };

   /* The entries in this table are used in all combinations, with
   + and - signs preceding them. */

   static long long tabs[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
      10, 11, 12, 13, 14, 15, 16, 1000, 2003,
      32765, 32766, 32767, 32768, 32769, 32760,
      65533, 65534, 65535, 65536, 65537, 65538,
      0x7ffffffe, 0x7fffffff, 0x80000000, 0x80000001,
      0x7000000000000000, 0x7000000080000000, 0x7000000080000001,
      0x7fffffffffffffff, 0x7fffffff8fffffff, 0x7fffffff8ffffff1,
      0x7fffffff00000000, 0x7fffffff80000000, 0x7fffffff00000001,
      0x8000000000000000, 0x8000000080000000, 0x8000000080000001,
      0x0123456789abcdef, 0x00000000abcdef01, 0x0000000012345678,
      };

   int n, i, j, k, errs = 0;
   unsigned long long uu, vu, qu, ru;
   long long u, v, q, r;

   printf("Testing unsigned division.\n");
   n = sizeof(tabu)/sizeof(tabu[0]);
   for (i = 0; i < n; i++) {
      for (j = 1; j < n; j++) {         // Skip tabu[0], which is 0.
         uu = tabu[i];
         vu = tabu[j];
         qu = udivdi3(uu, vu);          // Call the program being tested.
         ru = uu - qu*vu;
         if (qu > uu || ru >= vu) {
            printf("Error for %016llx/%016llx, got %016llx rem %016llx\n",
               uu, vu, qu, ru);
            errs = errs + 1;
         }
      }
   }
   if (errs == 0) printf("Passed all %d tests (unsigned).\n", n*(n-1));
   else printf("Failed %d cases (unsigned).\n", errs);
   if (errs != 0) return errs;

   /* Now test signed division. */

   printf("Testing signed division.\n");
   n = sizeof(tabs)/sizeof(tabs[0]);
   for (i = 0; i < n; i++) {
      for (j = 1; j < n; j++) {         // Skip tabs[0], which is 0.
         u = tabs[i];
         v = tabs[j];
         for (k = 0; k <= 3; k++) {     // Do all combinations of +, -.
            if (k & 1)  u = -u;
            if (k >= 2) v = -v;
            q = divdi3(u, v);           // Call the program being tested.
            r = u - q*v;
            if (llabs(q) > llabs(u) ||
               llabs(r) >= llabs(v) ||
               (r != 0 && (r ^ u) < 0)) {
               printf("Error for %016llx/%016llx, got %016llx rem %016llx\n",
                  u, v, q, r);
               errs = errs + 1;
            }
         }
      }
   }
   if (errs == 0) printf("Passed all %d tests (signed).\n", n*(n-1)*4);
   else printf("Failed %d cases (signed).\n", errs);
   return errs;
}
