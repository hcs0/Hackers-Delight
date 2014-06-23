// Programs for reversing bits.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     // To define "exit", req'd by XLC.

// Rotate shift left.
unsigned shlr(unsigned x, int n) {
   return (x << n) | (x >> (32 - n));
}

// Reversing bits in a word, basic interchange scheme.
unsigned rev1(unsigned x) {
   x = (x & 0x55555555) <<  1 | (x & 0xAAAAAAAA) >>  1;
   x = (x & 0x33333333) <<  2 | (x & 0xCCCCCCCC) >>  2;
   x = (x & 0x0F0F0F0F) <<  4 | (x & 0xF0F0F0F0) >>  4;
   x = (x & 0x00FF00FF) <<  8 | (x & 0xFF00FF00) >>  8;
   x = (x & 0x0000FFFF) << 16 | (x & 0xFFFF0000) >> 16;
   return x;
}

// Reversing bits in a word, refined basic scheme.
// 24 operations + 6 for loading constants = 30 insns.
// This is Figure 7-1 in HD.
unsigned rev2(unsigned x) {
   x = (x & 0x55555555) <<  1 | (x >>  1) & 0x55555555;
   x = (x & 0x33333333) <<  2 | (x >>  2) & 0x33333333;
   x = (x & 0x0F0F0F0F) <<  4 | (x >>  4) & 0x0F0F0F0F;
   x = (x << 24) | ((x & 0xFF00) << 8) |
       ((x >> 8) & 0xFF00) | (x >> 24);
   return x;
}

// Reversing the rightmost 6 bits in a word.
unsigned rev3(unsigned x) {
   return (x*0x00082082 & 0x01122408) % 255;
}

// Reversing the rightmost 6 bits in a word.
unsigned rev4(unsigned x) {
   unsigned t;
   t = x*0x00082082 & 0x01122408;
   return (t*0x01010101) >> 24;
}

// Reversing the rightmost 8 bits in a word.
// Problem: can't change divide by 1023 to a 32x32 ==> 32 multiply.
unsigned rev5(unsigned x) {
   unsigned s, t;
   s = x*0x02020202 & 0x84422010;
   t = x*8 & 0x00000420;
   return (s + t) % 1023;
}

// Reversing the rightmost 8 bits in a word.
unsigned rev6(unsigned x) {
   unsigned s, t;
   s = x*0x00020202 & 0x00422010;
   t = x*0x00008008 & 0x00210420;
   return (s + t) % 1023;
}

// Reversing the rightmost 8 bits in a word.
unsigned rev7(unsigned x) {
   unsigned s, t;

   s = x*0x00020202 & 0x01044010;
   t = x*0x00080808 & 0x02088020;
   return (s + t) % 4095;
}

// Reversing the rightmost 8 bits in a word.
// Same as above but made more efficient, 15 Brisc ops.
unsigned rev8(unsigned x) {
   unsigned u, m, s, t;

   u = x*0x00020202;
   m = 0x01044010;
   s = u & m;
   t = (u << 2) & (m << 1);
   return 0x01001001*(s + t) >> 24;
// return (s + t) % 4095;       // Alternative, equivalent to above.
}

// Reversing the rightmost 9 bits in a word.
// Can't change the remainder to a multiply.
unsigned rev9(unsigned x) {
   unsigned s, t;

   s = x*0x01001001 & 0x84108010;
// t = x*0x00040040 & 0x00841080;
   t = ((x*0x01001001) >> 6) & (0x84108010 >> 8);  // Alternative,
                                // better after commoning.
   return (s + t) % 1023;
}

// Reversing the rightmost 9 bits in a word.
// Can't change the remainder to a multiply.
// Probably rev9 is a little better --- smaller numbers.
unsigned rev10(unsigned x) {
   unsigned s, t;

   s = x*0x40100401 & 0x84108010;
   t = x*0x04010040 & 0x21042000;
   return (s + t) % 4095;
}

// Same as above but made more efficient.
unsigned rev10a(unsigned x) {
   unsigned u, m, s, t;

   u = x*0x40100401;
   m = 0x84108010;
   s = u & m;
   t = (u << 6) & (m >> 2);
   return (s + t) % 4095;
}

/* The following routines are new (not in HD first edition).

   The routine below reverses the rightmost 16 bits of a word, assuming
the leftmost 16 bits are clear at the start. It is an old algorithm by
Christopher Strachey (Bitwise Operations. Communications of the ACM 4, 3
(March 1961), 146).
   The way it works is to build up the reversed halfword in the left
half of the register x. It does this by noting how many positions each
bit must move to the left, to be placed in its proper position in the
left half of the word. Then, each bit that must move 16 or more
positions is moved left 16. Then each bit that must move 8 or more
positions (after the first move of 16 positions) is moved left 8. Then
4, and 2. Lastly a shift left of 1 is required to put the result in the
left half of the register (as in Strachey), or a shift right of 15 is
done to put the result in the rightmost 16 bits (as is done here).
   Here is how far to the left the bits in postions 0 to 15 must move:
   31 30 ... 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
                    1  3  5  7  9 11 13 15 17 19 21 23 25 27 29 31

   0000 0000 0000 0000 abcd efgh ijkl mnop  Given
   0000 0000 ijkl mnop abcd efgh ijkl mnop  After shl 16
   0000 mnop ijkl efgh abcd mnop ijkl mnop  After shl 8
   00op mnkl ijgh efcd abop mnkl ijop mnop  After shl 4
   0pon mlkj ihgf edcb apon mlkj ipon mpop  After shl 2
   0000 0000 0000 0000 ponm lkji hgfe dcba  After shr 15

16 operations.
 */

unsigned rev11(unsigned x) {
   x = x | ((x & 0x000000FF) << 16);
   x = (x & 0xF0F0F0F0) | ((x & 0x0F0F0F0F) << 8);
   x = (x & 0xCCCCCCCC) | ((x & 0x33333333) << 4);
   x = (x & 0xAAAAAAAA) | ((x & 0x55555555) << 2);
   x = x >> 15;
   return x;
}

/* By using different (and more irregular) masks, the code for
Strachey's method can be made to preserve the right half of the register
and to put the reversed halfword in the left half. It can be made to use
fewer masks at no loss of efficiency in ways that will be discussed with
rev12 below.
   Here is the more efficient version (still 16 operations): */

unsigned rev11a(unsigned x) {
   unsigned t;
   x = x | ((x & 0x000000FF) << 16);
   t = x & 0x0F0F0F0F; x = (t <<  8) | (t ^ x);
   t = x & 0x33333333; x = (t <<  4) | (t ^ x);
   t = x & 0x55555555; x = (t <<  2) | (t ^ x);
   x = x >> 15;
   return x;
}

/* Reversing bits in a word using rotate shifts and the "and not"
instruction. 17 operations (25 full RISC instructions).
15 cycles on a machine with sufficient parallelism.
   Don't know why I didn't put this in HD. Maybe next time. */

unsigned rev12(unsigned x) {
   x = shlr(x & 0x00FF00FF, 16) | x & ~0x00FF00FF;
   x = shlr(x & 0x0F0F0F0F,  8) | x & ~0x0F0F0F0F;
   x = shlr(x & 0x33333333,  4) | x & ~0x33333333;
   x = shlr(x & 0x55555555,  2) | x & ~0x55555555;
   x = shlr(x, 1);
   return x;
}

/* The "and not" operation can be avoided by using the
3-instruction method of doing the MUX operation, i.e.,
   (x & m) | (y & ~m) = ((x ^ y) & m) ^ y.
Rewriting the first line above as
   x = shlr(x, 16) & 0x00FF00FF | x & ~0x00FF00FF;
it can be changed to
   x = ((shlr(x, 16) ^ x) & 0x00FF00FF) ^ x;
This gives the function below (17 operations, 25 full RISC
instructions). */

unsigned rev12a(unsigned x) {
   x = ((shlr(x, 16) ^ x) & 0x00FF00FF) ^ x;
   x = ((shlr(x,  8) ^ x) & 0x0F0F0F0F) ^ x;
   x = ((shlr(x,  4) ^ x) & 0x33333333) ^ x;
   x = ((shlr(x,  2) ^ x) & 0x55555555) ^ x;
   x = shlr(x, 1);
   return x;
}

/* The "and not" can also be avoided by using the identity
   x & ~m = (x & m) ^ x
to change
   x = shlr(x & 0x00FF00FF, 16) | x & ~0x00FF00FF;
to the first executable line in the function below. By Vesa Karvonen,
found in The Assembly Gems Page,
http://www.df.lth.se/~john_e/fr_gems.html.
   17 operations (25 full RISC instructions). This is slightly
preferable to rev12a for many machines because it has a little
instruction-level parallelism in each of the four steps, whereas rev12a
has none. 15 cycles on a machine with sufficient parallelism. */

unsigned rev12b(unsigned x) {
   unsigned t;
   t = x & 0x00FF00FF; x = shlr(t, 16) | (t ^ x);
   t = x & 0x0F0F0F0F; x = shlr(t,  8) | (t ^ x);
   t = x & 0x33333333; x = shlr(t,  4) | (t ^ x);
   t = x & 0x55555555; x = shlr(t,  2) | (t ^ x);
   x = shlr(x, 1);
   return x;
}

/* Basic ternary method, applied to a 27-bit word. I.e., this works "in
place" on the 27 bits, and hence would work on a machine with 27-bit
registers. The scheme:

   012345678 9abcdefgh ijklmnopq   The given 27-bit word
   ijklmnopq 9abcdefgh 012345678   First ternary swap
   opqlmnijk fghcde9ab 678345012   Second ternary swap
   qponmlkji hgfedcba9 876543210   Third ternary swap

This is coded below. It is 21 operations. Counting loading of the 5
masks (the 1FF mask need not be loaded), it is 31 basic RISC
instructions. */

unsigned rev13(unsigned x) {
   x = (x & 0x000001FF) << 18 | (x & 0x0003FE00) |
       (x >> 18) & 0x000001FF;
   x = (x & 0x001C0E07) <<  6 | (x & 0x00E07038) |
       (x >> 6) & 0x001C0E07;
   x = (x & 0x01249249) <<  2 | (x & 0x02492492) |
       (x >> 2) & 0x01249249;
   return x;
}

/* If operating in a 32-bit register, we could have used the binary
method (rev2) followed by a right shift of 5. This would be 25
operations, or 31 basic RISC instructions. */

/* The next routine, by Donald E. Knuth, reverses a 32-bit word in only
four steps. It uses "ternary" swapping of some fields, and a rotate
shift for one step (nice, especially if you have that instruction,
because it doesn't need masks). It operates as follows (c.f. a note
from DEK):

   01234567 89abcdef ghijklmn opqrstuv (given)
   fghijklm nopqrstu v0123456 789abcde (rotate left 15)
   pqrstuvm nofghijk labcde56 78901234 (10-swap)
   tuvspqrm nojklifg hebcda96 78541230 (4-swap)
   vutsrqpo mnlkjihg fedcba98 76543210 (2-swap)

If you have the rotate shift, this is 22 operations, or 34 instructions,
counting loading the six distinct masks. Beats rev2 by 2 operations
or 2 instructions. But it doesn't beat rev12a  or rev12b (17 ops, 25 insns). */

unsigned rev14(unsigned x) {
   x = shlr(x, 15);             // Rotate left 15.
// x = (x << 15) | (x >> 17);   // Alternative.
   x = (x & 0x003F801F) << 10 | (x & 0x01C003E0) |
       (x >> 10) & 0x003F801F;
   x = (x & 0x0E038421) <<  4 | (x & 0x11C439CE) |
       (x >>  4) & 0x0E038421;
   x = (x & 0x22488842) <<  2 | (x & 0x549556B5) |
       (x >>  2) & 0x22488842;
   return x;
}

/* Next is a variation of rev14 that replaces assignments of the form

   x = (x & M1) << s | (x & M2) | (x >> s) & M1;

where M2 is ~(M1 | (M1 << s)), with:

   t = (x ^ (x >> s)) & M1; x = (t | (t << s)) ^ x;

which saves one operation (three instructions) on each of the ternary
swap steps. Thus, it takes 19 operations or 25 instructions. */

unsigned rev14a(unsigned x) {
   unsigned t;

   x = shlr(x, 15);             // Rotate left 15.
// x = (x << 15) | (x >> 17);   // Alternative.
   t = (x ^ (x>>10)) & 0x003F801F; x = (t | (t<<10)) ^ x;
   t = (x ^ (x>> 4)) & 0x0E038421; x = (t | (t<< 4)) ^ x;
   t = (x ^ (x>> 2)) & 0x22488842; x = (t | (t<< 2)) ^ x;
   return x;
}

/* How would you extend Knuth's algorithm to rotating a 64-bit quantity
on a 64-bit machine?
   A simple method is to first swap the two halves of the 64-bit word,
which takes one instruction if you have the rotate shift. Then, rotate
left 15 positions the two halves of the 64-bit word. This takes five
operations:

   x = (x & 0x0001FFFF0001FFFF) << 15 | (x & 0xFFFE0000FFFE0000) >> 17;

Then do the 10-swap, 4-swap, and 2-swap as in Knuth's code but with each
mask m doubled to the form mm. Thus this method adds five operations to
Knuth's code, making it 24 operations for the "exclusive or" version
(rev14a), if you have the rotate shift, or "swap register halves."
   Another way is to develop code similar to Knuth's that first rotates
left 31 positions, and then does a 20-swap, an 8-swap, a 4-swap, and a
2-swap. This adds one stage of swapping, which takes seven operations as
in rev14, or 6 operations as in rev14a. Thus the simple method seems to
be best, in terms of operation count (but not if parallelism in rev14
can be used to advantage).
   First, the simple method. 24 operations if the swap (rotate 32)
counts as 1. */

unsigned long long rev15(unsigned long long x) {
   unsigned long long t;

   x = (x << 32) | (x >> 32);   // Swap register halves.
   x = (x & 0x0001FFFF0001FFFFLL) << 15 | // Rotate left
       (x & 0xFFFE0000FFFE0000LL) >> 17;  // 15.
   t = (x ^ (x >> 10)) & 0x003F801F003F801FLL;
   x = (t | (t << 10)) ^ x;
   t = (x ^ (x >> 4)) & 0x0E0384210E038421LL;
   x = (t | (t << 4)) ^ x;
   t = (x ^ (x >> 2)) & 0x2248884222488842LL;
   x = (t | (t << 2)) ^ x;
   return x;
}

/* Here's the other way. 25 operations if the rotate shift counts as 1. */

unsigned long long rev15a(unsigned long long x) {
   unsigned long long t;

   x = (x << 31) | (x >> 33);   // I.e., shlr(x, 31).
   t = (x ^ (x >> 20)) & 0x00000FFF800007FFLL;
   x = (t |(t << 20)) ^ x;
   t = (x ^ (x >> 8)) & 0x00F8000F80700807LL;
   x = (t |(t << 8)) ^ x;
   t = (x ^ (x >> 4)) & 0x0808708080807008LL;
   x = (t |(t << 4)) ^ x;
   t = (x ^ (x >> 2)) & 0x1111111111111111LL;
   x = (t |(t << 2)) ^ x;
   return x;
}


/* A table lookup method. 13 ops + 4 loads if the loop is strung out. */

unsigned rev16(unsigned x) {
   static unsigned char table[256] = {
   0x00, 0x80, 0x40, 0xC0, 0x20, 0xA0, 0x60, 0xE0, 0x10, 0x90, 0x50, 0xD0, 0x30, 0xB0, 0x70, 0xF0,
   0x08, 0x88, 0x48, 0xC8, 0x28, 0xA8, 0x68, 0xE8, 0x18, 0x98, 0x58, 0xD8, 0x38, 0xB8, 0x78, 0xF8,
   0x04, 0x84, 0x44, 0xC4, 0x24, 0xA4, 0x64, 0xE4, 0x14, 0x94, 0x54, 0xD4, 0x34, 0xB4, 0x74, 0xF4,
   0x0C, 0x8C, 0x4C, 0xCC, 0x2C, 0xAC, 0x6C, 0xEC, 0x1C, 0x9C, 0x5C, 0xDC, 0x3C, 0xBC, 0x7C, 0xFC,
   0x02, 0x82, 0x42, 0xC2, 0x22, 0xA2, 0x62, 0xE2, 0x12, 0x92, 0x52, 0xD2, 0x32, 0xB2, 0x72, 0xF2,
   0x0A, 0x8A, 0x4A, 0xCA, 0x2A, 0xAA, 0x6A, 0xEA, 0x1A, 0x9A, 0x5A, 0xDA, 0x3A, 0xBA, 0x7A, 0xFA,
   0x06, 0x86, 0x46, 0xC6, 0x26, 0xA6, 0x66, 0xE6, 0x16, 0x96, 0x56, 0xD6, 0x36, 0xB6, 0x76, 0xF6,
   0x0E, 0x8E, 0x4E, 0xCE, 0x2E, 0xAE, 0x6E, 0xEE, 0x1E, 0x9E, 0x5E, 0xDE, 0x3E, 0xBE, 0x7E, 0xFE,
   0x01, 0x81, 0x41, 0xC1, 0x21, 0xA1, 0x61, 0xE1, 0x11, 0x91, 0x51, 0xD1, 0x31, 0xB1, 0x71, 0xF1,
   0x09, 0x89, 0x49, 0xC9, 0x29, 0xA9, 0x69, 0xE9, 0x19, 0x99, 0x59, 0xD9, 0x39, 0xB9, 0x79, 0xF9,
   0x05, 0x85, 0x45, 0xC5, 0x25, 0xA5, 0x65, 0xE5, 0x15, 0x95, 0x55, 0xD5, 0x35, 0xB5, 0x75, 0xF5,
   0x0D, 0x8D, 0x4D, 0xCD, 0x2D, 0xAD, 0x6D, 0xED, 0x1D, 0x9D, 0x5D, 0xDD, 0x3D, 0xBD, 0x7D, 0xFD,
   0x03, 0x83, 0x43, 0xC3, 0x23, 0xA3, 0x63, 0xE3, 0x13, 0x93, 0x53, 0xD3, 0x33, 0xB3, 0x73, 0xF3,
   0x0B, 0x8B, 0x4B, 0xCB, 0x2B, 0xAB, 0x6B, 0xEB, 0x1B, 0x9B, 0x5B, 0xDB, 0x3B, 0xBB, 0x7B, 0xFB,
   0x07, 0x87, 0x47, 0xC7, 0x27, 0xA7, 0x67, 0xE7, 0x17, 0x97, 0x57, 0xD7, 0x37, 0xB7, 0x77, 0xF7,
   0x0F, 0x8F, 0x4F, 0xCF, 0x2F, 0xAF, 0x6F, 0xEF, 0x1F, 0x9F, 0x5F, 0xDF, 0x3F, 0xBF, 0x7F, 0xFF};
   int i;
   unsigned r;

   r = 0;
   for (i = 3; i >= 0; i--) {
      r = (r << 8) + table[x & 0xFF];
      x = x >> 8;
   }
   return r;
}

/* ----------------------------- errors ----------------------------- */

int errors, errords;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %08x\n", x, y);
}

void errord(long long x, long long y) {
   errords = errords + 1;
   printf("Error for x = %016llx, got %016llx\n", x, y);
}

/* ------------------------------ main ------------------------------ */

int main() {
   int i, n, m;
   unsigned r;
   unsigned long long rd;

   static unsigned test[] = {0,0, 1,0x80000000, 2,0x40000000,
      3,0xC0000000, 4,0x20000000, 5,0xA0000000, 6,0x60000000,
      7,0xE0000000, 8,0x10000000, 9,0x90000000, 10,0x50000000,
      11,0xD0000000, 12,0x30000000, 13,0xB0000000, 14,0x70000000,
      15,0xF0000000, 16,0x08000000, 59,0xDC000000, 60,0x3C000000,
      61,0xBC000000, 62,0x7C000000, 63,0xFC000000, 64,0x02000000,
      125,0xBE000000, 126,0x7E000000, 127,0xFE000000, 128,0x01000000,
      253,0xBF000000, 254,0x7F000000, 255,0xFF000000, 256,0x00800000,
      257,0x80800000, 509,0xBF800000, 510,0x7F800000, 511,0xFF800000,
      512,0x00400000, 0x55555555,0xAAAAAAAA, 0x01234567,0xE6A2C480,
      0xFFFFFFF1,0x8FFFFFFF, 0xFFFFFFFF,0xFFFFFFFF};

   static unsigned long long testd[] = {0,0, 1,0x8000000000000000LL,
      2,0x4000000000000000LL, 4,0x2000000000000000LL,
      8,0x1000000000000000LL, 16,0x0800000000000000LL,
      0x1111111111111111LL,0x8888888888888888LL,
      0x2222222222222222LL,0x4444444444444444LL,
      0X3333333333333333LL,0XCCCCCCCCCCCCCCCCLL,
      0X5555555555555555LL,0XAAAAAAAAAAAAAAAALL,
      0X6666666666666666LL,0X6666666666666666LL,
      0X7777777777777777LL,0XEEEEEEEEEEEEEEEELL,
      0X9999999999999999LL,0X9999999999999999LL,
      0XBBBBBBBBBBBBBBBBLL,0XDDDDDDDDDDDDDDDDLL,
      0X0123456789ABCDEFLL,0XF7B3D591E6A2C480LL,
      0XFFFFFFFFFFFFFFFFLL,0XFFFFFFFFFFFFFFFFLL};

   n = sizeof(test)/4;

   printf("rev1:\n");
   for (i = 0; i < n; i += 2) {
      r = rev1(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev1(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev2:\n");
   for (i = 0; i < n; i += 2) {
      r = rev2(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev2(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev3:\n");
   for (i = 0; i < n; i += 2) {
      r = rev3(test[i] & 0x0000003F);
      if (r != test[i+1] >> 26) error(test[i], r);
      r = rev3(test[i+1] & 0x0000003F);
      if (r != test[i] >> 26) error(test[i+1], r);}

   printf("rev4:\n");
   for (i = 0; i < n; i += 2) {
      r = rev4(test[i] & 0x0000003F);
      if (r != test[i+1] >> 26) error(test[i], r);
      r = rev4(test[i+1] & 0x0000003F);
      if (r != test[i] >> 26) error(test[i+1], r);}

   printf("rev5:\n");
   for (i = 0; i < n; i += 2) {
      r = rev5(test[i] & 0x000000FF);
      if (r != test[i+1] >> 24) error(test[i], r);
      r = rev5(test[i+1] & 0x000000FF);
      if (r != test[i] >> 24) error(test[i+1], r);}

   printf("rev6:\n");
   for (i = 0; i < n; i += 2) {
      r = rev6(test[i] & 0x000000FF);
      if (r != test[i+1] >> 24) error(test[i], r);
      r = rev6(test[i+1] & 0x000000FF);
      if (r != test[i] >> 24) error(test[i+1], r);}

   printf("rev7:\n");
   for (i = 0; i < n; i += 2) {
      r = rev7(test[i] & 0x000000FF);
      if (r != test[i+1] >> 24) error(test[i], r);
      r = rev7(test[i+1] & 0x000000FF);
      if (r != test[i] >> 24) error(test[i+1], r);}

   printf("rev8:\n");
   for (i = 0; i < n; i += 2) {
      r = rev8(test[i] & 0x000000FF);
      if (r != test[i+1] >> 24) error(test[i], r);
      r = rev8(test[i+1] & 0x000000FF);
      if (r != test[i] >> 24) error(test[i+1], r);}

   printf("rev9:\n");
   for (i = 0; i < n; i += 2) {
      r = rev9(test[i] & 0x000001FF);
      if (r != test[i+1] >> 23) error(test[i], r);
      r = rev9(test[i+1] & 0x000001FF);
      if (r != test[i] >> 23) error(test[i+1], r);}

   printf("rev10:\n");
   for (i = 0; i < n; i += 2) {
      r = rev10(test[i] & 0x000001FF);
      if (r != test[i+1] >> 23) error(test[i], r);
      r = rev10(test[i+1] & 0x000001FF);
      if (r != test[i] >> 23) error(test[i+1], r);}

   printf("rev10a:\n");
   for (i = 0; i < n; i += 2) {
      r = rev10a(test[i] & 0x000001FF);
      if (r != test[i+1] >> 23) error(test[i], r);
      r = rev10a(test[i+1] & 0x000001FF);
      if (r != test[i] >> 23) error(test[i+1], r);}

   printf("rev11:\n");
   for (i = 0; i < n; i += 2) {
      r = rev11(test[i] & 0x0000FFFF);
      if (r != test[i+1] >> 16) error(test[i], r);
      r = rev11(test[i+1] & 0x0000FFFF);
      if (r != test[i] >> 16) error(test[i+1], r);}

   printf("rev11a:\n");
   for (i = 0; i < n; i += 2) {
      r = rev11a(test[i] & 0x0000FFFF);
      if (r != test[i+1] >> 16) error(test[i], r);
      r = rev11a(test[i+1] & 0x0000FFFF);
      if (r != test[i] >> 16) error(test[i+1], r);}

   printf("rev12:\n");
   for (i = 0; i < n; i += 2) {
      r = rev12(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev12(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev12a:\n");
   for (i = 0; i < n; i += 2) {
      r = rev12a(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev12a(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev12b:\n");
   for (i = 0; i < n; i += 2) {
      r = rev12b(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev12b(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev13:\n");
   for (i = 0; i < n; i += 2) {
      r = rev13(test[i]);
      if (r != test[i+1] >> 5) error(test[i], r);
      r = rev13(test[i+1]);
      if (r != test[i] >> 5) error(test[i+1], r);}

   printf("rev14:\n");
   for (i = 0; i < n; i += 2) {
      r = rev14(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev14(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   printf("rev14a:\n");
   for (i = 0; i < n; i += 2) {
      r = rev14a(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev14a(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   m = sizeof(testd)/8;
   printf("rev15:\n");
   for (i = 0; i < m; i += 2) {
      rd = rev15(testd[i]);
      if (rd != testd[i+1]) errord(testd[i], rd);
      rd = rev15(testd[i+1]);
      if (rd != testd[i]) errord(testd[i+1], rd);
   }

   printf("rev15a:\n");
   for (i = 0; i < m; i += 2) {
      rd = rev15a(testd[i]);
      if (rd != testd[i+1]) errord(testd[i], rd);
      rd = rev15a(testd[i+1]);
      if (rd != testd[i]) errord(testd[i+1], rd);
   }

   printf("rev16:\n");
   for (i = 0; i < n; i += 2) {
      r = rev16(test[i]);
      if (r != test[i+1]) error(test[i], r);
      r = rev16(test[i+1]);
      if (r != test[i]) error(test[i+1], r);}

   if (errors == 0)
      printf("All functions passed all %d 32-bit cases.\n", sizeof(test)/8);
   if (errords == 0)
      printf("All functions passed all %d 64-bit cases.\n", sizeof(testd)/16);
   return errors + errords;
}
