# Advanced Bitwise Operators and Bit Manipulation in C

This document covers advanced techniques for bit manipulation, including efficient algorithms, hardware register manipulation, and complex bit operations commonly used in embedded systems.

## Bit Fields and Hardware Registers

Bit fields allow you to specify the exact number of bits for struct members, which is crucial for hardware register manipulation in embedded systems. This saves memory and provides direct access to individual bits or bit groups within registers.

```c
#include <stdint.h>  // Include for fixed-width integer types

// Define a union that allows accessing the same memory as either:
// 1. A full 32-bit register value, or
// 2. Individual bit fields for hardware control
typedef union {
    uint32_t reg;  // Full 32-bit register - access entire register at once
    struct {
        // Each field specifies exactly how many bits it occupies
        uint32_t bit0 : 1;     // 1-bit field (can be 0 or 1)
        uint32_t bit1 : 1;     // Another 1-bit field
        uint32_t bit2 : 1;     // Third 1-bit field
        uint32_t bit3 : 1;     // Fourth 1-bit field
        uint32_t nibble : 4;   // 4-bit field (values 0-15)
        uint32_t byte : 8;     // 8-bit field (values 0-255)
        uint32_t reserved : 16; // Remaining 16 bits (padding/reserved)
    } bits;  // Access individual bits or groups
} HardwareRegister;

// Example usage:
HardwareRegister gpio_reg;  // Declare a register variable
gpio_reg.reg = 0x00000000;  // Initialize entire register to 0

// Set individual control bits
gpio_reg.bits.bit0 = 1;     // Enable feature 0
gpio_reg.bits.bit1 = 0;     // Disable feature 1
gpio_reg.bits.nibble = 0xA; // Set 4-bit mode to 10 (hex A)

// Now gpio_reg.reg contains the combined value of all bit fields
```

## Efficient Bit Counting Algorithms

### Basic Bit Counting
This method checks each bit individually by shifting through all 32 bits.

```c
int count_set_bits(uint32_t n) {
    int count = 0;           // Initialize counter to 0
    while (n) {              // Loop until n becomes 0
        count += n & 1;      // Add 1 if least significant bit is set
        n >>= 1;             // Shift right by 1 to check next bit
    }
    return count;            // Return total number of set bits
}
```

### Brian Kernighan's Algorithm (More Efficient)
This algorithm is more efficient because it only iterates once for each set bit, rather than once per bit position.

```c
int count_set_bits_efficient(uint32_t n) {
    int count = 0;              // Initialize counter
    while (n) {                 // Continue until no more set bits
        n &= (n - 1);           // Clear the least significant set bit
                                 // Example: n = 0b1100 (12)
                                 // n-1 = 0b1011 (11)
                                 // n & (n-1) = 0b1000 (8) - cleared bit 2
        count++;                // Increment count for each set bit found
    }
    return count;               // Return the total count
}
```

## Bit Reversal

Bit reversal takes the bits of a number and reverses their order. This is useful in digital signal processing, cryptography, and certain hardware interfaces.

```c
uint8_t reverse_bits(uint8_t n) {
    uint8_t result = 0;        // Start with all bits clear
    for (int i = 0; i < 8; i++) {  // Process each of the 8 bits
        result = (result << 1) | (n & 1);  // Shift result left, add LSB of n
        n >>= 1;               // Shift n right to process next bit
    }
    return result;             // Return the reversed bits
}

// Example: reverse_bits(0b11010011) = 0b11001011
// Original: bit7=1, bit6=1, bit5=0, bit4=1, bit3=0, bit2=0, bit1=1, bit0=1
// Reversed: bit7=1, bit6=1, bit5=0, bit4=0, bit3=1, bit2=0, bit1=1, bit0=1
```

## Power of 2 Operations

### Check if Power of 2
A number is a power of 2 if it has exactly one bit set. This function uses the property that `n & (n-1)` clears the least significant set bit, so for powers of 2, this result should be 0.

```c
int is_power_of_two(uint32_t n) {
    return n && !(n & (n - 1));  // n != 0 AND no other bits set
}

// Examples:
// is_power_of_two(1) = true  (0b0001)
// is_power_of_two(2) = true  (0b0010)
// is_power_of_two(3) = false (0b0011)
// is_power_of_two(4) = true  (0b0100)
```

### Find Next Power of 2
This algorithm finds the smallest power of 2 that is greater than or equal to n. It uses bitwise operations to set all bits below the highest set bit.

```c
uint32_t next_power_of_two(uint32_t n) {
    if (n == 0) return 1;     // Special case: 2^0 = 1
    n--;                      // Decrement to handle exact powers of 2
    n |= n >> 1;              // Set bits to the right of highest bit
    n |= n >> 2;              // Continue spreading bits
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;             // For 32-bit numbers
    return n + 1;             // Add 1 to get the next power of 2
}

// Examples:
// next_power_of_two(5) = 8   (100 -> 1000)
// next_power_of_two(9) = 16  (1001 -> 10000)
// next_power_of_two(15) = 16 (1111 -> 10000)
```

## Gray Code Conversion

Gray code is a binary numeral system where two consecutive values differ in only one bit. This is useful in rotary encoders, error correction, and digital communications to minimize errors during transitions.

### Binary to Gray Code
Converting binary to Gray code involves XORing each bit with the next higher bit.

```c
uint32_t binary_to_gray(uint32_t n) {
    return n ^ (n >> 1);  // XOR each bit with the bit to its right
}

// Example: binary_to_gray(0b0101) = 0b0111
// Binary:  0 1 0 1
// Shifted:   0 0 1 0  (n >> 1)
// XOR:     0 1 1 1  = 7
```

### Gray Code to Binary
Converting Gray code back to binary requires XORing each bit with all higher bits.

```c
uint32_t gray_to_binary(uint32_t n) {
    uint32_t result = n;        // Start with the Gray code
    while (n >>= 1) {          // While there are higher bits to process
        result ^= n;            // XOR with the remaining higher bits
    }
    return result;              // Return the binary equivalent
}

// Example: gray_to_binary(0b0111) = 0b0101
// Gray:    0 1 1 1
// XOR with shifted: XOR with 0011 = 0100, then XOR with 0001 = 0101
```

## Bit Masking and Extraction

Bit masking is essential for extracting or modifying specific portions of data without affecting other bits. This is commonly used when working with packed data structures or hardware registers.

### Extracting Specific Bits
```c
uint32_t data = 0xABCD1234;  // Example: 0xAB CD 12 34 in hex

// Extract lower 8 bits (bits 0-7)
uint8_t lower_byte = data & 0xFF;        // Result: 0x34

// Extract bits 8-15 (second byte from right)
uint8_t middle_byte = (data >> 8) & 0xFF; // Result: 0x12

// Extract upper 16 bits (bits 16-31)
uint16_t upper_half = (data >> 16) & 0xFFFF; // Result: 0xABCD
```

### Setting Specific Bit Ranges
```c
// Set bits 8-15 to 0x42 while preserving other bits
uint32_t modified = (data & ~0xFF00) | (0x42 << 8);
// Breakdown:
// (data & ~0xFF00) clears bits 8-15
// (0x42 << 8) positions 0x42 at bits 8-15
// OR combines them, preserving other bits
```

## Advanced Bit Manipulation Techniques

### Bit Rotation
Bit rotation moves bits in a circular fashion, wrapping around from one end to the other. This is different from regular shifts which lose bits.

```c
// Left rotate: bits move left, high bits wrap to low positions
uint32_t rotate_left(uint32_t n, int positions) {
    return (n << positions) | (n >> (32 - positions));
}

// Right rotate: bits move right, low bits wrap to high positions
uint32_t rotate_right(uint32_t n, int positions) {
    return (n >> positions) | (n << (32 - positions));
}

// Example: rotate_left(0b11010011, 2) = 0b01001111
// Original:  1 1 0 1 0 0 1 1
// Left rot 2: 0 1 0 0 1 1 1 1  (last 2 bits move to front)
```

### Finding Bit Position
These functions find the position of specific bits within a number, useful for bit scanning operations.

```c
// Find position of least significant set bit (rightmost 1)
int lowest_set_bit(uint32_t n) {
    if (n == 0) return -1;     // No set bits
    int pos = 0;
    while ((n & 1) == 0) {    // While LSB is 0
        n >>= 1;               // Shift right to check next bit
        pos++;                 // Increment position counter
    }
    return pos;                // Return position (0-based from right)
}

// Find position of most significant set bit (leftmost 1)
int highest_set_bit(uint32_t n) {
    if (n == 0) return -1;     // No set bits
    int pos = 0;
    while (n >>= 1) {          // While there are higher bits
        pos++;                 // Count how many shifts until n becomes 0
    }
    return pos;                // Return position (0-based from right)
}

// Examples:
// lowest_set_bit(0b00011000) = 3  (bit 3 is the rightmost set bit)
// highest_set_bit(0b00011000) = 4  (bit 4 is the leftmost set bit)
```

### Bit Interleaving (Morton Code)
Morton codes (also called Z-order curves) interleave bits from multiple coordinates, useful for spatial indexing and cache-efficient data structures.

```c
// Interleave bits of two 16-bit numbers into a 32-bit Morton code
uint32_t morton_code(uint16_t x, uint16_t y) {
    uint32_t result = 0;       // Start with empty result
    for (int i = 0; i < 16; i++) {  // Process each bit position
        // Extract bit i from x, place at even positions (0,2,4,...)
        result |= ((x >> i) & 1) << (2 * i);
        // Extract bit i from y, place at odd positions (1,3,5,...)
        result |= ((y >> i) & 1) << (2 * i + 1);
    }
    return result;             // Return interleaved Morton code
}

// Example: morton_code(0b0001, 0b0010) = 0b00001010
// x=0001 (binary), y=0010 (binary)
// Result: x0 y0 x1 y1 x2 y2 x3 y3 = 0 0 0 1 0 1 0 0
```

## Hardware-Specific Applications

In embedded systems, bitwise operations are crucial for directly manipulating hardware registers. These examples demonstrate typical usage for GPIO control and interrupt handling, using memory-mapped registers common in microcontrollers like ARM Cortex-M.

### GPIO Pin Control
GPIO (General Purpose Input/Output) pins are controlled via memory-mapped registers. The `volatile` keyword ensures the compiler doesn't optimize away repeated reads/writes to hardware memory.

```c
// Memory-mapped register for GPIO Port A data (example address for STM32-like MCU)
#define GPIO_PORTA_DATA  (*((volatile uint32_t *)0x40004000))
// Macro to create a bit mask for a specific pin (0-31)
#define PIN_MASK(pin)    (1U << (pin))  // Use U for unsigned to avoid sign issues

// Set a specific pin high (logic 1)
void gpio_set_pin(int pin) {
    GPIO_PORTA_DATA |= PIN_MASK(pin);  // OR with mask to set bit without affecting others
}

// Clear a specific pin low (logic 0)
void gpio_clear_pin(int pin) {
    GPIO_PORTA_DATA &= ~PIN_MASK(pin); // AND with inverted mask to clear bit
}

// Toggle a specific pin (flip its state)
void gpio_toggle_pin(int pin) {
    GPIO_PORTA_DATA ^= PIN_MASK(pin);  // XOR with mask to flip the bit
}

// Example usage:
// gpio_set_pin(5);      // Set pin 5 high
// gpio_clear_pin(5);    // Set pin 5 low
// gpio_toggle_pin(5);   // Toggle pin 5
```

### Interrupt Flag Manipulation
Interrupts allow the CPU to respond to hardware events. Flags indicate pending interrupts, and clearing them acknowledges the event.

```c
// Memory-mapped NVIC (Nested Vectored Interrupt Controller) interrupt clear-pending register
#define INTERRUPT_FLAGS (*((volatile uint32_t *)0xE000E200))  // Example for Cortex-M

// Clear (acknowledge) a specific interrupt flag
void clear_interrupt_flag(int flag) {
    INTERRUPT_FLAGS |= (1U << flag);  // Writing 1 to the bit clears the pending flag
                                      // (Hardware-specific behavior: write-1-to-clear)
}

// Check if a specific interrupt is pending
int is_interrupt_pending(int flag) {
    return (INTERRUPT_FLAGS & (1U << flag)) != 0;  // Non-zero if flag bit is set
}

// Example usage:
// if (is_interrupt_pending(3)) {  // Check if interrupt 3 is pending
//     clear_interrupt_flag(3);    // Acknowledge and clear it
// }
```

## Performance Considerations

- Bitwise operations are generally very fast
- Use unsigned types to avoid undefined behavior with shifts
- Bit fields can save memory but may have alignment issues
- Consider endianness when working with multi-byte data
- Use volatile for hardware registers to prevent compiler optimization

## Common Pitfalls

1. **Sign Extension**: Right shift on signed integers can sign-extend
2. **Undefined Behavior**: Left shifting negative numbers or shifting by more than type width
3. **Endianness**: Bit field layout can vary between big-endian and little-endian systems
4. **Alignment**: Bit fields may introduce padding bytes
5. **Volatile**: Forgetting to use volatile for hardware registers

## Example: Complete Bit Manipulation Library

```c
#include <stdint.h>

// Bit manipulation macros
#define BIT_SET(reg, bit)     ((reg) |= (1 << (bit)))
#define BIT_CLEAR(reg, bit)   ((reg) &= ~(1 << (bit)))
#define BIT_TOGGLE(reg, bit)  ((reg) ^= (1 << (bit)))
#define BIT_CHECK(reg, bit)   ((reg) & (1 << (bit)))

// Bit mask operations
#define MASK_SET(reg, mask)   ((reg) |= (mask))
#define MASK_CLEAR(reg, mask) ((reg) &= ~(mask))
#define MASK_TOGGLE(reg, mask) ((reg) ^= (mask))

// Utility functions
uint32_t reverse_bits_32(uint32_t n);
int count_leading_zeros(uint32_t n);
int count_trailing_zeros(uint32_t n);
uint32_t isolate_rightmost_set_bit(uint32_t n);
uint32_t isolate_rightmost_cleared_bit(uint32_t n);
```

This advanced guide provides the foundation for complex bit manipulation tasks in embedded systems, from basic hardware control to sophisticated algorithms.
