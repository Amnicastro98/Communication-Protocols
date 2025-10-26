# Basics of Bitwise Operators and Bit Manipulation in C

This document covers the fundamental bitwise operators in C and basic bit manipulation techniques. Bitwise operations are essential in embedded systems for efficient memory usage, hardware control, and performance optimization.

## Bitwise Operators

### 1. Bitwise AND (&)
Performs a bitwise AND operation. The result has a 1 only where both operands have 1. This is commonly used for:
- Masking specific bits (extracting certain bits from a value)
- Clearing bits (setting specific bits to 0)
- Checking if multiple flags are set simultaneously

```c
#include <stdio.h>

int main() {
    unsigned char a = 0b10101010;  // 170 in decimal - alternating 1s and 0s
    unsigned char b = 0b11001100;  // 204 in decimal - different pattern

    unsigned char result = a & b;  // 0b10001000 (136 in decimal)
    printf("a & b = %d\n", result);
    return 0;
}
```

### 2. Bitwise OR (|)
Performs a bitwise OR operation. The result has a 1 where either operand has 1. This is commonly used for:
- Setting multiple bits simultaneously (combining flags)
- Creating bit masks that include certain bits
- Merging different bit fields into a single value

```c
// Bitwise OR: result bit is 1 if either input bit is 1
// a:     1 0 1 0 1 0 1 0
// b:     1 1 0 0 1 1 0 0
// a | b: 1 1 1 0 1 1 1 0  = 238 in decimal
unsigned char result = a | b;
printf("a | b = %d\n", result);
```

### 3. Bitwise XOR (^)
Performs a bitwise exclusive OR operation. The result has a 1 where the operands differ. This is commonly used for:
- Finding bits that are different between two values
- Toggling specific bits (XOR with 1 flips, XOR with 0 preserves)
- Simple encryption/decryption algorithms
- Swapping values without a temporary variable

```c
// Bitwise XOR: result bit is 1 if bits are different
// a:     1 0 1 0 1 0 1 0
// b:     1 1 0 0 1 1 0 0
// a ^ b: 0 1 1 0 0 1 1 0  = 102 in decimal
unsigned char result = a ^ b;
printf("a ^ b = %d\n", result);
```

### 4. Bitwise NOT (~)
Flips all bits in the operand. Every 0 becomes 1 and every 1 becomes 0. This is commonly used for:
- Creating inverted masks (all bits set except specific ones)
- Finding the two's complement (for negative numbers in two's complement)
- Clearing all bits in a register by ANDing with ~0 (though this is redundant)

```c
// Bitwise NOT: flips all bits
// a:        1 0 1 0 1 0 1 0
// ~a:       0 1 0 1 0 1 0 1  = 85 in decimal (for 8-bit unsigned char)
unsigned char result = ~a;
printf("~a = %d\n", result);
```

### 5. Left Shift (<<)
Shifts bits to the left by the specified number of positions, filling with zeros on the right. This is equivalent to multiplying by 2^n (for n shift positions). Commonly used for:
- Fast multiplication by powers of 2
- Positioning bits at specific locations in a word
- Creating bit masks for specific bit ranges

```c
// Left shift: moves bits left, fills right with zeros
// a:        1 0 1 0 1 0 1 0
// a << 2:   1 0 1 0 1 0 0 0  = 168 in decimal (original * 4)
unsigned char result = a << 2;
printf("a << 2 = %d\n", result);
```

### 6. Right Shift (>>)
Shifts bits to the right by the specified number of positions. For unsigned types, fills with zeros on the left. For signed types, may sign-extend. This is equivalent to dividing by 2^n. Commonly used for:
- Fast division by powers of 2 (unsigned types)
- Extracting higher-order bits
- Converting between different bit field positions

```c
// Right shift: moves bits right, fills left with zeros (unsigned)
// a:        1 0 1 0 1 0 1 0
// a >> 2:   0 0 1 0 1 0 1 0  = 42 in decimal (original / 4)
unsigned char result = a >> 2;
printf("a >> 2 = %d\n", result);
```

## Basic Bit Manipulation Techniques

### Setting a Bit
To set a specific bit (make it 1). This is commonly used to enable features or flags:

```c
// Create a mask with only the target bit set: (1 << position)
// OR with the original value ensures the bit becomes 1 without affecting others
unsigned char set_bit = a | (1 << position);  // position is 0-indexed from right (LSB is 0)
```

### Clearing a Bit
To clear a specific bit (make it 0). This is commonly used to disable features or reset flags:

```c
// Create a mask with only the target bit cleared: ~(1 << position)
// AND with the original value ensures the bit becomes 0 without affecting others
unsigned char clear_bit = a & ~(1 << position);
```

### Toggling a Bit
To flip a specific bit. This is commonly used for switching states (on/off, high/low):

```c
// XOR with a mask containing only the target bit set
// XOR with 1 flips the bit, XOR with 0 leaves it unchanged
unsigned char toggle_bit = a ^ (1 << position);
```

### Checking if a Bit is Set
To check if a specific bit is 1. This is commonly used for conditional logic based on flags:

```c
// AND with a mask isolates the target bit
// Non-zero result means the bit was set, zero means it was clear
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
