# Advanced SPI Quick Reference

## Table of Contents
- [Multi-Slave Configurations](#multi-slave-configurations)
- [Daisy Chaining](#daisy-chaining)
- [Error Handling](#error-handling)
- [Performance Optimization](#performance-optimization)
- [Advanced Code Examples](#advanced-code-examples)

## Multi-Slave Configurations
In systems with multiple SPI slaves, each slave requires its own SS/CS pin from the master. The master controls which slave is active by pulling the corresponding SS/CS low.

```
Master
  |
  |--SS1--> Slave 1
  |--SS2--> Slave 2
  |--SS3--> Slave 3
  |
  +--MOSI--> All slaves
  +--MISO<-- All slaves (but only active slave drives)
  +--SCK--> All slaves
```

Important: Only one slave should be selected at a time to avoid bus conflicts.

## Daisy Chaining
Daisy chaining allows multiple devices to be connected in series, reducing the number of SS/CS pins needed. Data shifts through each device in the chain.

```
Master --MOSI--> Device1 --MOSI--> Device2 --MOSI--> Device3
   ^              ^              ^              ^
   |              |              |              |
   +--MISO<-- Device1 <--MISO<-- Device2 <--MISO<-- Device3
   +--SCK--> All devices
   +--SS--> All devices (single SS for the chain)
```

Each device must support daisy chaining and have the same word length.

## Error Handling
Common SPI errors and handling strategies:

- **Overrun Error**: Slave receives new data before reading previous data
- **Collision Error**: Master and slave try to transmit simultaneously
- **Mode Fault**: Multiple masters on the bus

```c
// Check for SPI errors (AVR example)
uint8_t SPI_CheckErrors(void) {
    uint8_t errors = 0;
    if (SPSR & (1 << WCOL)) {
        errors |= (1 << 0); // Write collision
        SPSR |= (1 << WCOL); // Clear flag
    }
    if (SPSR & (1 << SPI2X)) {
        // SPI2X is not an error, but double speed mode
    }
    return errors;
}
```

## Performance Optimization
Techniques to improve SPI performance:

- **Higher Clock Speeds**: Increase SCK frequency (limited by device capabilities)
- **Double Speed Mode**: Use SPI2X bit for 2x speed (if supported)
- **DMA**: Use Direct Memory Access for large data transfers
- **Interrupt-Driven**: Use interrupts instead of polling for better CPU utilization

```c
// Enable SPI interrupt and double speed mode
void SPI_EnableInterrupt(void) {
    SPCR |= (1 << SPIE); // Enable SPI interrupt
    SPSR |= (1 << SPI2X); // Double SPI speed
}

// ISR for SPI
ISR(SPI_STC_vect) {
    // Handle received data
    uint8_t data = SPDR;
    // Process data...
}
```

## Advanced Code Examples
```c
#include <avr/io.h>
#include <avr/interrupt.h>

// Buffer for SPI data
#define SPI_BUFFER_SIZE 256
volatile uint8_t spi_buffer[SPI_BUFFER_SIZE];
volatile uint8_t spi_index = 0;

// Multi-slave management
#define NUM_SLAVES 3
const uint8_t slave_pins[NUM_SLAVES] = {PB2, PB1, PB0};

void SPI_SelectSlave(uint8_t slave_id) {
    // Deselect all slaves first
    for (uint8_t i = 0; i < NUM_SLAVES; i++) {
        PORTB |= (1 << slave_pins[i]);
    }
    // Select specific slave
    PORTB &= ~(1 << slave_pins[slave_id]);
}

void SPI_DeselectAll(void) {
    for (uint8_t i = 0; i < NUM_SLAVES; i++) {
        PORTB |= (1 << slave_pins[i]);
    }
}

// DMA-like transfer using interrupts
volatile uint8_t *tx_buffer;
volatile uint8_t *rx_buffer;
volatile uint16_t transfer_size;
volatile uint16_t transfer_count = 0;

void SPI_StartDMATransfer(uint8_t *tx, uint8_t *rx, uint16_t size) {
    tx_buffer = tx;
    rx_buffer = rx;
    transfer_size = size;
    transfer_count = 0;
    
    // Load first byte
    SPDR = tx_buffer[transfer_count++];
    SPCR |= (1 << SPIE); // Enable interrupt
}

ISR(SPI_STC_vect) {
    // Store received byte
    if (rx_buffer) {
        rx_buffer[transfer_count - 1] = SPDR;
    }
    
    if (transfer_count < transfer_size) {
        // Send next byte
        SPDR = tx_buffer[transfer_count++];
    } else {
        // Transfer complete
        SPCR &= ~(1 << SPIE); // Disable interrupt
        // Signal completion...
    }
}

// Example usage
int main(void) {
    // Initialize SPI
    SPI_MasterInit();
    
    // Set slave pins as output
    DDRB |= (1 << PB2) | (1 << PB1) | (1 << PB0);
    SPI_DeselectAll();
    
    sei(); // Enable global interrupts
    
    // Example multi-slave communication
    SPI_SelectSlave(0);
    uint8_t data = SPI_MasterTransmit(0x55);
    SPI_DeselectAll();
    
    // Example DMA transfer
    uint8_t tx_data[10] = {0,1,2,3,4,5,6,7,8,9};
    uint8_t rx_data[10];
    SPI_StartDMATransfer(tx_data, rx_data, 10);
    
    while(1) {
        // Main loop
    }
    
    return 0;
}
