/*
 * Basics of Bitwise Operators and Bit Manipulation in C
 *
 * This file covers the fundamental bitwise operators in C:
 * - & (AND): Performs bitwise AND operation
 * - | (OR): Performs bitwise OR operation
 * - ^ (XOR): Performs bitwise exclusive OR operation
 * - ~ (NOT): Performs bitwise NOT (complement) operation
 * - << (Left Shift): Shifts bits to the left
 * - >> (Right Shift): Shifts bits to the right
 *
 * Bit manipulation involves working with individual bits in integers.
 * This is crucial in embedded systems for efficient memory usage and hardware control.
 */

#include <stdio.h>

int main() {
    // Example variables
    unsigned char a = 0b10101010;  // 170 in decimal
    unsigned char b = 0b11001100;  // 204 in decimal

    printf("a = %d (binary: %08b)\n", a, a);
    printf("b = %d (binary: %08b)\n\n", b, b);

    // Bitwise AND (&)
    // Result has 1 only where both bits are 1
    unsigned char and_result = a & b;
    printf("a & b = %d (binary: %08b)\n", and_result, and_result);

    // Bitwise OR (|)
    // Result has 1 where either bit is 1
    unsigned char or_result = a | b;
    printf("a | b = %d (binary: %08b)\n", or_result, or_result);

    // Bitwise XOR (^)
    // Result has 1 where bits are different
    unsigned char xor_result = a ^ b;
    printf("a ^ b = %d (binary: %08b)\n", xor_result, xor_result);

    // Bitwise NOT (~)
    // Flips all bits
    unsigned char not_a = ~a;
    printf("~a = %d (binary: %08b)\n", not_a, not_a);

    // Left Shift (<<)
    // Shifts bits left, fills with 0s
    unsigned char left_shift = a << 2;
    printf("a << 2 = %d (binary: %08b)\n", left_shift, left_shift);

    // Right Shift (>>)
    // Shifts bits right, fills with 0s for unsigned
    unsigned char right_shift = a >> 2;
    printf("a >> 2 = %d (binary: %08b)\n", right_shift, right_shift);

    // Basic bit manipulation: Setting a bit
    unsigned char set_bit = a | (1 << 3);  // Set bit 3 (0-indexed from right)
    printf("Setting bit 3 in a: %d (binary: %08b)\n", set_bit, set_bit);

    // Clearing a bit
    unsigned char clear_bit = a & ~(1 << 3);  // Clear bit 3
    printf("Clearing bit 3 in a: %d (binary: %08b)\n", clear_bit, clear_bit);

    // Toggling a bit
    unsigned char toggle_bit = a ^ (1 << 3);  // Toggle bit 3
    printf("Toggling bit 3 in a: %d (binary: %08b)\n", toggle_bit, toggle_bit);

    // Checking if a bit is set
    int is_bit_set = (a & (1 << 3)) != 0;
    printf("Is bit 3 set in a? %s\n", is_bit_set ? "Yes" : "No");

    return 0;
}

