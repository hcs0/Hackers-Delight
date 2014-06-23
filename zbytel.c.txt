// Programs for finding the leftmost 0-byte in a word.
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

// Find leftmost 0-byte, simple sequence of tests.
int zbytel1(unsigned x) {
   if             ((x >> 24) == 0) return 0;
   else if ((x & 0x00FF0000) == 0) return 1;
   else if ((x & 0x0000FF00) == 0) return 2;
   else if ((x & 0x000000FF) == 0) return 3;
   else return 4;
}

// Find leftmost 0-byte, branch-free code.
int zbytel2(unsigned x) {
   unsigned y;
   int n;
                         // Original byte: 00 80 other
   y = (x & 0x7F7F7F7F) + 0x7F7F7F7F;   // 7F 7F 1xxxxxxx
   y = ~(y | x | 0x7F7F7F7F);           // 80 00 00000000
   n = nlz(y) >> 3;             // n = 0 ... 4, 4 if x
   return n;                    // has no 0-byte.
}

// Find leftmost 0-byte, not using nlz.
int zbytel3(unsigned x) {
   unsigned y;
                       // Original byte: 00 80 other
   y = (x & 0x7F7F7F7F) + 0x7F7F7F7F; // 7F 7F 1xxxxxxx
   y = ~(y | x | 0x7F7F7F7F);         // 80 00 00000000
                                      // These steps map:
   if (y == 0) return 4;              // 00000000 ==> 4,
   else if (y > 0x0000FFFF)           // 80xxxxxx ==> 0,
      return (y >> 31) ^ 1;           // 0080xxxx ==> 1,
   else                               // 000080xx ==> 2,
      return (y >> 15) ^ 3;           // 00000080 ==> 3.
}

int zbytel3a(unsigned x) {
   unsigned y;
   static char table[16] = {4, 3, 2, 2, 1, 1, 1, 1,
                            0, 0, 0, 0, 0, 0, 0, 0};
                       // Original byte: 00 80 other
   y = (x & 0x7F7F7F7F) + 0x7F7F7F7F; // 7F 7F 1xxxxxxx
   y = ~(y | x | 0x7F7F7F7F);         // 80 00 00000000
   return table[y*0x00204081 >> 28];
}

// Find leftmost 0-byte by evaluating a polynomial.
int zbytel4(unsigned x) {
   unsigned y, t1, t2, t3, t4;

   y = (x & 0x7F7F7F7F) + 0x7F7F7F7F;
   y = y | x;           // Leading 1 on nonzero bytes.

   t1 =  y >> 31;               // t1 = a.
   t2 = (y >> 23) & t1;         // t2 = ab.
   t3 = (y >> 15) & t2;         // t3 = abc.
   t4 = (y >>  7) & t3;         // t4 = abcd.
   return t1 + t2 + t3 + t4;
}

// Find leftmost byte having value le 9.
int valle9(unsigned x) {
   unsigned y;
   int n;

   y = (x & 0x7F7F7F7F) + 0x76767676;
   y = y | x;
   y = y | 0x7F7F7F7F;          // Bytes > 9 are 0xFF.
   y = ~y;                      // Bytes > 9 are 0x00,
                                // bytes <= 9 are 0x80.
   n = nlz(y) >> 3;
   return n;
}

// Find leftmost byte having an upper case letter.
int valupcase(unsigned x) {
   unsigned d, y;
   int n;

   d = (x | 0x80808080) - 0x41414141;
   d = ~((x | 0x7F7F7F7F) ^ d);
   y = (d & 0x7F7F7F7F) + 0x66666666;
   y = y | d;
   y = y | 0x7F7F7F7F;    // Bytes not from 41-5A are FF.
   y = ~y;                // Bytes not from 41-5A are 00,
                          // bytes from 41-5A are 80.
   n = nlz(y) >> 3;
   return n;
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, r, n;
   static unsigned test[] = {0x00000000,0, 0x00000001,0,
      0x00800000,0, 0x00FFFFFF,0,
      0x01000000,1, 0x0100FFFF,1, 0x7F000000,1, 0x8000FFFF,1,
      0x01010000,2, 0x010100FF,2, 0x7F7F0000,2, 0xFFFF00FF,2,
      0x01010100,3, 0x01017F00,3, 0x7F7F8000,3, 0xFFFFFF00,3,
      0x01010101,4, 0x80808080,4, 0x7F7F7F7F,4, 0xFFFFFFFF,4};

   static unsigned test2[] = {0x00000000,0, 0x00000001,0,
      0x09000000,0, 0x09FFFFFF,0, 0x0A010000,1, 0x1909FFFF,1,
      0xFF0A0000,2, 0x80800900,2, 0x81810980,2, 0xFEFE0822,2,
      0x0A0A0A00,3, 0x0A0A0A09,3, 0x0A0A0A07,3, 0xFFFFFF01,3,
      0x0A0A0A0A,4, 0xFFFFFFFF,4, 0x80808080,4, 0x7F7F7F7F,4};

   static unsigned test3[] = {0x00000000,4, 0x00000001,4,
      0x40000000,4, 0x40FFFFFF,4, 0x41000000,0, 0x41FFFFFF,0,
      0x5A000000,0, 0x5AFFFFFF,0, 0x5B000000,4, 0x5BFFFFFF,4,
      0x00400000,4, 0xFF40FFFF,4, 0x00410000,1, 0xFF41FFFF,1,
      0x005A0000,1, 0xFF5AFFFF,1, 0x005B0000,4, 0xFF5BFFFF,4,
      0x80804080,4, 0x80804180,2, 0x80805A80,2, 0x80805B80,4,
      0x7F7F7F40,4, 0x7F7F7F41,3, 0x7F7F7F5A,3, 0x7F7F7F5B,4,
      0x41414141,0, 0x5A5A5A5A,0, 0x40404141,2, 0x5B5B5A5A,2,
      0x80808080,4, 0x7F7F7F7F,4, 0xFFFFFFFF,4};

   n = sizeof(test)/4;

   printf("zbytel1:\n");
   for (i = 0; i < n; i += 2) {
      r = zbytel1(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("zbytel2:\n");
   for (i = 0; i < n; i += 2) {
      r = zbytel2(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("zbytel3:\n");
   for (i = 0; i < n; i += 2) {
      r = zbytel3(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("zbytel3a:\n");
   for (i = 0; i < n; i += 2) {
      r = zbytel3a(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   printf("zbytel4:\n");
   for (i = 0; i < n; i += 2) {
      r = zbytel4(test[i]);
      if (r != test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);

   n = sizeof(test2)/4;

   printf("valle9:\n");
   for (i = 0; i < n; i += 2) {
      r = valle9(test2[i]);
      if (r != test2[i+1]) error(test2[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);

   n = sizeof(test3)/4;

   printf("valupcase:\n");
   for (i = 0; i < n; i += 2) {
      r = valupcase(test3[i]);
      if (r != test3[i+1]) error(test3[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);
}
