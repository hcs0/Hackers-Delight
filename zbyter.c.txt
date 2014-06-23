// Program for finding the rightmost 0-byte in a word.
// Max line length is 57, to fit in hacker.book.
// (Decided not to put this in the book, not very useful.)
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

// Find rightmost 0-byte, not using nlz.
int zbyter(unsigned x) {
   unsigned y;
                       // Original byte: 00 80 other
   y = (x & 0x7F7F7F7F) + 0x7F7F7F7F; // 7F 7F 1xxxxxxx
   y = ~(y | x | 0x7F7F7F7F);         // 80 00 00000000
                                      // These steps map:
   if (y == 0) return 4;              // 00000000 ==> 4,
   else if (y & 0x0000FFFF)           // xxxxxx80 ==> 0,
      return ((y >> 7) & 1) ^ 1;      // xxxx8000 ==> 1,
   else                               // xx800000 ==> 2,
      return ((y >> 23) & 1) ^ 3;     // 80000000 ==> 3.
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, r, n;
   static unsigned test[] = {0x00000000,0, 0x80000000,0,
      0x00800000,0, 0xFFFFFF00,0,
      0x00000001,1, 0xFFFF0001,1, 0x0000007F,1, 0xFFFF0080,1,
      0x00000101,2, 0xFF000101,2, 0x00007F7F,2, 0xFF00FFFF,2,
      0x00010101,3, 0x007F0101,3, 0x00807F7F,3, 0x00FFFFFF,3,
      0x01010101,4, 0x80808080,4, 0x7F7F7F7F,4, 0xFFFFFFFF,4};

   n = sizeof(test)/4;

   printf("zbyter:\n");
   for (i = 0; i < n; i += 2) {
      r = zbyter(test[i]);
      if (r != (int)test[i+1]) error(test[i], r);}

   if (errors == 0)
      printf("Passed all %d cases.\n", n/2);
}
