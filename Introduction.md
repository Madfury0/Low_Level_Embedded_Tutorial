### **Week 1, Day 1: Introduction to Embedded Systems and Registers**  
**Objective**: Understand the fundamentals of embedded systems, memory-mapped I/O, and direct register manipulation in C.  

---

### **1. Embedded Systems Overview**  
#### **Key Concepts**:  
1. **Embedded Systems**:  
   - Specialized computing systems designed to perform dedicated tasks (e.g., microwave controllers, car ECUs).  
   - **Microcontrollers (MCUs)**:  
     - Integrate CPU, memory (Flash, RAM), and peripherals (GPIO, UART, ADC) on a single chip.  
     - Examples: STM32, AVR (Arduino), PIC.  
   - **Microprocessors**:  
     - General-purpose CPUs (e.g., Intel Core, AMD Ryzen) requiring external peripherals.  

2. **C Programming for Firmware**:  
   - **`volatile` Keyword**:  
     - Informs the compiler that a variable’s value can change unexpectedly (e.g., hardware registers).  
     - Prevents compiler optimizations that might skip reading/writing to memory-mapped registers.  
     ```c
     volatile uint32_t *reg = (volatile uint32_t*)0x40020000; // GPIO register  
     ```  
   - **`const` Keyword**:  
     - Marks pointers as read-only (e.g., fixed hardware addresses).  
     ```c
     const volatile uint32_t *UART_DR = (volatile uint32_t*)0x4000C000; // UART data register  
     ```  
   - **Pointers**:  
     - Directly access memory addresses (critical for register manipulation).  
     ```c
     uint32_t *ptr = (uint32_t*)0x20000000; // Points to SRAM address 0x20000000  
     ```  

---

### **2. Registers and Memory-Mapped I/O**  
#### **Key Concepts**:  
1. **Registers**:  
   - Special memory locations that control hardware behavior (e.g., turn LEDs on/off, read sensor data).  
   - Example: **GPIO** (General-Purpose Input/Output) registers:  
     - Configure pins as input/output.  
     - Set/clear output values.  

2. **Memory-Mapped I/O**:  
   - Peripherals (GPIO, UART, ADC) are accessed like memory.  
   - Each peripheral has a **base address** (defined in the MCU datasheet).  
   - Example: STM32F4 GPIOA registers:  
     - Base address: `0x40020000`  
     - `GPIOA_MODER` (mode register): Offset `0x00` → Address `0x40020000`.  
     - `GPIOA_ODR` (output data register): Offset `0x14` → Address `0x40020014`.  

---

### **3. Lab: Reading a Microcontroller Datasheet**  
#### **Steps to Find GPIO Register Addresses** (STM32F4 Example):  
1. **Locate the GPIO Section**:  
   - In the STM32F4 Reference Manual, search for "GPIO registers."  
2. **Find the Base Address**:  
   - GPIOA starts at `0x40020000`.  
3. **Calculate Register Addresses**:  
   - `GPIOA_MODER` = Base + Offset = `0x40020000 + 0x00 = 0x40020000`.  
   - `GPIOA_ODR` = Base + Offset = `0x40020000 + 0x14 = 0x40020014`.  

---

### **4. Homework: Configuring a GPIO Pin**  
**Task**: Write C code to configure **PA5** (Pin 5 of GPIO Port A) as an output pin using direct register access.  

#### **Solution Code** (STM32F4 Example):  
```c
#include <stdint.h>

int main(void) {
    // 1. Enable GPIOA Clock (RCC AHB1ENR Register)
    volatile uint32_t *RCC_AHB1ENR = (volatile uint32_t*)0x40023830;
    *RCC_AHB1ENR |= (1 << 0); // Bit 0 enables GPIOA

    // 2. Configure PA5 as Output (GPIOA_MODER Register)
    volatile uint32_t *GPIOA_MODER = (volatile uint32_t*)0x40020000;
    *GPIOA_MODER &= ~(0x3 << 10); // Clear bits 10-11 (PA5)
    *GPIOA_MODER |=  (0x1 << 10); // Set to Output (01)

    // 3. Turn on PA5 (GPIOA_ODR Register)
    volatile uint32_t *GPIOA_ODR = (volatile uint32_t*)0x40020014;
    *GPIOA_ODR |= (1 << 5); // Set bit 5 (PA5)

    while(1);
    return 0;
}
```  

#### **Explanation**:  
1. **Clock Enable**:  
   - The `RCC_AHB1ENR` register enables GPIOA’s clock (required to use the peripheral).  
2. **Pin Mode**:  
   - `GPIOA_MODER` uses 2 bits per pin. `01` sets PA5 to output mode.  
3. **Output Data**:  
   - `GPIOA_ODR`’s bit 5 controls PA5’s output state.  

---

### **5. Key Takeaways**  
- **`volatile`** is mandatory for hardware registers to prevent compiler optimizations.  
- **Memory-mapped I/O** allows peripherals to be controlled via pointers.  
- **Registers** are configured using bitwise operations (`|`, `&`, `~`, `<<`).  

### **Common Pitfalls**:  
- Forgetting to enable the peripheral clock.  
- Incorrect bitmask calculations (e.g., using `0x1` instead of `0x3` for 2-bit fields).  

---

**Next**: Day 2 will cover bitwise math and advanced register manipulation.
