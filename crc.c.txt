/* This is a collection of programs that computes the CRC-32 checksum.
Probably only the first three, and maybe the fourth (CRC32d) are of any
interest. The others are just experiments that did not result in
anything useful. More comments are at the bottom of this file. */

#include <stdio.h>
#include <stdlib.h>

// ---------------------------- reverse --------------------------------

// Reverses (reflects) bits in a 32-bit word.
unsigned reverse(unsigned x) {
   x = ((x & 0x55555555) <<  1) | ((x >>  1) & 0x55555555);
   x = ((x & 0x33333333) <<  2) | ((x >>  2) & 0x33333333);
   x = ((x & 0x0F0F0F0F) <<  4) | ((x >>  4) & 0x0F0F0F0F);
   x = (x << 24) | ((x & 0xFF00) << 8) |
       ((x >> 8) & 0xFF00) | (x >> 24);
   return x;
}

// ----------------------------- crc32a --------------------------------

/* This is the basic CRC algorithm with no optimizations. It follows the
logic circuit as closely as possible. */

unsigned int crc32a(unsigned char *message) {
   int i, j;
   unsigned int byte, crc;

   i = 0;
   crc = 0xFFFFFFFF;
   while (message[i] != 0) {
      byte = message[i];            // Get next byte.
      byte = reverse(byte);         // 32-bit reversal.
      for (j = 0; j <= 7; j++) {    // Do eight times.
         if ((int)(crc ^ byte) < 0)
              crc = (crc << 1) ^ 0x04C11DB7;
         else crc = crc << 1;
         byte = byte << 1;          // Ready next msg bit.
      }
      i = i + 1;
   }
   return reverse(~crc);
}

// ----------------------------- crc32b --------------------------------

/* This is the basic CRC-32 calculation with some optimization but no
table lookup. The the byte reversal is avoided by shifting the crc reg
right instead of left and by using a reversed 32-bit word to represent
the polynomial.
   When compiled to Cyclops with GCC, this function executes in 8 + 72n
instructions, where n is the number of bytes in the input message. It
should be doable in 4 + 61n instructions.
   If the inner loop is strung out (approx. 5*8 = 40 instructions),
it would take about 6 + 46n instructions. */

unsigned int crc32b(unsigned char *message) {
   int i, j;
   unsigned int byte, crc, mask;

   i = 0;
   crc = 0xFFFFFFFF;
   while (message[i] != 0) {
      byte = message[i];            // Get next byte.
      crc = crc ^ byte;
      for (j = 7; j >= 0; j--) {    // Do eight times.
         mask = -(crc & 1);
         crc = (crc >> 1) ^ (0xEDB88320 & mask);
      }
      i = i + 1;
   }
   return ~crc;
}

// ----------------------------- crc32c --------------------------------

/* This is derived from crc32b but does table lookup. First the table
itself is calculated, if it has not yet been set up.
Not counting the table setup (which would probably be a separate
function), when compiled to Cyclops with GCC, this function executes in
7 + 13n instructions, where n is the number of bytes in the input
message. It should be doable in 4 + 9n instructions. In any case, two
of the 13 or 9 instrucions are load byte.
   This is Figure 14-7 in the text. */

unsigned int crc32c(unsigned char *message) {
   int i, j;
   unsigned int byte, crc, mask;
   static unsigned int table[256];

   /* Set up the table, if necessary. */

   if (table[1] == 0) {
      for (byte = 0; byte <= 255; byte++) {
         crc = byte;
         for (j = 7; j >= 0; j--) {    // Do eight times.
            mask = -(crc & 1);
            crc = (crc >> 1) ^ (0xEDB88320 & mask);
         }
         table[byte] = crc;
      }
   }

   /* Through with table setup, now calculate the CRC. */

   i = 0;
   crc = 0xFFFFFFFF;
   while ((byte = message[i]) != 0) {
      crc = (crc >> 8) ^ table[(crc ^ byte) & 0xFF];
      i = i + 1;
   }
   return ~crc;
}

// ---------------------------- crc32cx --------------------------------

/* This is crc32b modified to load the message a fullword at a time.
It assumes the message is word aligned and consists of an integral
number of words before the 0-byte that marks the end of the message.
   This works only on a little-endian machine.
   Not counting the table setup (which would probably be a separate
function), this function should be doable in 3 + 22w instructions, where
w is the number of fullwords in the input message. This is equivalent to
3 + 5.5n instructions, where n is the number of bytes. 1.25 of those 5.5
instructions are loads.
   This is Exercise 1 in the text. C.f. Christopher Dannemiller,
who got it from Linux Source base,
www.gelato.unsw.edu.au/lxr/source/lib/crc32.c, lines 105-111. */

unsigned int crc32cx(unsigned char *message) {
   int j;
   unsigned int byte, crc, mask, word;
   static unsigned int table[256];

   /* Set up the table, if necessary. */

   if (table[1] == 0) {
      for (byte = 0; byte <= 255; byte++) {
         crc = byte;
         for (j = 7; j >= 0; j--) {    // Do eight times.
            mask = -(crc & 1);
            crc = (crc >> 1) ^ (0xEDB88320 & mask);
         }
         table[byte] = crc;
      }
   }

   /* Through with table setup, now calculate the CRC. */

   crc = 0xFFFFFFFF;
   while (((word = *(unsigned int *)message) & 0xFF) != 0) {
      crc = crc ^ word;
      crc = (crc >> 8) ^ table[crc & 0xFF];
      crc = (crc >> 8) ^ table[crc & 0xFF];
      crc = (crc >> 8) ^ table[crc & 0xFF];
      crc = (crc >> 8) ^ table[crc & 0xFF];
      message = message + 4;
   }
   return ~crc;
}

// ----------------------------- crc32d --------------------------------

/* This is like crc32c (does table lookup) but it processes two bytes at
a time, except for a possible odd byte at the beginning or end of the
message. The table size is 65536 words.
   Not counting the table setup (which would probably be a separate
function), when compiled to Cyclops with GCC, this function executes in
14 + 14n instructions, where n is the number of halfwords in the input
message. This assumes there are no odd bytes at either end.
   Note: When accessing the table for a single byte b, the entry to use
is b << 8. I.e., if the byte is the letter 'a', the entry to use is that
with index 0x6100, not 0x0061. */

unsigned int crc32d(unsigned char *message) {
   int i, j;
   unsigned int byte, half, crc, mask;
   static unsigned int table[65536];

   if (table[1] == 0) {         // If table has not yet
                                // been set up:
      for (half = 0; half <= 65535; half++) {
         crc = half;
         for (j = 15; j >= 0; j--) {    // Do 15 times.
            mask = -(crc & 1);
            crc = (crc >> 1) ^ (0xEDB88320 & mask);
         }
         table[half] = crc;
      }
   }

   crc = 0xFFFFFFFF;

   // First, if message is aligned on an odd address,
   // take care of the first byte.

   i = (int)&message[0] & 1;    // Start of halfwords.
   if (i) {                     // If i == 1:
      byte = message[0];
      if (byte == 0) return 0;  // If null message.
      crc = (crc >> 8) ^ table[(byte ^ 0xFF) << 8];
   }

   // Next process the message two bytes at a time as long
   // as both bytes are nonzero.

   while (1) {
      half = *(unsigned short *)(&message[i]);
      if (half <= 0xFF || (half & 0xFF) == 0) break;
      crc = (crc >> 16) ^ table[(crc ^ half) & 0xFFFF];
      i = i + 2;
   }

   // Lastly, process the odd byte at the end, if any.
   // "half" is of the form 00xx, xx00, or 0000.

   if (half & 0xFF) {
      crc = (crc >> 8) ^ table[((crc^half) & 0xFF) << 8];
   }

   return ~crc;
}

// ----------------------------- crc32e --------------------------------

/* This is sort of like the table lookup version (crc32c), but using
a 16-way switch statement instead.
   When compiled to Cyclops with GCC, this function executes in 6 + 38n
instructions, where n is the number of bytes in the input message. The
38 instructions per byte include 3 loads and 5 branches (not good). It
is actually 6 branches if you count the unnecessary one that GCC
generates because it isn't smart enough to know that the switch argument
cannot exceed 15. */

unsigned int crc32e(unsigned char *message) {
   int i, j;
   unsigned int byte, crc, c;
   const unsigned int g0 = 0xEDB88320, g1 = g0 >> 1,
                      g2 = g0 >> 2,    g3 = g0 >> 3;

   i = 0;
   crc = 0xFFFFFFFF;
   while ((byte = message[i]) != 0) {   // Get next byte.
      crc = crc ^ byte;
      for (j = 1; j >= 0; j--) {        // Do two times.
         switch(crc & 0xF) {
         case  0: c = 0;                  break;
         case  1: c =                g3;  break;
         case  2: c =           g2;       break;
         case  3: c =           g2 ^ g3;  break;
         case  4: c =      g1;            break;
         case  5: c =      g1 ^      g3;  break;
         case  6: c =      g1 ^ g2;       break;
         case  7: c =      g1 ^ g2 ^ g3;  break;
         case  8: c = g0;                 break;
         case  9: c = g0 ^           g3;  break;
         case 10: c = g0 ^      g2;       break;
         case 11: c = g0 ^      g2 ^ g3;  break;
         case 12: c = g0 ^ g1;            break;
         case 13: c = g0 ^ g1 ^      g3;  break;
         case 14: c = g0 ^ g1 ^ g2;       break;
         case 15: c = g0 ^ g1 ^ g2 ^ g3;  break;
         }
         crc = (crc >> 4) ^ c;
      }
      i = i + 1;
   }
   return ~crc;
}

// ----------------------------- crc32f --------------------------------

/* This is sort of like the table lookup version (crc32c), but using
a 256-way switch statement instead.
   The expressions for g1, g2, ..., g7 are determined by examining what
the CRC-32 algorithm does to a byte of value 1, 2, 4, 8, 16, 32, 64, and
128, respectively. g6 and g7 are complicated because the rightmost 1-bit
in g0 enters the picture.
   We rely on the compiler to evaluate, at compile time, all the
expressions involving the g's. They are the table values used in
function crc32c above (i.e., g7 = table[1], g6 = table[2], g5 =
table[4], etc.).
   This idea of using a switch statement is a dumb idea if a compiler is
used, because the compiler (GCC anyway) implements the switch statement
with a 256-word label table. Thus the program still has the load from a
table, and it is larger than crc32c by the three words of instructions
used at each case statement (two instructions to load the constant, plus
a branch). However, since each case statement has the same amount of
code (three words), the label table could be avoided if the program were
coded in assembly language. But it would still have poor I-cache
performance.
   At any rate, when compiled to Cyclops with GCC, this function
executes 6 + 19n instructions, where n is the number of bytes in the
input message. The 19 includes 2 loads and 3 branches (per byte), not
counting the one GCC generates to check that the switch argument doesn't
exceed 255 (it can't exceed 255). */

unsigned int crc32f(unsigned char *message) {
   int i;
   unsigned int byte, crc, c;
   const unsigned int g0 = 0xEDB88320;
   const unsigned int g1 = g0>>1, g2 = g0>>2, g3 = g0>>3,
      g4 = g0>>4, g5 = g0>>5, g6 = (g0>>6)^g0,
      g7 = ((g0>>6)^g0)>>1;

   i = 0;
   crc = 0xFFFFFFFF;
   while ((byte = message[i]) != 0) {   // Get next byte.
      crc = crc ^ byte;
      switch(crc & 0xFF) {
         case   0: c = 0;                        break;
         case   1: c =                      g7;  break;
         case   2: c =                   g6;     break;
         case   3: c =                   g6^g7;  break;
         case   4: c =                g5;        break;
         case   5: c =                g5^   g7;  break;
         case   6: c =                g5^g6;     break;
         case   7: c =                g5^g6^g7;  break;
         case   8: c =             g4;           break;
         case   9: c =             g4^      g7;  break;
         case  10: c =             g4^   g6;     break;
         case  11: c =             g4^   g6^g7;  break;
         case  12: c =             g4^g5;        break;
         case  13: c =             g4^g5^   g7;  break;
         case  14: c =             g4^g5^g6;     break;
         case  15: c =             g4^g5^g6^g7;  break;

         case  16: c =          g3;              break;
         case  17: c =          g3^         g7;  break;
         case  18: c =          g3^      g6;     break;
         case  19: c =          g3^      g6^g7;  break;
         case  20: c =          g3^   g5;        break;
         case  21: c =          g3^   g5^   g7;  break;
         case  22: c =          g3^   g5^g6;     break;
         case  23: c =          g3^   g5^g6^g7;  break;
         case  24: c =          g3^g4;           break;
         case  25: c =          g3^g4^      g7;  break;
         case  26: c =          g3^g4^   g6;     break;
         case  27: c =          g3^g4^   g6^g7;  break;
         case  28: c =          g3^g4^g5;        break;
         case  29: c =          g3^g4^g5^   g7;  break;
         case  30: c =          g3^g4^g5^g6;     break;
         case  31: c =          g3^g4^g5^g6^g7;  break;

         case  32: c =       g2;                 break;
         case  33: c =       g2^            g7;  break;
         case  34: c =       g2^         g6;     break;
         case  35: c =       g2^         g6^g7;  break;
         case  36: c =       g2^      g5;        break;
         case  37: c =       g2^      g5^   g7;  break;
         case  38: c =       g2^      g5^g6;     break;
         case  39: c =       g2^      g5^g6^g7;  break;
         case  40: c =       g2^   g4;           break;
         case  41: c =       g2^   g4^      g7;  break;
         case  42: c =       g2^   g4^   g6;     break;
         case  43: c =       g2^   g4^   g6^g7;  break;
         case  44: c =       g2^   g4^g5;        break;
         case  45: c =       g2^   g4^g5^   g7;  break;
         case  46: c =       g2^   g4^g5^g6;     break;
         case  47: c =       g2^   g4^g5^g6^g7;  break;

         case  48: c =       g2^g3;              break;
         case  49: c =       g2^g3^         g7;  break;
         case  50: c =       g2^g3^      g6;     break;
         case  51: c =       g2^g3^      g6^g7;  break;
         case  52: c =       g2^g3^   g5;        break;
         case  53: c =       g2^g3^   g5^   g7;  break;
         case  54: c =       g2^g3^   g5^g6;     break;
         case  55: c =       g2^g3^   g5^g6^g7;  break;
         case  56: c =       g2^g3^g4;           break;
         case  57: c =       g2^g3^g4^      g7;  break;
         case  58: c =       g2^g3^g4^   g6;     break;
         case  59: c =       g2^g3^g4^   g6^g7;  break;
         case  60: c =       g2^g3^g4^g5;        break;
         case  61: c =       g2^g3^g4^g5^   g7;  break;
         case  62: c =       g2^g3^g4^g5^g6;     break;
         case  63: c =       g2^g3^g4^g5^g6^g7;  break;

         case  64: c =    g1;                    break;
         case  65: c =    g1^               g7;  break;
         case  66: c =    g1^            g6;     break;
         case  67: c =    g1^            g6^g7;  break;
         case  68: c =    g1^         g5;        break;
         case  69: c =    g1^         g5^   g7;  break;
         case  70: c =    g1^         g5^g6;     break;
         case  71: c =    g1^         g5^g6^g7;  break;
         case  72: c =    g1^      g4;           break;
         case  73: c =    g1^      g4^      g7;  break;
         case  74: c =    g1^      g4^   g6;     break;
         case  75: c =    g1^      g4^   g6^g7;  break;
         case  76: c =    g1^      g4^g5;        break;
         case  77: c =    g1^      g4^g5^   g7;  break;
         case  78: c =    g1^      g4^g5^g6;     break;
         case  79: c =    g1^      g4^g5^g6^g7;  break;

         case  80: c =    g1^   g3;              break;
         case  81: c =    g1^   g3^         g7;  break;
         case  82: c =    g1^   g3^      g6;     break;
         case  83: c =    g1^   g3^      g6^g7;  break;
         case  84: c =    g1^   g3^   g5;        break;
         case  85: c =    g1^   g3^   g5^   g7;  break;
         case  86: c =    g1^   g3^   g5^g6;     break;
         case  87: c =    g1^   g3^   g5^g6^g7;  break;
         case  88: c =    g1^   g3^g4;           break;
         case  89: c =    g1^   g3^g4^      g7;  break;
         case  90: c =    g1^   g3^g4^   g6;     break;
         case  91: c =    g1^   g3^g4^   g6^g7;  break;
         case  92: c =    g1^   g3^g4^g5;        break;
         case  93: c =    g1^   g3^g4^g5^   g7;  break;
         case  94: c =    g1^   g3^g4^g5^g6;     break;
         case  95: c =    g1^   g3^g4^g5^g6^g7;  break;

         case  96: c =    g1^g2;                 break;
         case  97: c =    g1^g2^            g7;  break;
         case  98: c =    g1^g2^         g6;     break;
         case  99: c =    g1^g2^         g6^g7;  break;
         case 100: c =    g1^g2^      g5;        break;
         case 101: c =    g1^g2^      g5^   g7;  break;
         case 102: c =    g1^g2^      g5^g6;     break;
         case 103: c =    g1^g2^      g5^g6^g7;  break;
         case 104: c =    g1^g2^   g4;           break;
         case 105: c =    g1^g2^   g4^      g7;  break;
         case 106: c =    g1^g2^   g4^   g6;     break;
         case 107: c =    g1^g2^   g4^   g6^g7;  break;
         case 108: c =    g1^g2^   g4^g5;        break;
         case 109: c =    g1^g2^   g4^g5^   g7;  break;
         case 110: c =    g1^g2^   g4^g5^g6;     break;
         case 111: c =    g1^g2^   g4^g5^g6^g7;  break;

         case 112: c =    g1^g2^g3;              break;
         case 113: c =    g1^g2^g3^         g7;  break;
         case 114: c =    g1^g2^g3^      g6;     break;
         case 115: c =    g1^g2^g3^      g6^g7;  break;
         case 116: c =    g1^g2^g3^   g5;        break;
         case 117: c =    g1^g2^g3^   g5^   g7;  break;
         case 118: c =    g1^g2^g3^   g5^g6;     break;
         case 119: c =    g1^g2^g3^   g5^g6^g7;  break;
         case 120: c =    g1^g2^g3^g4;           break;
         case 121: c =    g1^g2^g3^g4^      g7;  break;
         case 122: c =    g1^g2^g3^g4^   g6;     break;
         case 123: c =    g1^g2^g3^g4^   g6^g7;  break;
         case 124: c =    g1^g2^g3^g4^g5;        break;
         case 125: c =    g1^g2^g3^g4^g5^   g7;  break;
         case 126: c =    g1^g2^g3^g4^g5^g6;     break;
         case 127: c =    g1^g2^g3^g4^g5^g6^g7;  break;

         case 128: c = g0;                       break;
         case 129: c = g0^                  g7;  break;
         case 130: c = g0^               g6;     break;
         case 131: c = g0^               g6^g7;  break;
         case 132: c = g0^            g5;        break;
         case 133: c = g0^            g5^   g7;  break;
         case 134: c = g0^            g5^g6;     break;
         case 135: c = g0^            g5^g6^g7;  break;
         case 136: c = g0^         g4;           break;
         case 137: c = g0^         g4^      g7;  break;
         case 138: c = g0^         g4^   g6;     break;
         case 139: c = g0^         g4^   g6^g7;  break;
         case 140: c = g0^         g4^g5;        break;
         case 141: c = g0^         g4^g5^   g7;  break;
         case 142: c = g0^         g4^g5^g6;     break;
         case 143: c = g0^         g4^g5^g6^g7;  break;

         case 144: c = g0^      g3;              break;
         case 145: c = g0^      g3^         g7;  break;
         case 146: c = g0^      g3^      g6;     break;
         case 147: c = g0^      g3^      g6^g7;  break;
         case 148: c = g0^      g3^   g5;        break;
         case 149: c = g0^      g3^   g5^   g7;  break;
         case 150: c = g0^      g3^   g5^g6;     break;
         case 151: c = g0^      g3^   g5^g6^g7;  break;
         case 152: c = g0^      g3^g4;           break;
         case 153: c = g0^      g3^g4^      g7;  break;
         case 154: c = g0^      g3^g4^   g6;     break;
         case 155: c = g0^      g3^g4^   g6^g7;  break;
         case 156: c = g0^      g3^g4^g5;        break;
         case 157: c = g0^      g3^g4^g5^   g7;  break;
         case 158: c = g0^      g3^g4^g5^g6;     break;
         case 159: c = g0^      g3^g4^g5^g6^g7;  break;

         case 160: c = g0^   g2;                 break;
         case 161: c = g0^   g2^            g7;  break;
         case 162: c = g0^   g2^         g6;     break;
         case 163: c = g0^   g2^         g6^g7;  break;
         case 164: c = g0^   g2^      g5;        break;
         case 165: c = g0^   g2^      g5^   g7;  break;
         case 166: c = g0^   g2^      g5^g6;     break;
         case 167: c = g0^   g2^      g5^g6^g7;  break;
         case 168: c = g0^   g2^   g4;           break;
         case 169: c = g0^   g2^   g4^      g7;  break;
         case 170: c = g0^   g2^   g4^   g6;     break;
         case 171: c = g0^   g2^   g4^   g6^g7;  break;
         case 172: c = g0^   g2^   g4^g5;        break;
         case 173: c = g0^   g2^   g4^g5^   g7;  break;
         case 174: c = g0^   g2^   g4^g5^g6;     break;
         case 175: c = g0^   g2^   g4^g5^g6^g7;  break;

         case 176: c = g0^   g2^g3;              break;
         case 177: c = g0^   g2^g3^         g7;  break;
         case 178: c = g0^   g2^g3^      g6;     break;
         case 179: c = g0^   g2^g3^      g6^g7;  break;
         case 180: c = g0^   g2^g3^   g5;        break;
         case 181: c = g0^   g2^g3^   g5^   g7;  break;
         case 182: c = g0^   g2^g3^   g5^g6;     break;
         case 183: c = g0^   g2^g3^   g5^g6^g7;  break;
         case 184: c = g0^   g2^g3^g4;           break;
         case 185: c = g0^   g2^g3^g4^      g7;  break;
         case 186: c = g0^   g2^g3^g4^   g6;     break;
         case 187: c = g0^   g2^g3^g4^   g6^g7;  break;
         case 188: c = g0^   g2^g3^g4^g5;        break;
         case 189: c = g0^   g2^g3^g4^g5^   g7;  break;
         case 190: c = g0^   g2^g3^g4^g5^g6;     break;
         case 191: c = g0^   g2^g3^g4^g5^g6^g7;  break;

         case 192: c = g0^g1;                   break;
         case 193: c = g0^g1^               g7;  break;
         case 194: c = g0^g1^            g6;     break;
         case 195: c = g0^g1^            g6^g7;  break;
         case 196: c = g0^g1^         g5;        break;
         case 197: c = g0^g1^         g5^   g7;  break;
         case 198: c = g0^g1^         g5^g6;     break;
         case 199: c = g0^g1^         g5^g6^g7;  break;
         case 200: c = g0^g1^      g4;           break;
         case 201: c = g0^g1^      g4^      g7;  break;
         case 202: c = g0^g1^      g4^   g6;     break;
         case 203: c = g0^g1^      g4^   g6^g7;  break;
         case 204: c = g0^g1^      g4^g5;        break;
         case 205: c = g0^g1^      g4^g5^   g7;  break;
         case 206: c = g0^g1^      g4^g5^g6;     break;
         case 207: c = g0^g1^      g4^g5^g6^g7;  break;

         case 208: c = g0^g1^   g3;              break;
         case 209: c = g0^g1^   g3^         g7;  break;
         case 210: c = g0^g1^   g3^      g6;     break;
         case 211: c = g0^g1^   g3^      g6^g7;  break;
         case 212: c = g0^g1^   g3^   g5;        break;
         case 213: c = g0^g1^   g3^   g5^   g7;  break;
         case 214: c = g0^g1^   g3^   g5^g6;     break;
         case 215: c = g0^g1^   g3^   g5^g6^g7;  break;
         case 216: c = g0^g1^   g3^g4;           break;
         case 217: c = g0^g1^   g3^g4^      g7;  break;
         case 218: c = g0^g1^   g3^g4^   g6;     break;
         case 219: c = g0^g1^   g3^g4^   g6^g7;  break;
         case 220: c = g0^g1^   g3^g4^g5;        break;
         case 221: c = g0^g1^   g3^g4^g5^   g7;  break;
         case 222: c = g0^g1^   g3^g4^g5^g6;     break;
         case 223: c = g0^g1^   g3^g4^g5^g6^g7;  break;

         case 224: c = g0^g1^g2;                 break;
         case 225: c = g0^g1^g2^            g7;  break;
         case 226: c = g0^g1^g2^         g6;     break;
         case 227: c = g0^g1^g2^         g6^g7;  break;
         case 228: c = g0^g1^g2^      g5;        break;
         case 229: c = g0^g1^g2^      g5^   g7;  break;
         case 230: c = g0^g1^g2^      g5^g6;     break;
         case 231: c = g0^g1^g2^      g5^g6^g7;  break;
         case 232: c = g0^g1^g2^   g4;           break;
         case 233: c = g0^g1^g2^   g4^      g7;  break;
         case 234: c = g0^g1^g2^   g4^   g6;     break;
         case 235: c = g0^g1^g2^   g4^   g6^g7;  break;
         case 236: c = g0^g1^g2^   g4^g5;        break;
         case 237: c = g0^g1^g2^   g4^g5^   g7;  break;
         case 238: c = g0^g1^g2^   g4^g5^g6;     break;
         case 239: c = g0^g1^g2^   g4^g5^g6^g7;  break;

         case 240: c = g0^g1^g2^g3;              break;
         case 241: c = g0^g1^g2^g3^         g7;  break;
         case 242: c = g0^g1^g2^g3^      g6;     break;
         case 243: c = g0^g1^g2^g3^      g6^g7;  break;
         case 244: c = g0^g1^g2^g3^   g5;        break;
         case 245: c = g0^g1^g2^g3^   g5^   g7;  break;
         case 246: c = g0^g1^g2^g3^   g5^g6;     break;
         case 247: c = g0^g1^g2^g3^   g5^g6^g7;  break;
         case 248: c = g0^g1^g2^g3^g4;           break;
         case 249: c = g0^g1^g2^g3^g4^      g7;  break;
         case 250: c = g0^g1^g2^g3^g4^   g6;     break;
         case 251: c = g0^g1^g2^g3^g4^   g6^g7;  break;
         case 252: c = g0^g1^g2^g3^g4^g5;        break;
         case 253: c = g0^g1^g2^g3^g4^g5^   g7;  break;
         case 254: c = g0^g1^g2^g3^g4^g5^g6;     break;
         case 255: c = g0^g1^g2^g3^g4^g5^g6^g7;  break;
      } // end switch

      crc = (crc >> 8) ^ c;
      i = i + 1;
   }
   return ~crc;
}

// ----------------------------- crc32g --------------------------------

/* This is derived from crc32e by constructing the constant c using
algebraic expressions involving the rightmost four bits of the crc
register, rather than using a 16-way switch statement.
   We rely on the compiler to compute the constants g>>1, g>>2, and
g>>3, and load them into registers ahead of the loops. Note that crc is
now a SIGNED integer, so the right shifts of 31 are sign-propagating
shifts.
   When compiled to Cyclops with GCC, this function executes in 14 + 46n
instructions, where n is the number of bytes in the input message. There
is only one load per byte executed (but three branches). */

unsigned int crc32g(unsigned char *message) {
   int i, j, crc;
   unsigned int byte, c;
   const unsigned int g0 = 0xEDB88320, g1 = g0 >> 1,
                      g2 = g0 >> 2,    g3 = g0 >> 3;

   i = 0;
   crc = 0xFFFFFFFF;
   while (message[i] != 0) {
      byte = message[i];                // Get next byte.
      crc = crc ^ byte;
      for (j = 1; j >= 0; j--) {        // Do two times.
         c = ((crc<<31>>31) & g3) ^ ((crc<<30>>31) & g2) ^
             ((crc<<29>>31) & g1) ^ ((crc<<28>>31) & g0);
         crc = ((unsigned)crc >> 4) ^ c;
      }
      i = i + 1;
   }
   return ~crc;
}

// ----------------------------- crc32h --------------------------------

/* This is derived from crc32f by constructing the constant c using
algebraic expressions involving the rightmost eight bits of the crc
register, rather than using a 256-way switch statement.
   We rely on the compiler to compute the constants g>>1, g>>2, etc.,
and load them into registers ahead of the loops. Note that crc is now a
SIGNED integer, so the right shifts of 31 are sign-propagating shifts.
   When compiled to Cyclops with GCC, this function executes in 22 + 38n
instructions, where n is the number of bytes in the input message. There
is only one load and one branch executed per byte. */

unsigned int crc32h(unsigned char *message) {
   int i, crc;
   unsigned int byte, c;
   const unsigned int g0 = 0xEDB88320,    g1 = g0>>1,
      g2 = g0>>2, g3 = g0>>3, g4 = g0>>4, g5 = g0>>5,
      g6 = (g0>>6)^g0, g7 = ((g0>>6)^g0)>>1;

   i = 0;
   crc = 0xFFFFFFFF;
   while ((byte = message[i]) != 0) {    // Get next byte.
      crc = crc ^ byte;
      c = ((crc<<31>>31) & g7) ^ ((crc<<30>>31) & g6) ^
          ((crc<<29>>31) & g5) ^ ((crc<<28>>31) & g4) ^
          ((crc<<27>>31) & g3) ^ ((crc<<26>>31) & g2) ^
          ((crc<<25>>31) & g1) ^ ((crc<<24>>31) & g0);
      crc = ((unsigned)crc >> 8) ^ c;
      i = i + 1;
   }
   return ~crc;
}

// ------------------------------ main ---------------------------------

int main(int argc, char ** argv) {

   if (argc != 2) {
      printf("Must have exactly one argument, a message.\n"
      "You can put quotes around it if it has blanks.\n");

      exit(1);
   }

   printf("crc32a = %08x\n", crc32a(argv[1]));
   printf("crc32b = %08x\n", crc32b(argv[1]));
   printf("crc32c = %08x\n", crc32c(argv[1]));
   printf("crc32cx= %08x\n", crc32cx(argv[1])); // Must be a mult of 4 in length.
   printf("crc32d = %08x\n", crc32d(argv[1]));
   printf("crc32e = %08x\n", crc32e(argv[1]));
   printf("crc32f = %08x\n", crc32f(argv[1]));
   printf("crc32g = %08x\n", crc32g(argv[1]));
   printf("crc32h = %08x\n", crc32h(argv[1]));

   // This one skips the first character, to test the
   // halfword function with an odd starting address.
   printf("\ncrc32d = %08x\n", crc32d(argv[1] + 1));
   printf("crc32a = %08x\n", crc32a(argv[1] + 1));
   return 0;
}

/* The code above computes, in several ways, the cyclic redundancy check
usually referred to as CRC-32. This code is used by IEEE-802 (LAN/MAN
standard), PKZip, WinZip, Ethernet, and some DOD applications.

I investigated this because an early reviewer of Hacker's Delight
suggested that I might try to find some trick to speed up the
calculation of the CRC checksum. I have tried and don't see any way to
speed it up, over the standard table lookup method, which does one table
lookup per byte of message. The table size is 256 32-bit words. One
could do two bytes at a time with a table of size 65536 words, but
that's not very interesting. Not interesting, but maybe useful on a
machine with a large data cache, so the code is shown above.

This file contains eight routines for doing the CRC-32 calculation, and
a simple test driver main program.

For references, there are a few web sites, and the book "Numerical
Recipes in Fortran, The Art of Scientific Computing," by William H.
Press, Saul A. Teukolsky, William T. Vetterling, and Brian P. Flannery,
Cambridge University Press, 1992 (2nd ed.), pps 888-894.

Another book reference is "Computer Networks," by Andrew S. Tanenbaum,
second edition, pages 208-212.

Another reference, which serves as a good introduction to the subject,
and which I found very well-written and interesting, is:

Peterson, W.W. and Brown, D.T. "Cyclic Codes for Error Detection." In
Proceedings of the IRE, January 1961, 228-235.

The web site http://www.ciphersbyritter.com/ARTS/CRCMYST.HTM by Terry
Ritter is a good intro. This material also appeared as "The Great CRC
Mystery," in Dr. Dobb's Journal of Software Tools, February 1986, 26-34
and 76-83. He gives several more references.

There are other programs for computing CRC-32 at the PC Magazine web
site http://www.createwindow.com/programming/crc32/.

This code is not in Hacker's Delight, although a few of the simpler
programs may be included in a future edition. */
