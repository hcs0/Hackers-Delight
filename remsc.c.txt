/* Computes rems(n, d) for d = 3, 5, 7, 9 (remainder of signed
division). This material is in HD second edition, but not in the first.
Max line length is 57, to fit in hacker.book, except some of the tables
have a line length of 59 (looks ok when printed). */

#include <stdio.h>
int min1, max1, min2, max2, min3, max3, min4, max4, min5, max5;

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

//                  METHODS BASED ON SUMMING DIGITS

/* If you have the pop instruction, the code below, which uses the
Bonzini identity, is eight ops plus an indexed load. Two of the ops are
to generate the large constant (and hence can move out of a loop). */

int rems3a(int n) {

   int r;
   static char table[33] = {2, 0,1,2, 0,1,2, 0,1,2,
          0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
          0,1,2, 0,1};

   r = pop(n ^ 0xAAAAAAAA);     // Bonzini identity.
   r = table[r];
   return r - (((unsigned)n >> 31) << (r & 2));
}

/* The code below follows the technique of Figure 5-2 on page 82. 20 ops
plus an indexed load. Don't even bother to mention this in the book,
because the subsequent function (rems3c) is better and this possibility
was already mentioned in connection with unsigned remainder modulo 3. In
this code, r could as well be int (making all the shifts signed shifts).
*/

int rems3b(int n) {

   unsigned r;
   static char table[49] = {0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2,
          0, 1, 2, 0, 1, 2, 0, 1, 2, 0};

   r = (n & 0x33333333) + ((n >> 2) & 0x33333333);
   r = (r + (r >> 4)) & 0x0F0F0F0F;
   r = r + (r >> 8);
   r = r + (r >> 16);
   r = table[r & 0x3F];
   return r - (((unsigned)n >> 31) << (r & 2));
}

/* The code below is 13 elementary ops plus an indexed load. If a table
of size 62 bytes is used, it is 14 ops + an indexed load. If a table of
size 766 bytes is used, it is 11 ops + an indexed load. */

int rems3c(int n) {
   unsigned r;
   static char table[62] = {0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2, 0,1,2,
       0,1,2, 0,1,2, 0,1};

   r = n;
   r = (r >> 16) + (r & 0xFFFF);        // Max 0x1FFFE.
   r = (r >>  8) + (r & 0x00FF);        // Max 0x2FD.
   r = (r >>  4) + (r & 0x000F);        // Max 0x3D.
   r = table[r];
   return r - (((unsigned)n >> 31) << (r & 2));
}

/* The code below is 27 elementary ops assuming the multiplication by 3
is expanded into a shift and add. Don't bother to put this in the book,
enough is enough! */

int rems5a(int n) {
   int r;
   r = (n >> 16) + (n & 0xFFFF);    // FFFF8000 to 17FFE.
   r = (r >>  8) + (r & 0x00FF);    // FFFFFF80 to 27D.
   r = (r >>  4) + (r & 0x000F);    // FFFFFFF8 to 0x35.
   r = (r>>4) - ((r>>2) & 3) + (r & 3); // -4 to 6.
   r = (010432104321 >> 3*(r + 4)) & 7;   // Octal const.
   return r - (((int)(n & -r) >> 31) & 5);
}

/* The code below is 15 elementary ops + an indexed load. */

int rems5b(int n) {
   int r;
   static char table[62] = {2,3,4, 0,1,2,3,4, 0,1,2,3,4,
             0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4,
             0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4, 0,1,2,3,4,
             0,1,2,3,4, 0,1,2,3};

   r = (n >> 16) + (n & 0xFFFF);   // FFFF8000 to 17FFE.
   r = (r >>  8) + (r & 0x00FF);   // FFFFFF80 to 27D.
   r = (r >>  4) + (r & 0x000F);   // -8 to 53 (decimal).
   r = table[r + 8];
   return r - (((int)(n & -r) >> 31) & 5);
}

/* The code below is 15 elementary ops + an indexed load. */

int rems7(int n) {
   int r;
   static char table[75] =   {5,6, 0,1,2,3,4,5,6,
     0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4,5,6,
     0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4,5,6,
     0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2,3,4,5,6, 0,1,2};

   r = (n >> 15) + (n & 0x7FFF);   // FFFF0000 to 17FFE.
   r = (r >>  9) + (r & 0x001FF);  // FFFFFF80 to 2BD.
   r = (r >>  6) + (r & 0x0003F);  // -2 to 72 (decimal).
   r = table[r + 2];
   return r - (((int)(n & -r) >> 31) & 7);
}

/* The code below is 14 elementary ops + an indexed load. Don't bother
putting this in the book, because rems9b is probably better (one more
instruction but a smaller table, particularly if a larger table is used
to reduce the instruction count by three). */

int rems9a(int n) {
   int r;
   static char table[128] = {0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
          0,1,2,3,4,5,6,7,8, 0,1};

   r = (n >> 18) + (n & 0x3FFFF);  // FFFFE000 to 41FFE.
   r = (r >> 12) + (r & 0x00FFF);  // FFFFFFFE to 103F.
   r = (r >>  6) + (r & 0x0003F);  // 0 to 127 (decimal).
   r = table[r];
   return r - (((int)(n & -r) >> 31) & 9);
}

/* The code below is 15 elementary ops + an indexed load.
12 ops if a table of size 831 (decimal) is used. */

int rems9b(int n) {
   int r;
   static char table[75] = {7,8, 0,1,2,3,4,5,6,7,8,
              0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
              0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
              0,1,2,3,4,5,6,7,8, 0,1,2,3,4,5,6,7,8,
              0,1,2,3,4,5,6,7,8, 0};

   r = (n & 0x7FFF) - (n >> 15);   // FFFF7001 to 17FFF.
   r = (r & 0x01FF) - (r >>  9);   // FFFFFF41 to 0x27F.
   r = (r & 0x003F) + (r >>  6);   // -2 to 72 (decimal).
   r = table[r + 2];
   return r - (((int)(n & -r) >> 31) & 9);
}

//                  METHODS BASED ON MULTIPLICATION

/* The code below is 12 ops including a multiply. */

int rems3m(int n) {
   unsigned r;

   r = n;
   r = (0x55555555*r + (r >> 1) - (r >> 3)) >> 30;
   return r - (((unsigned)n >> 31) << (r & 2));
}

/* Based on floor((8/5)(5k+r)) mod 8 = 0, 1, 3, 4, 6, 7 for r = 0, 1, 2,
3, 4, 0 resp. Hence must translate. Without the term n >> 3, it fails
for n = -1 and maybe all negative numbers, so it's not worth mentioning
anything about dropping the accuracy terms. 10 ops including a multiply.
*/

#define u 99
int rems5m(int n) {
   unsigned r;
   static signed char table[16] = {0, 1, 2, 2, 3, u, 4, 0,
                                   u, 0,-4, u,-3,-2,-2,-1};

   r = n;
   r = ((0x33333333*r) + (r >> 3)) >> 29;
   return table[r + (((unsigned)n >> 31) << 3)];
}

/* Based on floor((8/7)(7k+r)) mod 8 = 0, 1, 2, 3, 4, 5, 6, 7 for r = 0,
1, 2, 3, 4, 5, 6, 0 resp. Hence must translate, just mapping 7 to 0 and
then fixing the cases n < 0. Without the term r >> 4, it fails for n =
-4 and many other negative numbers, so it's not worth mentioning
anything about dropping the accuracy terms. 19 ops including a multiply.
*/

int rems7m(int n) {
   unsigned r;

   r = n - (((unsigned)n >> 31) << 2);  // Fix for sign.
   r = ((0x24924924*r) + (r >> 1) + (r >> 4)) >> 29;
   r = r & ((int)(r - 7) >> 31);        // Change 7 to 0.
   return r - (((int)(n&-r) >> 31) & 7);// Fix n<0 case.
}

/* Based on as approximation of ((16/9)(9k+r)) mod 16. Must translate 2
to 1, 3 to 2, etc. The table was arrived at by experimentation. Without
the term n >> 1, first error is at n = 9. 4 ops including a multiply,
plus an indexed load. Table entries marked "u" are unused. */

int rems9m(int n) {
   unsigned r;
   static signed char table[32] = {0, 1, 1, 2, u, 3, u, 4,
                                   5, 5, 6, 6, 7, u, 8, u,
                                  -4, u,-3, u,-2,-1,-1, 0,
                                   u,-8, u,-7,-6,-6,-5,-5};

   r = n;
   r = (0x1C71C71C*r + (r >> 1)) >> 28;
   return table[r + (((unsigned)n >> 31) << 4)];
}

/* Based on an approximation of ((16/10)(10k+r)) mod 16. Must translate
3 to 2, 4 to 3, etc. Without the term n >> 3, it fails for n = -1 and
maybe all negative numbers, so it's not worth mentioning anything about
dropping the accuracy terms. 11 ops including a multiply. */

int rems10m(int n) {
   unsigned r;
   static signed char table[32] = {0, 1, u, 2, 3, u, 4, 5,
                                   5, 6, u, 7, 8, u, 9, u,
                                  -6,-5, u,-4,-3,-3,-2, u,
                                  -1, 0, u,-9, u,-8,-7, u};
   r = n;
   r = (0x19999999*r + (r >> 1) + (r >> 3)) >> 28;
   return table[r + (((unsigned)n >> 31) << 4)];
}

int errors;
void error(int n, int r) {
   errors = errors + 1;
   printf("Error for n = %08x, got %d\n", n, r);
}

int main() {
   int n, r;
   const int low = 0xFF000000,    // Set these equal for an exhaustive
            high = 0x00F00000;    // test.

   printf("rems3a:\n");
   n = low; do {
      r = rems3a(n);
      if (r != n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems3b:\n");
   n = low; do {
      r = rems3b(n);
      if (r != n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems3c:\n");
   n = low; do {
      r = rems3c(n);
      if (r != n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems5a:\n");
   n = low; do {
      r = rems5a(n);
      if (r != n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems5b:\n");
   n = low; do {
      r = rems5b(n);
      if (r != n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems7:\n");
   n = low; do {
      r = rems7(n);
      if (r != n%7) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems9a:\n");
   n = low; do {
      r = rems9a(n);
      if (r != n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems9b:\n");
   n = low; do {
      r = rems9b(n);
      if (r != n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems3m:\n");
   n = low; do {
      r = rems3m(n);
      if (r != n%3) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems5m:\n");
   n = low; do {
      r = rems5m(n);
      if (r != n%5) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems7m:\n");
   n = low; do {
      r = rems7m(n);
      if (r != n%7) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems9m:\n");
   n = low; do {
      r = rems9m(n);
      if (r != n%9) error(n, r);
      n = n + 1;
   } while (n != high);

   printf("rems10m:\n");
   n = low; do {
      r = rems10m(n);
      if (r != n%10) error(n, r);
      n = n + 1;
   } while (n != high);

// printf("min1 = %X, max1 = %X\n", min1, max1);
// printf("min2 = %X, max2 = %X\n", min2, max2);
// printf("min3 = %X, max3 = %X\n", min3, max3);
// printf("min4 = %X, max4 = %X\n", min4, max4);
// printf("min5 = %X, max5 = %X\n", min5, max5);
   if (errors == 0)
      printf("Passed all tests.\n");
}
