/* Long division, unsigned (64/32 ==> 32).
   This procedure performs unsigned "long division" i.e., division of a
64-bit unsigned dividend by a 32-bit unsigned divisor, producing a
32-bit quotient.  In the overflow cases (divide by 0, or quotient
exceeds 32 bits), it returns a remainder of 0xFFFFFFFF (an impossible
value).
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

/* This first version is Knuth's Algorithm D (my "divmnu") pretty much
mechanically reduced to the special case m = 4, n = 2. But some details
differ.  The parameters are fullwords passed in registers, and it
returns an error indication if the result cannot be expressed in 32 bits
(divmnu would return a 3-halfword quotient). */

unsigned divlu(unsigned u1, unsigned u0, unsigned v, unsigned *r) {
   const unsigned b = 65536; // Number base (16 bits).
   unsigned short un[4],     // Dividend and divisor
                  vn[2];     // normalized and broken
                             // up into halfwords.
   unsigned short q[2];      // Quotient as halfwords.
   unsigned un1, un0,        // Dividend and divisor
            vn0;             // as fullwords.
   unsigned qhat;            // Estimated quotient digit.
   unsigned rhat;            // A remainder.
   unsigned p;               // Product of two digits.
   int s, i, j, t, k;

   if (u1 >= v) {            // If overflow, set rem.
      if (r != NULL)         // to an impossible value,
         *r = 0xFFFFFFFF;    // and return the largest
      return 0xFFFFFFFF;}    // possible quotient.

   s = nlz(v);               // 0 <= s <= 31.
   vn0 = v << s;             // Normalize divisor.
   vn[1] = vn0 >> 16;        // Break divisor up into
   vn[0] = vn0 & 0xFFFF;     // two 16-bit halves.

                             // Shift dividend left.
   un1 = (u1 << s) | (u0 >> 32 - s) & (-s >> 31);
   un0 = u0 << s;
   un[3] = un1 >> 16;        // Break dividend up into
   un[2] = un1;              // four 16-bit halfwords
   un[1] = un0 >> 16;        // Note: storing into
   un[0] = un0;              // halfwords truncates.

   for (j = 1; j >= 0; j--) {
      // Compute estimate qhat of q[j].
      qhat = (un[j+2]*b + un[j+1])/vn[1];
      rhat = (un[j+2]*b + un[j+1]) - qhat*vn[1];
again:
      if (qhat >= b || qhat*vn[0] > b*rhat + un[j]) {
        qhat = qhat - 1;
        rhat = rhat + vn[1];
        if (rhat < b) goto again;
      }

      // Multiply and subtract.
      k = 0;
      for (i = 0; i < 2; i++) {
         p = qhat*vn[i];
         t = un[i+j] - k - (p & 0xFFFF);
         un[i+j] = t;
         k = (p >> 16) - (t >> 16);
      }
      t = un[j+2] - k;
      un[j+2] = t;

      q[j] = qhat;              // Store quotient digit.
      if (t < 0) {              // If we subtracted too
         q[j] = q[j] - 1;       // much, add back.
         printf("Doing add back, %08x %08x %08x\n", u1, u0, v);
         k = 0;
         for (i = 0; i < 2; i++) {
            t = un[i+j] + vn[i] + k;
            un[i+j] = t;
            k = t >> 16;
         }
         un[j+2] = un[j+2] + k;
      }
   } // End j.

   if (r != NULL)            // If remainder is wanted,
      *r = (un[1]*b + un[0]) >> s;      // return it.
   return q[1]*b + q[0];
}

/* This version is simplified from Knuth's because the adding back step
is never required, as the estimated quotient digit qhat is always exact.
Also, the inner loop is coded in a more efficient way (not possible in
divmnu), unrolled, and obvious resulting simplifications made.  And, the
quotient and divisor are elementary variables (kept in registers) rather
than arrays. */

unsigned divlu1(unsigned u1, unsigned u0, unsigned v, unsigned *r) {
   const unsigned b = 65536; // Number base (16 bits).
   unsigned short un[4],     // Dividend and divisor
                  vn1, vn0;  // normalized and broken
                             // up into halfwords.
   unsigned q;               // Quotient.
   unsigned un1, un0;        // Dividend as fullwords.
   unsigned qhat;            // Estimated quotient digit.
   unsigned rhat;            // A remainder.
   unsigned p;               // Product of two digits.
   int s, j, t, k;

   if (u1 >= v) {            // If overflow, set rem.
      if (r != NULL)         // to an impossible value,
         *r = 0xFFFFFFFF;    // and return the largest
      return 0xFFFFFFFF;}    // possible quotient.

   s = nlz(v);               // 0 <= s <= 31.
   v = v << s;               // Normalize divisor.
   vn1 = v >> 16;            // Break divisor up into
   vn0 = v & 0xFFFF;         // two 16-bit halves.

                             // Shift dividend left.
   un1 = (u1 << s) | (u0 >> 32 - s) & (-s >> 31);
   un0 = u0 << s;
   un[3] = un1 >> 16;        // Break dividend up into
   un[2] = un1;              // four 16-bit halfwords
   un[1] = un0 >> 16;        // Note: storing into
   un[0] = un0;              // halfwords truncates.

   for (j = 1; j >= 0; j--) {
      // Compute estimate qhat of current digit of q.
      qhat = (un[j+2]*b + un[j+1])/vn1;
      rhat = (un[j+2]*b + un[j+1]) - qhat*vn1;
again:
      if (qhat >= b || qhat*vn0 > b*rhat + un[j]) {
        qhat = qhat - 1;
        rhat = rhat + vn1;
        if (rhat < b) goto again;
      }
      q = q*b + qhat;        // Put away quotient digit.

      // Multiply and subtract.
      p = qhat*vn0;
      t = un[j] - p;
      un[j] = t;             // Store right half of t.
      k = t >> 16;
      p = qhat*vn1;
      t = un[j+1] + k - p;
      un[j+1] = t;           // Store right half of t.
   } // End j.

   if (r != NULL)            // If remainder is wanted,
      *r = (un[1]*b + un[0]) >> s;      // return it.
   return q;
}

/* This version is divlu1 except with outer loop unrolled, and array un
changed into local variables.  Several of the variables below could be
"short," but having them fullwords gives better code on gcc/Intel.
Statistics:  Based on one million executions of this program, with
uniformly distributed random values for the dividend and divisor, the
number of times in each loop per execution of the program is:

again1: 0.4060
again2: 0.3469

This is the version that's in the hacker book. */

// ------------------------------ cut ----------------------------------
unsigned divlu2(unsigned u1, unsigned u0, unsigned v,
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
// ------------------------------ cut ----------------------------------

int errors;

void check(char *name, unsigned q, unsigned r, unsigned u1, unsigned u0, unsigned v,
           unsigned cq, unsigned cr) {

   if (q != cq || r != cr) {
      errors = errors + 1;
      printf("Error in %s, dividend u = %08x %08x\n", name, u1, u0);
      printf("                 divisor  v = %08x\n", v);
      printf("Got         q = %08x,  r = %08x\n", q, r);
      printf("Should get cq = %08x, cr = %08x\n", cq, cr);
   }
   return;
}

int main() {
   static unsigned test[] = {
   // u...,                  v...,
               0,         0,          0,        // Overflow.
               1,         0,          0,        // Overflow.
               1,         0,          1,        // Overflow.
      0xffffffff,         0, 0xffffffff,        // Overflow.
               0,         1,          1,
               0,         7,          3,
      0x00000000,0xffffffff, 0x00000001,
      0x01234567,0x89abcdef, 0x01234568,
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
      0x0000fffe,0x00000000, 0x0000ffff,
      0x0000fffe,0x0000ffff, 0x0000ffff,
      0x0000fffe,0xffff0000, 0x0000ffff,
      0x0000fffe,0xffffffff, 0x0000ffff,

      0x00000000,0x00000000, 0xffff0000,
      0x00000000,0x0000ffff, 0xffff0000,
      0x00000000,0xffff0000, 0xffff0000,
      0x00000000,0xffffffff, 0xffff0000,
      0x0000ffff,0x00000000, 0xffff0000,
      0x0000ffff,0x0000ffff, 0xffff0000,
      0x0000ffff,0xffff0000, 0xffff0000,
      0x0000ffff,0xffffffff, 0xffff0000,

      0xfffe0000,0x00000000, 0xffff0000,
      0xfffe0000,0x0000ffff, 0xffff0000,
      0xfffe0000,0xffff0000, 0xffff0000,
      0xfffe0000,0xffffffff, 0xffff0000,
      0xfffeffff,0x00000000, 0xffff0000,
      0xfffeffff,0x0000ffff, 0xffff0000,
      0xfffeffff,0xffff0000, 0xffff0000,
      0xfffeffff,0xffffffff, 0xffff0000,

      0x00000000,0x00000000, 0xffffffff,
      0x00000000,0x0000ffff, 0xffffffff,
      0x00000000,0xffff0000, 0xffffffff,
      0x00000000,0xffffffff, 0xffffffff,
      0x0000ffff,0x00000000, 0xffffffff,
      0x0000ffff,0x0000ffff, 0xffffffff,
      0x0000ffff,0xffff0000, 0xffffffff,
      0x0000ffff,0xffffffff, 0xffffffff,

      0xffff0000,0x00000000, 0xffffffff,
      0xffff0000,0x0000ffff, 0xffffffff,
      0xffff0000,0xffff0000, 0xffffffff,
      0xffff0000,0xffffffff, 0xffffffff,
      0xfffffffe,0x00000000, 0xffffffff,
      0xfffffffe,0x0000ffff, 0xffffffff,
      0xfffffffe,0xffff0000, 0xffffffff,
      0xfffffffe,0xffffffff, 0xffffffff,

      0x12345678,0x9abcdef0, 0x12345679,
      0x00010000,0x00000000, 0x00010001,
      0x80008000,0x00000000, 0x80008001,        // First qhat = b + 1.
      0x8000fffe,0xffffffff, 0x8000ffff,        // First qhat = b + 1.
      0x7fff8000,0x00000000, 0x8000ffff,        // First qhat too big by 2.
   };

   int i, n;
   unsigned u1, u0, v, q, r, cq, cr;

   n = sizeof(test)/12 + 10;

   for (i = 0; i < n; i++) {
      if (i < sizeof(test)/12) {
         u1 = test[3*i];   u0 = test[3*i+1];  v = test[3*i+2];
      }
      else {
         u1 = myrand();   u0 = myrand();  v = myrand();
         if (v == 0) v = 1;
         if (u1 >= v) u1 = u1 & (v - 1);
      }

      if (i <= 3) {             // If intentional overflow case:
         cq = 0xFFFFFFFF;;
         cr = 0xFFFFFFFF;
      }
      else {                    // Non-overflow case:
         cq = ((unsigned long long)u1*0x100000000LL + u0)/v;
         cr = ((unsigned long long)u1*0x100000000LL + u0) - (unsigned long long)cq*v;
      }
//    printf("u = %08x %08x v = %08x cq = %08x cr = %08x\r\n", u1, u0, v, cq, cr);

      q = divlu(u1, u0, v, &r);
      check("divlu", q, r, u1, u0, v, cq, cr);

      q = divlu1(u1, u0, v, &r);
      check("divlu1", q, r, u1, u0, v, cq, cr);

      q = divlu2(u1, u0, v, &r);
      check("divlu2", q, r, u1, u0, v, cq, cr);
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", n);
   else
      printf("Got %d errors out of %d cases.\n", errors, n);
}
