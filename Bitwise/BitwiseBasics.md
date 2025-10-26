# Basics of Bitwise Operators and Bit Manipulation in C

This document covers the fundamental bitwise operators in C and basic bit manipulation techniques. Bitwise operations are essential in embedded systems for efficient memory usage, hardware control, and performance optimization.

## Bitwise Operators

### 1. Bitwise AND (&)
Performs a bitwise AND operation. The result has a 1 only where both operands have 1.

```c
#include <stdio.h>

int main() {
    unsigned char a = 0b10101010;  // 170 in decimal
    unsigned char b = 0b11001100;  // 204 in decimal

    unsigned char result = a & b;  // 0b10001000 (136 in decimal)
    printf("a & b = %d\n", result);
    return 0;
}
```

### 2. Bitwise OR (|)
Performs a bitwise OR operation. The result has a 1 where either operand has 1.

```c
unsigned char result = a | b;  // 0b11101110 (238 in decimal)
printf("a | b = %d\n", result);
```

### 3. Bitwise XOR (^)
Performs a bitwise exclusive OR operation. The result has a 1 where the operands differ.

```c
unsigned char result = a ^ b;  // 0b01100110 (102 in decimal)
printf("a ^ b = %d\n", result);
```

### 4. Bitwise NOT (~)
Flips all bits in the operand.

```c
unsigned char result = ~a;  // For 8-bit: 0b01010101 (85 in decimal)
printf("~a = %d\n", result);
```

### 5. Left Shift (<<)
Shifts bits to the left by the specified number of positions, filling with zeros.

```c
unsigned char result = a << 2;  // 0b10101000 (168 in decimal)
printf("a << 2 = %d\n", result);
```

### 6. Right Shift (>>)
Shifts bits to the right by the specified number of positions. For unsigned types, fills with zeros.

```c
unsigned char result = a >> 2;  // 0b00101010 (42 in decimal)
printf("a >> 2 = %d\n", result);
```

## Basic Bit Manipulation Techniques

### Setting a Bit
To set a specific bit (make it 1):

```c
unsigned char set_bit = a | (1 << position);  // position is 0-indexed from right
```

### Clearing a Bit
To clear a specific bit (make it 0):

```c
unsigned char clear_bit = a & ~(1 << position);
```

### Toggling a Bit
To flip a specific bit:

```c
unsigned char toggle_bit = a ^ (1 << position);
```

### Checking if a Bit is Set
To check if a specific bit is 1:

```c
int is_set = (a & (1 << position)) != 0;
```

## Example Program

```c
#include <stdio.h>  // Include standard I/O library for printf function

int main() {
    // Initialize two unsigned char variables with binary literals
    // unsigned char is 8 bits, perfect for bit manipulation examples
    unsigned char a = 0b10101010;  // 170 in decimal (AA in hex)
    unsigned char b = 0b11001100;  // 204 in decimal (CC in hex)

    // Print the initial values in both decimal and binary format
    // %08b prints exactly 8 binary digits with leading zeros
    printf("a = %d (0b%08b)\n", a, a);
    printf("b = %d (0b%08b)\n\n", b, b);

    // Demonstrate all bitwise operators with explanations
    // Each operation shows the result in decimal and binary

    // Bitwise AND: Result has 1 only where both bits are 1
    printf("a & b = %d (0b%08b)\n", a & b, a & b);

    // Bitwise OR: Result has 1 where either bit is 1
    printf("a | b = %d (0b%08b)\n", a | b, a | b);

    // Bitwise XOR: Result has 1 where bits are different
    printf("a ^ b = %d (0b%08b)\n", a ^ b, a ^ b);

    // Bitwise NOT: Flips all bits (cast to unsigned char to avoid sign extension)
    printf("~a = %d (0b%08b)\n", (unsigned char)~a, (unsigned char)~a);

    // Left shift: Shifts bits left by 2 positions, fills with zeros
    printf("a << 2 = %d (0b%08b)\n", a << 2, a << 2);

    // Right shift: Shifts bits right by 2 positions, fills with zeros (unsigned)
    printf("a >> 2 = %d (0b%08b)\n\n", a >> 2, a >> 2);

    // Practical bit manipulation examples
    // These are commonly used in embedded systems for hardware control

    // Setting a bit: Use OR with a mask to set bit 3 (4th bit from right)
    // (1 << 3) creates 0b00001000, OR ensures that bit becomes 1
    printf("Setting bit 3 in a: %d (0b%08b)\n", a | (1 << 3), a | (1 << 3));

    // Clearing a bit: Use AND with inverted mask to clear bit 3
    // ~(1 << 3) creates 0b11110111, AND ensures that bit becomes 0
    printf("Clearing bit 3 in a: %d (0b%08b)\n", a & ~(1 << 3), a & ~(1 << 3));

    // Toggling a bit: Use XOR with mask to flip bit 3
    // XOR with 1 flips the bit, XOR with 0 leaves it unchanged
    printf("Toggling bit 3 in a: %d (0b%08b)\n", a ^ (1 << 3), a ^ (1 << 3));

    // Checking if a bit is set: Use AND with mask and compare to 0
    // If result != 0, the bit was set; if == 0, the bit was clear
    printf("Is bit 3 set in a? %s\n", (a & (1 << 3)) ? "Yes" : "No");

    return 0;  // Return 0 to indicate successful program execution
}
```

## Key Points
- Bitwise operations work on individual bits
- Use unsigned types to avoid sign extension issues
- Left shift multiplies by powers of 2, right shift divides
- Bit manipulation is crucial for embedded systems programming
- Always consider the data type size (8-bit, 16-bit, 32-bit, etc.)
