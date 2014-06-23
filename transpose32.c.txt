// Program for computing the transpose of a 32x32 bit matrix.
// Max line length is 57, to fit in hacker.book.
// This has been tested on both AIX/xlc and Windows/gcc, and
// so is believed to be independent of endian mode.
#include <stdio.h>
// #include <stdlib.h>     // To define "exit", req'd by XLC.

/* Below is the original version from Guy Steele. */

void transpose32a(unsigned long a[32]) {
   int j, k;
   unsigned long m, t;

   for (j = 16, m = 0x0000FFFF; j; j >>= 1, m ^= m << j) {
      for (k = 0; k < 32; k = ((k | j) + 1) & ~j) {
         t = (a[k] ^ (a[k|j] >> j)) & m;
         a[k] ^= t;
         a[k|j] ^= (t << j);
      }
   }
}

/* Below is essentially the same code, but modified to avoid certain C
expressions out of sympathy for readers who are not very familiar with
C. Also modified to use k + j rather than k | j, because + seems more
natural for this program.  None of these changes affects the number of
instructions executed. */

// ------------------------------ cut ----------------------------------
void transpose32b(unsigned A[32]) {
   int j, k;
   unsigned m, t;

   m = 0x0000FFFF;
   for (j = 16; j != 0; j = j >> 1, m = m ^ (m << j)) {
      for (k = 0; k < 32; k = (k + j + 1) & ~j) {
         t = (A[k] ^ (A[k+j] >> j)) & m;
         A[k] = A[k] ^ t;
         A[k+j] = A[k+j] ^ (t << j);
      }
   }
}
// ---------------------------- end cut --------------------------------

/* Straight-line version of transpose32a & b. */

#define swap(a0, a1, j, m) t = (a0 ^ (a1 >> j)) & m; \
                           a0 = a0 ^ t; \
                           a1 = a1 ^ (t << j);

void transpose32c(unsigned A[32], unsigned B[32]) {
   unsigned m, t;
   unsigned a0, a1, a2, a3, a4, a5, a6, a7,
            a8, a9, a10, a11, a12, a13, a14, a15,
            a16, a17, a18, a19, a20, a21, a22, a23,
            a24, a25, a26, a27, a28, a29, a30, a31;

   a0  = A[ 0];  a1  = A[ 1];  a2  = A[ 2];  a3  = A[ 3];
   a4  = A[ 4];  a5  = A[ 5];  a6  = A[ 6];  a7  = A[ 7];
   a8  = A[ 8];  a9  = A[ 9];  a10 = A[10];  a11 = A[11];
   a12 = A[12];  a13 = A[13];  a14 = A[14];  a15 = A[15];
   a16 = A[16];  a17 = A[17];  a18 = A[18];  a19 = A[19];
   a20 = A[20];  a21 = A[21];  a22 = A[22];  a23 = A[23];
   a24 = A[24];  a25 = A[25];  a26 = A[26];  a27 = A[27];
   a28 = A[28];  a29 = A[29];  a30 = A[30];  a31 = A[31];

   m = 0x0000FFFF;
   swap(a0,  a16, 16, m)
   swap(a1,  a17, 16, m)
   swap(a2,  a18, 16, m)
   swap(a3,  a19, 16, m)
   swap(a4,  a20, 16, m)
   swap(a5,  a21, 16, m)
   swap(a6,  a22, 16, m)
   swap(a7,  a23, 16, m)
   swap(a8,  a24, 16, m)
   swap(a9,  a25, 16, m)
   swap(a10, a26, 16, m)
   swap(a11, a27, 16, m)
   swap(a12, a28, 16, m)
   swap(a13, a29, 16, m)
   swap(a14, a30, 16, m)
   swap(a15, a31, 16, m)
   m = 0x00FF00FF;
   swap(a0,  a8,   8, m)
   swap(a1,  a9,   8, m)
   swap(a2,  a10,  8, m)
   swap(a3,  a11,  8, m)
   swap(a4,  a12,  8, m)
   swap(a5,  a13,  8, m)
   swap(a6,  a14,  8, m)
   swap(a7,  a15,  8, m)
   swap(a16, a24,  8, m)
   swap(a17, a25,  8, m)
   swap(a18, a26,  8, m)
   swap(a19, a27,  8, m)
   swap(a20, a28,  8, m)
   swap(a21, a29,  8, m)
   swap(a22, a30,  8, m)
   swap(a23, a31,  8, m)
   m = 0x0F0F0F0F;
   swap(a0,  a4,   4, m)
   swap(a1,  a5,   4, m)
   swap(a2,  a6,   4, m)
   swap(a3,  a7,   4, m)
   swap(a8,  a12,  4, m)
   swap(a9,  a13,  4, m)
   swap(a10, a14,  4, m)
   swap(a11, a15,  4, m)
   swap(a16, a20,  4, m)
   swap(a17, a21,  4, m)
   swap(a18, a22,  4, m)
   swap(a19, a23,  4, m)
   swap(a24, a28,  4, m)
   swap(a25, a29,  4, m)
   swap(a26, a30,  4, m)
   swap(a27, a31,  4, m)
   m = 0x33333333;
   swap(a0,  a2,   2, m)
   swap(a1,  a3,   2, m)
   swap(a4,  a6,   2, m)
   swap(a5,  a7,   2, m)
   swap(a8,  a10,  2, m)
   swap(a9,  a11,  2, m)
   swap(a12, a14,  2, m)
   swap(a13, a15,  2, m)
   swap(a16, a18,  2, m)
   swap(a17, a19,  2, m)
   swap(a20, a22,  2, m)
   swap(a21, a23,  2, m)
   swap(a24, a26,  2, m)
   swap(a25, a27,  2, m)
   swap(a28, a30,  2, m)
   swap(a29, a31,  2, m)
   m = 0x55555555;
   swap(a0,  a1,   1, m)
   swap(a2,  a3,   1, m)
   swap(a4,  a5,   1, m)
   swap(a6,  a7,   1, m)
   swap(a8,  a9,   1, m)
   swap(a10, a11,  1, m)
   swap(a12, a13,  1, m)
   swap(a14, a15,  1, m)
   swap(a16, a17,  1, m)
   swap(a18, a19,  1, m)
   swap(a20, a21,  1, m)
   swap(a22, a23,  1, m)
   swap(a24, a25,  1, m)
   swap(a26, a27,  1, m)
   swap(a28, a29,  1, m)
   swap(a30, a31,  1, m)

   B[ 0] = a0;   B[ 1] = a1;   B[ 2] = a2;   B[ 3] = a3;
   B[ 4] = a4;   B[ 5] = a5;   B[ 6] = a6;   B[ 7] = a7;
   B[ 8] = a8;   B[ 9] = a9;   B[10] = a10;  B[11] = a11;
   B[12] = a12;  B[13] = a13;  B[14] = a14;  B[15] = a15;
   B[16] = a16;  B[17] = a17;  B[18] = a18;  B[19] = a19;
   B[20] = a20;  B[21] = a21;  B[22] = a22;  B[23] = a23;
   B[24] = a24;  B[25] = a25;  B[26] = a26;  B[27] = a27;
   B[28] = a28;  B[29] = a29;  B[30] = a30;  B[31] = a31;
}

/* Copied from GLS's note.  This is the "three shearing transformations"
method.  The code below takes 1280 ops to do the bit rearrangements
(i.e., not counting loop control, loads, stores, and indexing).  Not
competitive with the other methods.  */

#define rotateright(x, k) ((x) >> (k)) | ((x) << (32 - (k)))
#define rotateleft(x, k)  ((x) << (k)) | ((x) >> (32 - (k)))

void transpose32d(unsigned a[]) {
  int j, k, q;
  unsigned m, t, u;
  for (k = 0; k < 32; k++) a[k] = rotateright(a[k], k);
  for (j = 16, m = 0xFFFF0000; j; j >>= 1, m ^= m >> j) {
    for (k = 0; k < j; k++) {
      t = a[k] & m;
      a[k] = a[k] ^ t;
      for (q = k+j; q < 32; q += j) {
                 u = a[q] & m;
                 a[q] = a[q] ^ u ^ t;
                 t = u;
      }
      a[k] = a[k] ^ t;
    }
  }
  for (k = 0; k < 32; k++) a[k] = rotateleft(a[k], 31-k);
  for (k = 0; k < 16; k++) {
    t = a[k];
    a[k] = a[31-k];
    a[31-k] = t;
  }
}

int errors;
void error(int i, unsigned res) {
   errors = errors + 1;
   printf("Error for i = %d, got %08x\n", i, res);
}

int main() {
   int i;
   static unsigned A[32] = {            // Test matrix.
      0x01020304, 0x05060708, 0x090A0B0C, 0x0D0E0F00,
      0xF0E0D0C0, 0xB0A09080, 0x70605040, 0x30201000,
      0x00000000, 0x01010101, 0x02020202, 0x04040404,
      0x08080808, 0x10101010, 0x20202020, 0x40404040,

      0x80808080, 0xFFFFFFFF, 0xFEFEFEFE, 0xFDFDFDFD,
      0xFBFBFBFB, 0xF7F7F7F7, 0xEFEFEFEF, 0xDFDFDFDF,
      0xBFBFBFBF, 0x7F7F7F7F, 0x80000001, 0xC0000003,
      0xE0000007, 0xF000000F, 0xF800001F, 0xFC00003F};

   const unsigned AT[32] = {            // A-transpose.
      0x0C00FFBF, 0x0A017F5F, 0x0F027ECF, 0x0F047DC7,
      0x30087BC3, 0x501077C1, 0x00206FC0, 0xF0405FC0,
      0x0C00FF80, 0x0A017F40, 0x0F027EC0, 0x00047DC0,
      0x30087BC0, 0x501077C0, 0xF0206FC0, 0x00405FC0,

      0x0C00FF80, 0x0A017F40, 0x00027EC0, 0x0F047DC0,
      0x30087BC0, 0x501077C0, 0xF0206FC0, 0xF0405FC0,
      0x0C00FF80, 0x0A017F40, 0x00027EC1, 0x00047DC3,
      0x60087BC7, 0xA01077CF, 0x00206FDF, 0x00405FFF};

   unsigned trial[32];                  // Trial area.

   for (i = 0; i < 32; i++)
      trial[i] = A[i];

   printf("transpose32a, forward test:\n");
   transpose32a((unsigned long *)trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != AT[i]) error(i, trial[i]);

   printf("transpose32a, reverse test:\n");
   transpose32a((unsigned long *)trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != A[i]) error(i, trial[i]);

   printf("transpose32b, forward test:\n");
   transpose32b(trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != AT[i]) error(i, trial[i]);

   printf("transpose32b, reverse test:\n");
   transpose32b(trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != A[i]) error(i, trial[i]);

   printf("transpose32c, forward test:\n");
   transpose32c(trial, trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != AT[i]) error(i, trial[i]);

   printf("transpose32c, reverse test:\n");
   transpose32c(trial, trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != A[i]) error(i, trial[i]);

   printf("transpose32d, forward test:\n");
   transpose32d((unsigned *)trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != AT[i]) error(i, trial[i]);

   printf("transpose32d, reverse test:\n");
   transpose32d((unsigned *)trial);
   for (i = 0; i < 32; i++)
      if (trial[i] != A[i]) error(i, trial[i]);
}
