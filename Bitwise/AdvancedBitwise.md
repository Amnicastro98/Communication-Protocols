# Advanced Bitwise Operators and Bit Manipulation in C

This document covers advanced techniques for bit manipulation, including efficient algorithms, hardware register manipulation, and complex bit operations commonly used in embedded systems.

## Bit Fields and Hardware Registers

Bit fields allow you to specify the exact number of bits for struct members, which is useful for hardware register manipulation:

```c
#include <stdint.h>

typedef union {
    uint32_t reg;  // Full 32-bit register
    struct {
        uint32_t bit0 : 1;
        uint32_t bit1 : 1;
        uint32_t bit2 : 1;
        uint32_t bit3 : 1;
        uint32_t nibble : 4;  // 4-bit field
        uint32_t byte : 8;    // 8-bit field
        uint32_t reserved : 16; // Remaining bits
    } bits;
} HardwareRegister;
```

## Efficient Bit Counting Algorithms

### Basic Bit Counting
```c
int count_set_bits(uint32_t n) {
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    return count;
}
```

### Brian Kernighan's Algorithm (More Efficient)
```c
int count_set_bits_efficient(uint32_t n) {
    int count = 0;
    while (n) {
        n &= (n - 1);  // Clear the least significant set bit
        count++;
    }
    return count;
}
```

## Bit Reversal

```c
uint8_t reverse_bits(uint8_t n) {
    uint8_t result = 0;
    for (int i = 0; i < 8; i++) {
        result = (result << 1) | (n & 1);
        n >>= 1;
    }
    return result;
}
```

## Power of 2 Operations

### Check if Power of 2
```c
int is_power_of_two(uint32_t n) {
    return n && !(n & (n - 1));
}
```

### Find Next Power of 2
```c
uint32_t next_power_of_two(uint32_t n) {
    if (n == 0) return 1;
    n--;
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    return n + 1;
}
```

## Gray Code Conversion

Gray code is a binary numeral system where two consecutive values differ in only one bit.

### Binary to Gray Code
```c
uint32_t binary_to_gray(uint32_t n) {
    return n ^ (n >> 1);
}
```

### Gray Code to Binary
```c
uint32_t gray_to_binary(uint32_t n) {
    uint32_t result = n;
    while (n >>= 1) {
        result ^= n;
    }
    return result;
}
```

## Bit Masking and Extraction

### Extracting Specific Bits
```c
uint32_t data = 0xABCD1234;

// Extract lower 8 bits
uint8_t lower_byte = data & 0xFF;

// Extract bits 8-15
uint8_t middle_byte = (data >> 8) & 0xFF;

// Extract upper 16 bits
uint16_t upper_half = (data >> 16) & 0xFFFF;
```

### Setting Specific Bit Ranges
```c
// Set bits 8-15 to 0x42
uint32_t modified = (data & ~0xFF00) | (0x42 << 8);
```

## Advanced Bit Manipulation Techniques

### Bit Rotation
```c
// Left rotate
uint32_t rotate_left(uint32_t n, int positions) {
    return (n << positions) | (n >> (32 - positions));
}

// Right rotate
uint32_t rotate_right(uint32_t n, int positions) {
    return (n >> positions) | (n << (32 - positions));
}
```

### Finding Bit Position
```c
// Find position of least significant set bit
int lowest_set_bit(uint32_t n) {
    if (n == 0) return -1;
    int pos = 0;
    while ((n & 1) == 0) {
        n >>= 1;
        pos++;
    }
    return pos;
}

// Find position of most significant set bit
int highest_set_bit(uint32_t n) {
    if (n == 0) return -1;
    int pos = 0;
    while (n >>= 1) {
        pos++;
    }
    return pos;
}
```

### Bit Interleaving (Morton Code)
```c
// Interleave bits of two 16-bit numbers into a 32-bit Morton code
uint32_t morton_code(uint16_t x, uint16_t y) {
    uint32_t result = 0;
    for (int i = 0; i < 16; i++) {
        result |= ((x >> i) & 1) << (2 * i);
        result |= ((y >> i) & 1) << (2 * i + 1);
    }
    return result;
}
```

## Hardware-Specific Applications

### GPIO Pin Control
```c
#define GPIO_PORTA_DATA  (*((volatile uint32_t *)0x40004000))
#define PIN_MASK(pin)    (1 << pin)

// Set pin high
void gpio_set_pin(int pin) {
    GPIO_PORTA_DATA |= PIN_MASK(pin);
}

// Clear pin low
void gpio_clear_pin(int pin) {
    GPIO_PORTA_DATA &= ~PIN_MASK(pin);
}

// Toggle pin
void gpio_toggle_pin(int pin) {
    GPIO_PORTA_DATA ^= PIN_MASK(pin);
}
```

### Interrupt Flag Manipulation
```c
#define INTERRUPT_FLAGS (*((volatile uint32_t *)0xE000E200))

// Clear interrupt flag
void clear_interrupt_flag(int flag) {
    INTERRUPT_FLAGS |= (1 << flag);
}

// Check if interrupt is pending
int is_interrupt_pending(int flag) {
    return (INTERRUPT_FLAGS & (1 << flag)) != 0;
}
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
