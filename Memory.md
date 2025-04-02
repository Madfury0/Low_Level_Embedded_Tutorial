### **Week 1, Day 3: Memory Organization and Peripheral Registers**  
**Objective**: Understand memory types (Flash, SRAM, EEPROM), peripheral register mapping, and the role of the `volatile` keyword in low-level firmware.  

---

### **1. Memory Organization in Embedded Systems**  
#### **Key Concepts**:  
1. **Flash Memory**:  
   - Stores firmware code and read-only data (persistent after power-off).  
   - Example: Bootloader, interrupt vector table.  
2. **SRAM (Static RAM)**:  
   - Stores runtime variables (stack, heap, global/static data).  
   - Volatile (loses data on power-off).  
3. **EEPROM**:  
   - Stores non-volatile user data (e.g., calibration values).  
   - Slower write/erase cycles than Flash.  

4. **Memory-Mapped Peripherals**:  
   - Peripherals (GPIO, UART, ADC) are controlled via registers mapped to specific memory addresses.  
   - Example: UART data register at `0x4000C000`.  

---

### **2. Peripheral Registers (UART Example)**  
#### **UART Register Map** (Generic ARM Cortex-M):  
- **Base Address**: `0x4000C000` (hypothetical UART0).  
- **Key Registers**:  
  - `UART_DR` (Data Register): Offset `0x00` → Read/write data.  
  - `UART_FR` (Flag Register): Offset `0x18` → Status flags (e.g., TX busy, RX full).  
  - `UART_IBRD` (Integer Baud Rate Divisor): Offset `0x24` → Sets baud rate.  

#### **Code Example: UART Initialization**  
```c
#include <stdint.h>

// UART0 Base Address
#define UART0_BASE 0x4000C000

// Register Pointers
volatile uint32_t *UART_DR  = (volatile uint32_t*)(UART0_BASE + 0x00);
volatile uint32_t *UART_FR  = (volatile uint32_t*)(UART0_BASE + 0x18);
volatile uint32_t *UART_IBRD = (volatile uint32_t*)(UART0_BASE + 0x24);

void UART_Init(uint32_t baud_rate) {
    // 1. Disable UART (if needed)
    // 2. Set baud rate (assumes 16 MHz system clock)
    *UART_IBRD = 16000000 / (16 * baud_rate); // Example: 9600 baud → 104
    
    // 3. Configure data format (8N1, no parity)
    // (Other registers like LCR_H would go here)
    
    // 4. Enable UART (not shown for simplicity)
}
```  

---

### **3. The `volatile` Keyword and Compiler Optimizations**  
#### **Why `volatile` Matters**:  
- **Compiler Optimizations**:  
  - Compilers may skip "unnecessary" reads/writes to memory.  
  - Example: A loop polling a register might be optimized away.  
- **Solution**:  
  - Use `volatile` to force the compiler to read/write every time.  
  ```c
  // Without volatile, this loop could be optimized to infinite loop!
  while (*UART_FR & (1 << 3)) {} // Wait until TX buffer is empty
  ```  

---

### **4. Lab: Peripheral Register Mapping**  
**Task**: Map the **UART_DR** register for UART1 (base address `0x40010000`) and transmit the character `'A'`.  

#### **Solution Code**:  
```c
volatile uint32_t *UART1_DR = (volatile uint32_t*)0x40010000;

void send_char(char c) {
    // Wait until TX buffer is empty
    while (*UART_FR & (1 << 3)) {} // Assume UART_FR is mapped
    *UART1_DR = c; // Send character
}

int main() {
    send_char('A');
    return 0;
}
```  

---

### **5. Homework: Initialize UART with Baud Rate and Control Registers**  
**Task**: Write code to configure UART0 with a baud rate of **115200** and 8N1 format (8 data bits, no parity).  

#### **Example Code**:  
```c
void UART_Init(void) {
    // Assume system clock = 50 MHz
    uint32_t baud_rate_divisor = 50000000 / (16 * 115200); // ≈27
    
    // Set integer baud rate divisor
    *UART_IBRD = baud_rate_divisor;
    
    // Configure line control register (8N1)
    volatile uint32_t *UART_LCR_H = (volatile uint32_t*)(UART0_BASE + 0x2C);
    *UART_LCR_H = (0x3 << 5); // 8-bit word length (bits 5-6 = 0b11)
}
```  

---

### **6. Key Takeaways**  
- **Memory Types**: Flash (code), SRAM (runtime data), EEPROM (user data).  
- **Peripheral Registers**: Use pointers to access them as memory addresses.  
- **`volatile`**: Critical for registers to prevent compiler optimizations.  

### **Common Pitfalls**:  
- Forgetting to enable the peripheral clock before accessing registers.  
- Incorrect baud rate calculations (depends on system clock speed).  

---

**Next**: Day 4 will cover interrupts and vector tables.
