### **Week 1, Day 2: Bitwise Math and Manipulation**  
**Objective**: Master bitwise operations (AND, OR, XOR, shifts) and apply them to embedded firmware tasks like setting, clearing, and toggling hardware register bits.  

---

### **1. Bitwise Operators in C**  
#### **Key Concepts**:  
1. **Bitwise Operators**:  
   - **AND (`&`)**:  
     - Clears bits: `result = value & mask`.  
     - Example: Clear bit 3: `value &= ~(1 << 3)`.  
   - **OR (`|`)**:  
     - Sets bits: `result = value | mask`.  
     - Example: Set bit 5: `value |= (1 << 5)`.  
   - **XOR (`^`)**:  
     - Toggles bits: `result = value ^ mask`.  
     - Example: Toggle bit 2: `value ^= (1 << 2)`.  
   - **NOT (`~`)**:  
     - Inverts all bits: `result = ~value`.  
   - **Left Shift (`<<`)** and **Right Shift (`>>`)**:  
     - Shift bits left/right by `n` positions.  
     - Example: Shift bit 0 to bit 4: `(1 << 0) → (1 << 4)`.  

2. **Bitmasks**:  
   - A predefined pattern of bits used to modify specific bits in a register.  
   - Example: `#define LED_PIN_MASK (1 << 5)` for GPIO pin PA5.  

3. **Bitfields**:  
   - Structs that map individual bits in a register (compiler-dependent).  
   - Example:  
     ```c
     typedef struct {
         uint32_t mode  : 2;  // 2 bits for pin mode
         uint32_t unused : 30; // Remaining bits
     } GPIO_MODER_Type;
     ```  

---

### **2. Setting/Clearing/Toggling Bits**  
#### **Applications in Firmware**:  
1. **Set a Bit** (e.g., turn on an LED):  
   ```c
   *GPIOA_ODR |= (1 << 5); // Set PA5 high  
   ```  
2. **Clear a Bit** (e.g., turn off an LED):  
   ```c
   *GPIOA_ODR &= ~(1 << 5); // Set PA5 low  
   ```  
3. **Toggle a Bit** (e.g., blink an LED):  
   ```c
   *GPIOA_ODR ^= (1 << 5); // Toggle PA5  
   ```  

4. **Read a Bit** (e.g., check a button state):  
   ```c
   uint8_t button_state = (*GPIOA_IDR >> 3) & 0x1; // Read PA3  
   ```  

---

### **3. Lab: Bitwise Problem Solving**  
#### **Exercises**:  
1. **Reverse the bits** of an 8-bit number:  
   ```c
   uint8_t reverse_bits(uint8_t num) {
       uint8_t result = 0;
       for (int i = 0; i < 8; i++) {
           result |= ((num >> i) & 1) << (7 - i);
       }
       return result;
   }
   ```  
2. **Count the number of set bits** in a 32-bit register:  
   ```c
   int count_set_bits(uint32_t reg) {
       int count = 0;
       while (reg) {
           count += reg & 1;
           reg >>= 1;
       }
       return count;
   }
   ```  

---

### **4. Homework: Safe Register Manipulation**  
**Task**: Write a function to configure **specific bits** in a register without affecting others.  

#### **Example Code**:  
```c
void set_register_bits(volatile uint32_t *reg, uint32_t mask, uint32_t value) {
    // Step 1: Clear the target bits
    *reg &= ~mask; 
    // Step 2: Set the new value
    *reg |= (value & mask); 
}

int main(void) {
    // Example: Set PA5 to output mode (01) in GPIOA_MODER
    // PA5 uses bits 10-11 (mask = 0x3 << 10)
    set_register_bits(GPIOA_MODER, (0x3 << 10), (0x1 << 10));
    return 0;
}
```  

#### **Explanation**:  
- **Step 1**: Clear the target bits using `*reg &= ~mask`.  
- **Step 2**: Apply the new value using `*reg |= (value & mask)`.  
- Ensures only the specified bits are modified.  

---

### **5. Key Takeaways**  
- **Bitwise operations** are foundational for low-level firmware (no HAL/APIs).  
- **Atomic operations** (read-modify-write) prevent race conditions in registers.  
- **Bitmasks** make code readable and reusable.  

### **Common Pitfalls**:  
- Forgetting to shift masks (e.g., `(1 << 5)` instead of `1`).  
- Using non-atomic operations in interrupts (e.g., `*reg |= x` vs. `*reg = x`).  

---

### **6. Advanced Example: GPIO Configuration**  
#### **Configure PA5 as Output and PA3 as Input** (STM32F4):  
```c
// Enable GPIOA Clock
*RCC_AHB1ENR |= (1 << 0);

// PA5: Output (01), PA3: Input (00)
volatile uint32_t *GPIOA_MODER = (volatile uint32_t*)0x40020000;
*GPIOA_MODER &= ~((0x3 << 10) | (0x3 << 6)); // Clear PA5 and PA3
*GPIOA_MODER |=  (0x1 << 10);                // Set PA5 to output
```  

---

**Next**: Day 3 will cover memory organization (Flash, SRAM) and peripheral register mapping.
