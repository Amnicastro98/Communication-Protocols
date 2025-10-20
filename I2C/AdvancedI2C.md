# Advanced I2C Quick Reference

## Table of Contents
- [Multi-Master Systems](#multi-master-systems)
- [Clock Stretching](#clock-stretching)
- [Arbitration](#arbitration)
- [Error Handling](#error-handling)
- [Performance Optimization](#performance-optimization)
- [Advanced Code Examples](#advanced-code-examples)

## Multi-Master Systems
I2C supports multiple masters on the same bus. Masters must cooperate to avoid conflicts.

```
Master 1 --SDA--+-- Pull-up -- VCC
                |
Master 2 --SDA--+
                |
Slave 1 --SDA--+
                |
Master 1 --SCL--+
                |
Master 2 --SCL--+
                |
Slave 1 --SCL--+
```

Only one master can control the bus at a time. Arbitration ensures fair access.

## Clock Stretching
Clock stretching allows a slave to slow down the master by holding SCL low. This is useful when the slave needs more time to process data.

```
Master sends clock:  _   _   _   _   _   _
Slave stretches:     ____|_____|_____|_____
Effective clock:     _|__|__|__|__|__|__|_
```

Slaves can stretch the clock during:
- Address acknowledgment
- Data acknowledgment
- Data reception/transmission

## Arbitration
In multi-master systems, arbitration prevents data corruption when multiple masters try to transmit simultaneously.

- **Bus arbitration**: Masters monitor SDA during transmission
- **Clock synchronization**: SCL is ANDed together
- **Winner determination**: Master that transmits '1' while other transmits '0' loses arbitration

```c
// Check for arbitration lost (AVR example)
uint8_t I2C_ArbitrationLost(void) {
    return (TWSR & (1 << TWSR)) ? 1 : 0;
}
```

## Error Handling
Common I2C errors and handling strategies:

- **No ACK received**: Device not present or busy
- **Arbitration lost**: Another master took control
- **Bus busy**: START condition detected while bus is busy
- **Bus error**: Illegal START/STOP conditions

```c
// I2C error checking function
uint8_t I2C_GetError(void) {
    uint8_t status = TWSR & 0xF8;
    switch(status) {
        case 0x00: return I2C_ERROR_BUS_ERROR;
        case 0x20: return I2C_ERROR_SLA_NACK;
        case 0x30: return I2C_ERROR_DATA_NACK;
        case 0x38: return I2C_ERROR_ARBITRATION_LOST;
        default: return I2C_NO_ERROR;
    }
}
```

## Performance Optimization
Techniques to improve I2C performance:

- **Higher Clock Speeds**: Standard (100kHz), Fast (400kHz), Fast+ (1MHz), High-speed (3.4MHz)
- **Burst transfers**: Send multiple bytes in one transaction
- **DMA**: Use Direct Memory Access for large transfers
- **Interrupt-driven**: Use interrupts for better CPU utilization

```c
// Configure for Fast Mode (400kHz)
void I2C_SetFastMode(void) {
    TWSR = 0x00;  // Prescaler = 1
    TWBR = 12;    // For 16MHz clock: (16M/400k - 16)/2 = 12
}

// Burst read function
uint8_t I2C_ReadBurst(uint8_t dev_addr, uint16_t mem_addr, uint8_t* buffer, uint8_t length) {
    I2C_Start();
    if (I2C_Write(dev_addr << 1) != TW_MT_SLA_ACK) return 0;
    I2C_Write(mem_addr >> 8);
    I2C_Write(mem_addr & 0xFF);
    I2C_Start();
    I2C_Write((dev_addr << 1) | 1);
    
    for (uint8_t i = 0; i < length - 1; i++) {
        buffer[i] = I2C_ReadACK();
    }
    buffer[length - 1] = I2C_ReadNACK();
    I2C_Stop();
    return 1;
}
```

## Advanced Code Examples
```c
#include <avr/io.h>
#include <avr/interrupt.h>

// I2C Buffer for interrupt-driven transfers
#define I2C_BUFFER_SIZE 256
volatile uint8_t i2c_tx_buffer[I2C_BUFFER_SIZE];
volatile uint8_t i2c_rx_buffer[I2C_BUFFER_SIZE];
volatile uint8_t i2c_tx_index = 0;
volatile uint8_t i2c_rx_index = 0;
volatile uint8_t i2c_tx_length = 0;
volatile uint8_t i2c_rx_length = 0;
volatile uint8_t i2c_state = I2C_IDLE;

// I2C States
#define I2C_IDLE 0
#define I2C_START_SENT 1
#define I2C_ADDR_SENT 2
#define I2C_DATA_TX 3
#define I2C_DATA_RX 4
#define I2C_STOP_SENT 5

// Initialize I2C with interrupts
void I2C_MasterInitInterrupt(uint32_t freq) {
    TWSR = 0x00;
    TWBR = ((F_CPU / freq) - 16) / 2;
    TWCR = (1 << TWEN) | (1 << TWIE);  // Enable TWI and interrupt
    sei();
}

// Start interrupt-driven transfer
void I2C_StartTransfer(uint8_t addr, uint8_t* tx_data, uint8_t tx_len, uint8_t* rx_data, uint8_t rx_len) {
    // Copy data to buffers
    for (uint8_t i = 0; i < tx_len; i++) {
        i2c_tx_buffer[i] = tx_data[i];
    }
    i2c_tx_length = tx_len;
    i2c_rx_length = rx_len;
    i2c_tx_index = 0;
    i2c_rx_index = 0;
    
    // Send START condition
    i2c_state = I2C_START_SENT;
    TWCR = (1 << TWINT) | (1 << TWSTA) | (1 << TWEN) | (1 << TWIE);
}

// I2C Interrupt Service Routine
ISR(TWI_vect) {
    switch (TWSR & 0xF8) {
        case TW_START:
        case TW_REP_START:
            // Send address
            TWDR = i2c_tx_buffer[i2c_tx_index++];
            TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWIE);
            i2c_state = I2C_ADDR_SENT;
            break;
            
        case TW_MT_SLA_ACK:
        case TW_MT_DATA_ACK:
            if (i2c_tx_index < i2c_tx_length) {
                // Send next data byte
                TWDR = i2c_tx_buffer[i2c_tx_index++];
                TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWIE);
                i2c_state = I2C_DATA_TX;
            } else if (i2c_rx_length > 0) {
                // Switch to read mode
                TWCR = (1 << TWINT) | (1 << TWSTA) | (1 << TWEN) | (1 << TWIE);
                i2c_state = I2C_START_SENT;
            } else {
                // Send STOP
                TWCR = (1 << TWINT) | (1 << TWSTO) | (1 << TWEN);
                i2c_state = I2C_IDLE;
            }
            break;
            
        case TW_MR_SLA_ACK:
            // Address acknowledged, prepare to receive
            if (i2c_rx_length == 1) {
                TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWIE);  // NACK for last byte
            } else {
                TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWEA) | (1 << TWIE);  // ACK
            }
            i2c_state = I2C_DATA_RX;
            break;
            
        case TW_MR_DATA_ACK:
            // Received data byte with ACK
            i2c_rx_buffer[i2c_rx_index++] = TWDR;
            if (i2c_rx_index < i2c_rx_length - 1) {
                TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWEA) | (1 << TWIE);
            } else {
                TWCR = (1 << TWINT) | (1 << TWEN) | (1 << TWIE);  // NACK for last byte
            }
            break;
            
        case TW_MR_DATA_NACK:
            // Received last data byte
            i2c_rx_buffer[i2c_rx_index++] = TWDR;
            TWCR = (1 << TWINT) | (1 << TWSTO) | (1 << TWEN);
            i2c_state = I2C_IDLE;
            // Transfer complete - handle in main code
            break;
            
        default:
            // Error handling
            TWCR = (1 << TWINT) | (1 << TWSTO) | (1 << TWEN);
            i2c_state = I2C_IDLE;
            break;
    }
}

// Example usage
int main(void) {
    I2C_MasterInitInterrupt(400000);  // Fast mode
    
    uint8_t tx_data[3] = {0x00, 0x00, 0xAA};  // Address + data
    uint8_t rx_data[2];
    
    // Write then read from EEPROM
    I2C_StartTransfer(0x50, tx_data, 3, NULL, 0);  // Write
    while (i2c_state != I2C_IDLE);  // Wait for completion
    
    uint8_t read_addr[2] = {0x00, 0x00};
    I2C_StartTransfer(0x50, read_addr, 2, rx_data, 2);  // Read
    while (i2c_state != I2C_IDLE);  // Wait for completion
    
    while(1) {
        // Main loop
    }
    
    return 0;
}
