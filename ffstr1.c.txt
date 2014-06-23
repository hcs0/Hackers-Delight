// Programs for finding the first string of 1-bits of a given length.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

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

// Find first string of 1's of a given length, simple routine.
int ffstr11(unsigned x, int n) {
   int k, p;

   p = 0;               // Initialize position to return.
   while (x != 0) {
      k = nlz(x);       // Skip over initial 0's
      x = x << k;       // (if any).
      p = p + k;
      k = nlz(~x);      // Count first/next group of 1's.
      if (k >= n)       // If enough,
         return p;      // return.
      x = x << k;       // Not enough 1's, skip over
      p = p + k;        // them.
   }
   return 32;
}

// Find first string of n 1's, shift-and-and sequence.
int ffstr12(unsigned x, int n) {
   int s;

   while (n > 1) {
      s = n >> 1;
      x = x & (x << s);
      n = n - s;
   }
   return nlz(x);
}

// Find first string of n 1's, shift-and-and sequence unrolled.
int ffstr13(unsigned x, int n) {
   int s;

   s = n >> 1;
   x = x & (x << s);
   n = n - s;

   s = n >> 1;
   x = x & (x << s);
   n = n - s;

   s = n >> 1;
   x = x & (x << s);
   n = n - s;

   s = n >> 1;
   x = x & (x << s);
   n = n - s;

   s = n >> 1;
   x = x & (x << s);

   return nlz(x);
}

int errors;
void error(int x, int y, int r) {
   errors = errors + 1;
   printf("Error for x = %08x, y = %08x, got %d\n", x, y, r);
}

int main() {
   int i, r, n;
   static int test[] = {0,1,32, 0,32,32, 1,1,31, 1,2,32,
      2,1,30, 2,2,32,  3,1,30, 3,2,30,  3,3,32,
      4,1,29, 4,2,32,  4,31,32, 5,1,29, 5,2,32,
      6,1,29, 6,2,29,  6,3,32, 7,1,29,  7,2,29, 7,3,29, 7,4,32,
      8,1,28, 8,2,32,  0x80000000,1,0,  0x80000000,2,32,
      0xC0000000,1,0,  0xC0000000,2,0,  0xC0000000,3,32,
      0x5BBDF7EF,1,1,  0x5BBDF7EF,2,3,  0x5BBDF7EF,3,6,
      0x5BBDF7EF,4,10, 0x5BBDF7EF,5,15, 0x5BBDF7EF,6,21,
      0x5BBDF7EF,7,32, 0xF0F0F0F0,3,0,  0xF0F0F0F0,4,0,
      0xF0F0F0F0,5,32, 0x0F0F0F0F,3,4,  0x0F0F0F0F,4,4,
      0x0F0F0F0F,5,32, 0xFFFFFFFF,1,0,  0xFFFFFFFF,2,0,
      0xFFFFFFFF,16,0, 0xFFFFFFFF,32,0, 0x00FFFF00,15,8,
      0x00FFFF00,16,8, 0x00FFFF00,17,32};

   n = sizeof(test)/4;

   printf("ffstr11:\n");
   for (i = 0; i < n; i += 3) {
      r = ffstr11(test[i], test[i+1]);
      if (r != test[i+2]) error(test[i], test[i+1], r);}

   printf("ffstr12:\n");
   for (i = 0; i < n; i += 3) {
      r = ffstr12(test[i], test[i+1]);
      if (r != test[i+2]) error(test[i], test[i+1], r);}

   printf("ffstr13:\n");
   for (i = 0; i < n; i += 3) {
      r = ffstr13(test[i], test[i+1]);
      if (r != test[i+2]) error(test[i], test[i+1], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/3);
}
