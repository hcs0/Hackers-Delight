// Computes the m+n-halfword product of n halfwords x m halfwords, signed.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

// w[0], u[0], and v[0] contain the LEAST significant halfwords.
// (The words are in little-endian order).
// This is Knuth's Algorithm M from [Knuth Vol. 2 Third edition (1998)]
// section 4.3.1, altered for signed numbers.  Picture is:
//                   u[m-1] ... u[1] u[0]
//                 x v[n-1] ... v[1] v[0]
//                   --------------------
//        w[m+n-1] ............ w[1] w[0]

void mulmns(unsigned short w[], unsigned short u[],
   unsigned short v[], int m, int n) {

   unsigned int k, t, b;
   int i, j;

   for (i = 0; i < m; i++)
      w[i] = 0;

   for (j = 0; j < n; j++) {
      k = 0;
      for (i = 0; i < m; i++) {
         t = u[i]*v[j] + w[i + j] + k;
         w[i + j] = t;          // (I.e., t & 0xFFFF).
         k = t >> 16;
      }
      w[j + m] = k;
   }

   // Now w[] has the unsigned product.  Correct by
   // subtracting v*2**16m if u < 0, and
   // subtracting u*2**16n if v < 0.

   if ((short)u[m - 1] < 0) {
      b = 0;                    // Initialize borrow.
      for (j = 0; j < n; j++) {
         t = w[j + m] - v[j] - b;
         w[j + m] = t;
         b = t >> 31;
      }
   }
   if ((short)v[n - 1] < 0) {
      b = 0;
      for (i = 0; i < m; i++) {
         t = w[i + n] - u[i] - b;
         w[i + n] = t;
         b = t >> 31;
      }
   }
   return;
}

int errors;

void check(unsigned short result[], unsigned short u[],
   unsigned short v[], int m, int n, unsigned short correct[]) {
   int i, j;

   for (i = 0; i < m + n; i++) {
      if (correct[i] != result[i]) {
         errors = errors + 1;
         printf("Error, m = %d, n = %d, u = ", m, n);
         for (j = 0; j < m; j++) printf(" %04x", u[j]);
         printf(" v =");
         for (j = 0; j < n; j++) printf(" %04x", v[j]);
         printf("\nShould get:");
         for (j = 0; j < n+m; j++) printf(" %04x", correct[j]);
         printf("\n       Got:");
         for (j = 0; j < n+m; j++) printf(" %04x", result[j]);
         printf("\n");
         break;
      }
   }
}

int main() {
   static unsigned short test[] = {
   //  m, n, u ..., v ..., result.
      1, 1, 7, 3,                  21,0,
      1, 1,      2, 0xFFFF,        0xFFFE,0xFFFF, // (2)*(-1) = -2.
      1, 1, 0xFFFF, 0xFFFF,        1,0,           // (-1)*(-1) = 1.
      1, 2, 7, 5, 6,               35,42,0,
      1, 2, 65535, 65535, 65535,   1,0,0,         // (-1)*(-1) = 1.
      1, 2, 65000, 63000, 64000,   0xBDC0,0x8DFC,0x000C,
      1, 3, 65535, 31000, 32000, 33000, 0x86E8,0x82FF,0x7F17,0x0000,
      2, 3, 400, 300, 500, 100, 200, 0x0D40,0xE633,0xADB2,0xEA61,0,
      2, 3, 400, 65535, 500, 100, 65534, 0x0D40,0x9A4F,0xFC7C,0x0001,0x0000,
      2, 3,   800, 60000,   7,   5,   3, 0x15E0,0x7840,0x9D3F,0xBF1F,0xFFFF,
      4, 4, 65535, 65535, 65535, 65535, 65535, 65535, 65535, 65535,
                1,     0,     0,     0,     0,     0,     0,     0,
   };
   int i, n, m, ncases;
   unsigned short result[10];
   unsigned short *u, *v;

   printf("mulmns:\n");
   i = 0;
   ncases = 0;
   while (i < sizeof(test)/2) {
      m = test[i];
      n = test[i+1];
      u = &test[i+2];
      v = &test[i+2+m];
      mulmns(result, u, v, m, n);
      check (result, u, v, m, n, &test[i+2+m+n]);
      mulmns(result, v, u, n, m);       // Interchange operands.
      check (result, v, u, n, m, &test[i+2+m+n]);
      i = i + 2 + 2*(m + n);
      ncases = ncases + 1;
   }

   if (errors == 0)
      printf("Passed all %d cases.\n", ncases);
}
