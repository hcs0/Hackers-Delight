/* This has functions that find the length and position of the longest
contiguous string of 1's in a word.
   Max line length is 57, to fit in hacker.book. */

#include <stdio.h>

// ------------------------------ nlz ----------------------------------

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

// ---------------------------- maxstr1 --------------------------------

/* The nicely concise function below finds the length of the longest
string of 1's in x. By Paul Hsieh, newsgroup comp.lang.c, April 29,
2005.
   It executes in 4n + 3 instructions on the basic RISC, where n is the
length of the longest string of 1's. */

int maxstr1(unsigned x) {
   int k;
   for (k = 0; x != 0; k++) x = x & 2*x;
   return k;
}

// --------------------------- fmaxstr11 -------------------------------

/* Below is an elaboration that finds the length and position of the
longest string of 1's. The position is the distance of the leftmost
bit of the string, from the left end of the string, or 32 if x = 0.
If two or more contiguous strings are the same length, this function
finds the leftmost one.
   Example: For x = 0x00FF0FF0 it returns length 8, position 8.
   By Norbert Juffa.
   Executes in 5n instructions on the basic RISC, plus the time for the
nlz function, for n >= 1, where n is the length of the longest
contiguous string of 1's (160 instructions in the worst case). */

int fmaxstr11(unsigned x, int *apos) {
   int k;
   unsigned oldx;

   oldx = 0;
   for (k = 0; x != 0; k++) {
      oldx = x;
      x &= 2*x;
   }
   *apos = nlz(oldx);
   return k;
}

// --------------------------- fmaxstr12 -------------------------------

/* Below is a "logarithmic" version. It works by propagating 0's 1, 2,
4, 8, and 16 positions to the left, stopping at the last nonzero word.
It then backtracks to find the length of the longest contiguous string
of 1's.
   For example, suppose
         x = 0011 1111 1111 0011 1111 0011 1111 1000
   Then x2 = 0011 1111 1110 0011 1110 0011 1111 0000
        x4 = 0011 1111 1000 0011 1000 0011 1100 0000
        x8 = 0011 1000 0000 0000 0000 0000 0000 0000
       x16 = all 0's

In this case the last nonzero word is x8. Observe that each 1-bit in x8
indicates the leftmost position of a string of eight 1's. Thus the
longest string of 1's begins at the leftmost position of a 1-bit in x8,
bit position 29 in the example. To test for a string of length 12, one
can test the bit at position 21 (29 - 8) in x4. Since that is 0, there
is no string of length 12. To test for a string of length 10, one can
test the bit at position 21 in x2. Since that is 1, position 29 is the
start of a string of length 10 (or more). Lastly, to test for a string
of length 11, one can test the bit at position 19 (21 - 2) in x. Since
that is 0, the longest string is of length 10, and it starts at position
29.
   This scheme does not work if x is all 1's, so that is special-cased
below, in a place that is not executed frequently.
   The worst case execution time on the basic RISC is 38 instructions
plus those required for the nlz function. If only the length of the
longest string of 1's is wanted, there is no significant savings in
execution time, except for omitting the use of the nlz function. */

int fmaxstr12(unsigned x, int *apos) {
   unsigned x2, x4, x8, x16, y, t;
   int s;

   if (x == 0) {*apos = 32; return 0;}
   x2 = x & (x << 1);
   if (x2 == 0) {s = 1; y = x; goto L1;}
   x4 = x2 & (x2 << 2);
   if (x4 == 0) {s = 2; y = x2; goto L2;}
   x8 = x4 & (x4 << 4);
   if (x8 == 0) {s = 4; y = x4; goto L4;}
   x16 = x8 & (x8 << 8);
   if (x16 == 0) {s = 8; y = x8; goto L8;}
   if (x == 0xFFFFFFFF) {*apos = 0; return 32;}
   s = 16; y = x16;

L16: t = y & (x8 << s);
     if (t != 0) {s = s + 8; y = t;}
L8:  t = y & (x4 << s);
     if (t != 0) {s = s + 4; y = t;}
L4:  t = y & (x2 << s);
     if (t != 0) {s = s + 2; y = t;}
L2:  t = y & (x  << s);
     if (t != 0) {s = s + 1; y = t;}
L1:  *apos = nlz(y);
   return s;
}

/* There are ways to rearrange the first half of the above code that
slightly reduce the worst-case execution time at the expense of average
execution time (assuming some reasonable distribution). */

// --------------------------- fmaxstr13 -------------------------------

/* The next routine is similar to the above but it uses fewer registers.
It is based on the fact that in the above function, all the information
necessary to determine the length and position of the longest string of
1-bits is contained in the last nonzero xi, so the variables x2, x4, x8,
and x16 can be replaced with one variable. A small savings results from
using two alternately, x and y in the code below.
   The worst case execution time on the basic RISC is 39 instructions
plus those required for the nlz function.
   Probably this is the best of the three log W ones to put in HD. */

int fmaxstr13(unsigned x, int *apos) {
   unsigned  y;
   int s;

   if (x == 0) {*apos = 32; return 0;}
   y = x & (x << 1);
   if (y == 0) {s = 1; goto L1;}
   x = y & (y << 2);
   if (x == 0) {s = 2; x = y; goto L2;}
   y = x & (x << 4);
   if (y == 0) {s = 4; goto L4;}
   x = y & (y << 8);
   if (x == 0) {s = 8; x = y; goto L8;}
   if (x == 0xFFFF8000) {*apos = 0; return 32;}
   s = 16;

L16: y = x & (x << 8);
     if (y != 0) {s = s + 8; x = y;}
L8:  y = x & (x << 4);
     if (y != 0) {s = s + 4; x = y;}
L4:  y = x & (x << 2);
     if (y != 0) {s = s + 2; x = y;}
L2:  y = x & (x << 1);
     if (y != 0) {s = s + 1; x = y;}
L1:  *apos = nlz(x);
   return s;
}

// --------------------------- fmaxstr14 -------------------------------

/* The function below is very similar to that above.
   The worst case execution time on the basic RISC is 39 instructions
plus those required for the nlz function. It might be of interest on a
machine that can easily evaluate a predicate to a value of 0 or 1 in a
GPR. Then, the code at lines L16 to L2 can be compiled in a branch-free
way by using the compare that gives a 0/1 result, followed by a shift
and an add. That code takes 42 instructions on the basic RISC, but it
might be preferable on some machines because of the reduced branching.
*/

int fmaxstr14(unsigned x, int *apos) {
   unsigned  y;
   int s, m;

   if (x == 0) {*apos = 32; return 0;}
   m = 0;
   y = x & (x << 1);
   if (y == 0) {s = 1; goto L1;}
   x = y & (y << 2);
   if (x == 0) {s = 2; x = y; goto L2;}
   y = x & (x << 4);
   if (y == 0) {s = 4; goto L4;}
   x = y & (y << 8);
   if (x == 0) {s = 8; x = y; goto L8;}
   if (x == 0xFFFF8000) {*apos = 0; return 32;}
   s = 16;

L16: if ((x & (x << (8   ))) != 0) {m = m + 8;}
L8:  if ((x & (x << (m+4 ))) != 0) {m = m + 4;}
L4:  if ((x & (x << (m+2 ))) != 0) {m = m + 2;}
L2:  if ((x & (x << (m+1 ))) != 0) {m = m + 1;}
L1:  *apos = nlz(x & (x << m));
   return m + s;
}

// ----------------------------- error ---------------------------------

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

// ------------------------------ main ---------------------------------

int main() {
   int i, n, len, pos;

   /* This array has a test number x, the max length of a string of 1's
   in x, and its position (position from the left or MSB of the string
   of 1's). */

   static int test[] = {0,0,32, 1,1,31, 15,4,28, 0x80000000,1,0,
      0x0F0F0F0F,4,4,  0xF0F0F0F0,4,0,  0x55555555,1,1, 0xF0000000,4,0,
      0xF0E07060,4,0,  0xFFFF0000,16,0, 0xFFFE0000,15,0,
      0xFFFF8000,17,0, 0xB77BEFDF,6,20, 0xFFFEFFFF,16,16,
      0xFFFF7FFF,16,0, 0xfffffffe,31,0, 0x7fffffff,31,1,
      0x7FFFFFFE,30,1, -1,32,0};

   n = sizeof(test)/4;

   printf("maxstr1:\n");
   for (i = 0; i < n; i += 3) {
//    printf("i = %2d, x = %08x, len = %2d\n", i, test[i], maxstr1(test[i]));
      if (maxstr1(test[i]) != test[i+1]) error(test[i], maxstr1(test[i]));}

   printf("fmaxstr11:\n");
   for (i = 0; i < n; i += 3) {
      len = fmaxstr11(test[i], &pos);
//    printf("i = %2d, x = %08x, len = %2d, pos = %2d\n", i, test[i], len, pos);
      if (len != test[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", test[i], len); errors += 1;}
      if (pos != test[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", test[i], pos); errors += 1;}
   }

   printf("fmaxstr12:\n");
   for (i = 0; i < n; i += 3) {
      len = fmaxstr12(test[i], &pos);
//    printf("i = %2d, x = %08x, len = %2d, pos = %2d\n", i, test[i], len, pos);
      if (len != test[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", test[i], len); errors += 1;}
      if (pos != test[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", test[i], pos); errors += 1;}
   }

   printf("fmaxstr13:\n");
   for (i = 0; i < n; i += 3) {
      len = fmaxstr13(test[i], &pos);
//    printf("i = %d, x = %08x, len = %d, pos = %d\n", i, test[i], len, pos);
      if (len != test[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", test[i], len); errors += 1;}
      if (pos != test[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", test[i], pos); errors += 1;}
   }


   printf("fmaxstr14:\n");
   for (i = 0; i < n; i += 3) {
      len = fmaxstr14(test[i], &pos);
//    printf("i = %2d, x = %08x, len = %2d, pos = %2d\n", i, test[i], len, pos);
      if (len != test[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", test[i], len); errors += 1;}
      if (pos != test[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", test[i], pos); errors += 1;}
   }
   if (errors == 0)
      printf("All functions passed all %d cases.\n", n/3);
   else
      printf("Got %d errors\n", errors);
   return errors;
}
