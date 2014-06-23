/* This has two functions that find the length and position of the
shortest contiguous string of 1's in a word. Also below are two
functions that find the length and position of the shortest contiguous
string of 1's in a word that is at least as long as a given integer n
("bestfit").
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

// ------------------------------ pop ----------------------------------

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x >> 8);
   x = x + (x >> 16);
   return x & 0x0000003F;
}

// ---------------------------- fminstr1 -------------------------------

/* This function finds the length and position of the shortest
contiguous string of 1's in a word. The position is the distance of the
leftmost bit of the string, from the left end of the string, or 32 if x
= 0. If two or more contiguous strings are the same length, this
function finds the leftmost one.
   Example: For x = 0x00FF0FF0 it returns length 8, position 8.
   Executes in 8 + 4n instructions on the basic RISC (w/o andc), plus
the time for the nlz function, for n >= 2, where n is the length of the
shortest contiguous string of 1's in x. */

int fminstr1(unsigned x, int *apos) {
   int k;
   unsigned b, e;       // Beginnings, ends.

   if (x == 0) {*apos = 32; return 0;}
   b = ~(x >> 1) & x;   // 0-1 transitions.
   e = x & ~(x << 1);   // 1-0 transitions.
   for (k = 1; (b & e) == 0; k++)
      e = e << 1;
   *apos = nlz(b & e);
   return k;
}

// --------------------------- fminstr11 -------------------------------

/* This function performs the same functions as fminstr1. It might be
useful is your machine has population count as an instruction. The loop
is executed a number of times equal to the number of contiguous strings
of 1-bits in x.
   Executes in 5 + 11n instructions on the full RISC, where n is the
number of strings of 1's in x, for n >= 1 (that is, for x != 0). This
assumes the if-test goes either way half the time, and that pop and nlz
count as one instruction each.
   If "(k <= kmin)" is changed to "(k < kmin)", it finds the RIGHTmost
shortest contiguous string of 1's, when two or more such strings are the
same length.
   If that expression is changed to "(k >= kmin)" and the initialization
of kmin is changed to "kmin = 0;", it finds the leftmost LONGEST
contiguous string of 1's. And if the comparison is changed to ">" and
the initialization of kmin is changed to "kmin = -1;", it finds the
rightmost longest contiguous string of 1's.
   The code is also easily modified to compute the "bestfit" function. */

int fminstr11(unsigned x, int *apos) {
   int k, kmin, y0, y;
   unsigned int x0, xmin;

   kmin = 32;
   y0 = pop(x);
   x0 = x;
   do {
      x = ((x & -x) + x) & x;   // Turn off rightmost
      y = pop(x);               // string.
      k = y0 - y;               // k = length of string
      if (k <= kmin) {          // turned off.
         kmin = k;              // Save shortest length
         xmin = x;              // found, and the string.
      }
      y0 = y;
   } while (x != 0);
   *apos = nlz(x0 ^ xmin);
   return kmin;
}

// ---------------------------- bestfit --------------------------------

/* This function finds the length and position of the shortest string of
1's in x that is of length n or more, for n >= 1. If such a string does
not exist, it returns with apos = 32 and the length undefined (actually,
the returned value is n - 1). */

int bestfit(unsigned x, int n, int *apos) {
   int m, s;

   m = n;
   while (m > 1) {
      s = m >> 1;
      x = x & (x << s);
      m = m - s;
   }
   return fminstr1(x, apos) + n - 1;
}

// ---------------------------- bestfit1 -------------------------------

/* This function finds the length and position of the shortest string of
1's in x that is of length n or more, for n >= 0. If such a string does
not exist, it returns with apos = 32 and the length undefined (actually,
the returned value is 32).
   This is a simple modification of fminstr11. */

int bestfit1(unsigned x, int n, int *apos) {
   int k, kmin, y0, y;
   unsigned int x0, xmin;

   kmin = 32;
   xmin = x;
   y0 = pop(x);
   x0 = x;
   do {
      x = ((x | (x - 1)) + 1) & x;      // Turn off
      y = pop(x);               // rightmost string.
      k = y0 - y;               // k = length of string
      if (k <= kmin && k >= n) {// turned off.
         kmin = k;              // Save shortest length
         xmin = x;              // found, and the string.
      }
      y0 = y;
   } while (x != 0);
   *apos = nlz(x0 ^ xmin);
   return kmin;
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

   /* This array, for testing "fminstr1," has a test number x, the
   length of the shortest string of 1's in x, and its position (distance
   from the left or MSB). */

   static int testa[] = {0,0,32, 1,1,31, 15,4,28,        0x80000000,1,0,
      0x0F0F0F0F,4,4,   0xF0F0F0F0,4,0,  0x55555555,1,1, 0xF0000000,4,0,
      0xF0E07060,2,25,  0xFFFF0000,16,0, 0xFFFE0000,15,0,
      0xFFFF8000,17,0,  0xB57EEFDF,1,0,  0xFFFEFFFF,15,0,
      0xFFFF7FFF,15,17, 0xfffffffe,31,0, 0x7fffffff,31,1,
      0x7FFFFFFE,30,1,  -1,32,0,         0xfefdfdff,6,8};

   /* This array, for testing "bestfit," has a test number x, the
   minimum acceptable length of a string of 1's in x, the length
   actually found, and the position (distance from the left or MSB)
   where the first string of at least the minimum acceptable length is
   found. In the case that a string of the requested length is not
   found, the position is given as 32, and the length in this table is
   the requested length less 1, which is the value returned by bestfit,
   but not by bestfit1. */

   static int testb[] = {0,1,0,32, 1,1,1,31, 15,6,5,32, 15,5,4,32,
      15,4,4,28, 15,3,4,28, 15,2,4,28, 15,1,4,28, 0x80000000,1,1,0,
      0x80000000,2,1,32,    0x80000000,3,2,32,    0xE0000000,1,3,0,
      0xE0000000,2,3,0,     0xE0000000,3,3,0,     0xE0000000,4,3,32,
      0x0F0F0F0F,1,4,4,     0x0F0F0F0F,2,4,4,     0x0F0F0F0F,3,4,4,
      0x0F0F0F0F,4,4,4,     0x0F0F0F0F,5,4,32,    0x0F0F80FC,1,4,4,
      0x0F0F80FC,2,4,4,     0x0F0F80FC,3,4,4,     0x0F0F80FC,5,5,12,
      0x0F0F80FC,6,6,24,    0x0F0F80FC,7,6,32,    0x0F0F80FC,8,7,32,
      0x12345678,1,1,3,     0x12345678,2,2,10,    0x12345678,3,4,25,
      0x12345678,4,4,25,    0x12345678,5,4,32,    0x12345678,6,5,32,
      0xF8FFF7FF,10,11,21,  0xF8FFF7FF,11,11,21,  0xF8FFF7FF,12,12,8,
      0xF8FFF7FF,13,12,32,  0x7FFFFFFF,1,31,1,    0x7FFFFFFF,30,31,1,
      0x7FFFFFFF,31,31,1,   0x7FFFFFFF,32,31,32,  0xFFFFFFFE,1,31,0,
      0xFFFFFFFE,30,31,0,   0xFFFFFFFE,31,31,0,   0xFFFFFFFE,32,31,32,
      0xFFFFFFFF,1,32,0,    0xFFFFFFFF,31,32,0,   0xFFFFFFFF,32,32,0,
      0xFFFFFFFF,33,32,32,  0xFFFFFFFF,99,98,32};

   n = sizeof(testa)/4;

   printf("\nfminstr1:\n");
   for (i = 0; i < n; i += 3) {
      len = fminstr1(testa[i], &pos);
      printf("x = %08x, len = %2d, pos = %2d\n", testa[i], len, pos);
      if (len != testa[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", testa[i], len); errors += 1;}
      if (pos != testa[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", testa[i], pos); errors += 1;}
   }

   printf("\nfminstr11:\n");
   for (i = 0; i < n; i += 3) {
      len = fminstr11(testa[i], &pos);
      printf("x = %08x, len = %2d, pos = %2d\n", testa[i], len, pos);
      if (len != testa[i+1]) {
         printf("Incorrect length for x = %08x, got %d\n", testa[i], len); errors += 1;}
      if (pos != testa[i+2]){
         printf("Incorrect bit position for x = %08x, got %d\n", testa[i], pos); errors += 1;}
   }

   n = sizeof(testb)/4;

   printf("\nbestfit:\n");
   for (i = 0; i < n; i += 4) {
      len = bestfit(testb[i], testb[i+1], &pos);
      printf("x = %08x, req = %2d, len = %2d, pos = %2d\n", testb[i], testb[i+1], len, pos);
      if (len != testb[i+2]) {
         printf("Incorrect actual length for x = %08x, req = %d, got %d\n", testb[i], testb[i+1], len); errors += 1;}
      if (pos != testb[i+3]){
         printf("Incorrect bit position for x = %08x, req = %d, got %d\n", testb[i], testb[i+1], pos); errors += 1;}
   }

   printf("\nbestfit1:\n");
   for (i = 0; i < n; i += 4) {
      len = bestfit1(testb[i], testb[i+1], &pos);
      printf("x = %08x, req = %2d, len = %2d, pos = %2d\n", testb[i], testb[i+1], len, pos);
      if (pos != 32 && len != testb[i+2] ||
          pos == 32 && len != 32) {
         printf("Incorrect actual length for x = %08x, req = %d, got %d\n", testb[i], testb[i+1], len); errors += 1;}
      if (pos != testb[i+3]){
         printf("Incorrect bit position for x = %08x, req = %d, got %d\n", testb[i], testb[i+1], pos); errors += 1;}
   }

   if (errors == 0)
      printf("Passed all test cases.\n");
   else
      printf("Got %d errors\n", errors);
   return errors;
}
