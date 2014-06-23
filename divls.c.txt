/* Long division, signed (64/32 ==> 32).
   A procedure below performs signed long division (64/32 ==> 32) using
a procedure for unsigned long division.  In the overflow cases (divide
by 0, or quotient exceeds 32 bits), it returns a remainder of 0x80000000
(an impossible value), and for good measure a quotient of 0x80000000.
   The dividend is u1 and u0, with u1 being the most significant word.
The divisor is parameter v. The value returned is the quotient.
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

#define max(x, y) ((x) > (y) ? (x) : (y))

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

unsigned myrand(void) {         // Expands rand() to 32 bits.
   return rand() ^ (rand() << 1);
}

/* Here is the unsigned long division routine.  It is divlu2 in
divlu.cc. */

unsigned divlu(unsigned u1, unsigned u0, unsigned v,
                unsigned *r) {
   const unsigned b = 65536; // Number base (16 bits).
   unsigned un1, un0,        // Norm. dividend LSD's.
            vn1, vn0,        // Norm. divisor digits.
            q1, q0,          // Quotient digits.
            un32, un21, un10,// Dividend digit pairs.
            rhat;            // A remainder.
   int s;                    // Shift amount for norm.

   if (u1 >= v) {            // If overflow, set rem.
      if (r != NULL)         // to an impossible value,
         *r = 0xFFFFFFFF;    // and return the largest
      return 0xFFFFFFFF;}    // possible quotient.

   s = nlz(v);               // 0 <= s <= 31.
   v = v << s;               // Normalize divisor.
   vn1 = v >> 16;            // Break divisor up into
   vn0 = v & 0xFFFF;         // two 16-bit digits.

   un32 = (u1 << s) | (u0 >> 32 - s) & (-s >> 31);
   un10 = u0 << s;           // Shift dividend left.

   un1 = un10 >> 16;         // Break right half of
   un0 = un10 & 0xFFFF;      // dividend into two digits.

   q1 = un32/vn1;            // Compute the first
   rhat = un32 - q1*vn1;     // quotient digit, q1.
again1:
   if (q1 >= b || q1*vn0 > b*rhat + un1) {
     q1 = q1 - 1;
     rhat = rhat + vn1;
     if (rhat < b) goto again1;}

   un21 = un32*b + un1 - q1*v;  // Multiply and subtract.

   q0 = un21/vn1;            // Compute the second
   rhat = un21 - q0*vn1;     // quotient digit, q0.
again2:
   if (q0 >= b || q0*vn0 > b*rhat + un0) {
     q0 = q0 - 1;
     rhat = rhat + vn1;
     if (rhat < b) goto again2;}

   if (r != NULL)            // If remainder is wanted,
      *r = (un21*b + un0 - q0*v) >> s;     // return it.
   return q1*b + q0;
}

int errors;

void check(char *name, int q, int r, int u1, unsigned u0, int v,
           int cq, int cr) {

   if (q != cq || r != cr) {
      errors = errors + 1;
      printf("Error in %s, dividend u = %08x %08x\r\n", name, u1, u0);
      printf("                divisor  v = %08x\r\n", v);
      printf("Got       q = %08x,  r = %08x\r\n", q, r);
      printf("Should get cq = %08x, cr = %08x\r\n", cq, cr);
   }
   return;
}

/* Here is the signed long division routine, for inclusion in
the Hacker book. */

// ------------------------------ cut ----------------------------------
int divls(int u1, unsigned u0, int v, int *r) {
   int q, uneg, vneg, diff, borrow;

   uneg = u1 >> 31;          // -1 if u < 0.
   if (uneg) {               // Compute the absolute
      u0 = -u0;              // value of the dividend u.
      borrow = (u0 != 0);
      u1 = -u1 - borrow;}

   vneg = v >> 31;           // -1 if v < 0.
   v = (v ^ vneg) - vneg;    // Absolute value of v.

   if ((unsigned)u1 >= (unsigned)v) goto overflow;

   q = divlu(u1, u0, v, (unsigned *)r);

   diff = uneg ^ vneg;       // Negate q if signs of
   q = (q ^ diff) - diff;    // u and v differed.
   if (uneg && r != NULL)
      *r = -*r;

   if ((diff ^ q) < 0 && q != 0) {  // If overflow,
overflow:                    // set remainder
      if (r != NULL)         // to an impossible value,
         *r = 0x80000000;    // and return the largest
      q = 0x80000000;}       // possible neg. quotient.
   return q;
}
// ------------------------------ cut ----------------------------------

int main() {
   static unsigned test[] = {
   // u...,                  v...,
               0,         0,          0,        // Overflow.
               1,         0,          0,        // Overflow.
               1,         0,          1,        // Overflow.
               0,0x80000000,          1,        // Overflow.
      0xffffffff,0x7fffffff,          1,        // Overflow.
               0,0x80000001, 0xffffffff,        // Overflow.
      0xffffffff,0x80000000, 0xffffffff,        // Overflow.
      0x3fffffff,0x80000000, 0x7fffffff,        // Overflow.
      0xc0000000,         0, 0x80000000,        // Overflow.
               0,0x7fffffff,          1,        // Just barely not overflow.
      0xffffffff,0x80000000,          1,        // Just barely not overflow.
               0,0x80000000, 0xffffffff,        // Just barely not overflow.
      0xffffffff,0x80000001, 0xffffffff,        // Just barely not overflow.
      0x3fffffff,0x7fffffff, 0x7fffffff,        // Just barely not overflow.
      0xc0000000,         1, 0x80000000,        // Just barely not overflow.
               0,         1,          1,
               0,         7,          3,
      0x00000000,0x7fffffff, 0x00000001,
      0x01234567,0x89abcdef, 0x02468ad0,
      0x01234567,0x89abcdef, 0x12345678,
      0x01234567,0x89abcdef, 0x80000000,
      0x01234567,0x89abcdef, 0x80000001,
      0x01234567,0x89abcdef, 0x80007fff,
      0x01234567,0x89abcdef, 0x80008000,
      0x01234567,0x89abcdef, 0x80008001,
      0x01234567,0x89abcdef, 0x8000ffff,

      0x00000000,0x00000000, 0x0000ffff,
      0x00000000,0x0000ffff, 0x0000ffff,
      0x00000000,0xffff0000, 0x0000ffff,
      0x00000000,0xffffffff, 0x0000ffff,
      0x0000fffe,0x00000000, 0x0001ffff,
      0x0000fffe,0x0000ffff, 0x0001ffff,
      0x0000fffe,0xffff0000, 0x0001fffe,
      0x0000fffe,0xffffffff, 0x0001fffe,

      0x00000000,0x00000000, 0xffff0000,
      0x00000000,0x0000ffff, 0xffff0000,
      0x00000000,0xffff0000, 0xffff0000,
      0x00000000,0xffffffff, 0xffff0000,
      0x00007fff,0x00000000, 0xffff0000,
      0x00007fff,0x0000ffff, 0xffff0000,
      0x00007fff,0xffff0000, 0xffff0000,
      0x00007fff,0xffffffff, 0xffff0000,

      0xffff8000,0xffff0000, 0xffff0000,
      0xffff8000,0x0000ffff, 0xffff0000,
      0xffff8000,0xffffffff, 0xffff0000,
      0xffff8fff,0x00000000, 0xffff0000,
      0xffff8fff,0x0000ffff, 0xffff0000,
      0xffff8fff,0xffff0000, 0xffff0000,
      0xffff8fff,0xffffffff, 0xffff0000,

      0x00000000,0x00000000, 0xffffffff,
      0x00000000,0x0000ffff, 0xffffffff,
      0x00000000,0x7fff0000, 0xffffffff,
      0x00000000,0x7fffffff, 0xffffffff,
      0x00000000,0x80000000, 0xffffffff,

      0x12345678,0x9abcdef0, 0x2468ad00,
      0x00010000,0x00000000, 0x00020001,
      0xc0008000,0x00000000, 0x80008001,        // First qhat = b + 1.
      0xc000fffe,0xffffffff, 0x8000ffff,        // First qhat = b + 1.
      0x3fff8000,0x00000000, 0x8000ffff,        // First qhat too big by 2.
   };

   int i, n;
   int u1, v, q, r, cq, cr;
   unsigned u0;

   n = sizeof(test)/12 + 1000000;

   for (i = 0; i < n; i++) {
      if (i < sizeof(test)/12) {
         u1 = (int)test[3*i];   u0 = test[3*i+1];  v = (int)test[3*i+2];
      }
      else {
         u1 = (int)myrand();   u0 = myrand();  v = (int)myrand();
         if (v == 0) v = 1;
         if (abs(u1) >= abs(v)/2) u1 = (int)u0 >> 31;   // This overflow check
      }                                                 // not quite right.

      if (i <= 8) {             // If intentional overflow case:
         cq = 0x80000000;;
         cr = 0x80000000;
      }
      else {                    // Non-overflow case:
         cq = ((int long long)u1*0x100000000LL + u0)/v;
         cr = ((int long long)u1*0x100000000LL + u0) - (int long long)cq*v;
      }
//    printf("u = %08x %08x v = %08x cq = %08x cr = %08x\r\n", u1, u0, v, cq, cr);

      q = divls(u1, u0, v, &r);
      check("divls", q, r, u1, u0, v, cq, cr);
   }

   if (errors == 0)
      printf("Passed all %d cases.\r\n", n);
   else
      printf("Got %d errors out of %d cases.\r\n", errors, n);
}
