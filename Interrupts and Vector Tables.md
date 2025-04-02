### **Week 1, Day 4: Interrupts and Vector Tables**  
**Objective**: Learn how interrupts work, write interrupt service routines (ISRs), and configure the vector table for real-time firmware responses.  

---

### **1. Interrupt Basics**  
#### **Key Concepts**:  
1. **Interrupt**:  
   - A hardware signal that pauses the CPU’s current task to execute an **ISR** (Interrupt Service Routine).  
   - Example: A button press triggers an interrupt to toggle an LED.  

2. **Interrupt Sources**:  
   - **External** (e.g., GPIO pin change).  
   - **Internal** (e.g., timer overflow, UART data received).  

3. **Interrupt Workflow**:  
   - Trigger → Save CPU state → Execute ISR → Restore state → Resume main program.  

4. **NVIC (Nested Vectored Interrupt Controller)**:  
   - ARM Cortex-M hardware that prioritizes and manages interrupts.  
   - **Interrupt Vector Table (IVT)**: Array of function pointers to ISRs.  

---

### **2. Vector Table Setup**  
#### **ARM Cortex-M Vector Table** (STM32F4 Example):  
- **Location**: Defined in the linker script (usually Flash address `0x00000000`).  
- **Key Entries**:  
  - `0x00000000`: Initial stack pointer.  
  - `0x00000004`: Reset handler (start of firmware).  
  - `0x00000058`: EXTI0 interrupt handler (GPIO pin 0).  
  - `0x00000068`: TIM2 interrupt handler (Timer 2).  

#### **Custom ISR Mapping**:  
```c
// Define the vector table in code (simplified)  
void (*vector_table[])(void) __attribute__((section(".isr_vector"))) = {  
    (void(*)(void))0x20010000, // Initial stack pointer  
    Reset_Handler,             // Reset handler  
    // ...  
    EXTI0_IRQHandler,          // EXTI Line0 interrupt  
    TIM2_IRQHandler            // Timer2 interrupt  
};  
```  

---

### **3. Writing an ISR in C**  
#### **Syntax and Rules**:  
1. Use `__attribute__((interrupt))` for Cortex-M (optional in some toolchains).  
2. **Clear the interrupt flag** to avoid retriggering.  
3. Keep ISRs short (e.g., set flags, update buffers).  

#### **Example: GPIO External Interrupt (PA0)**  
```c
// ISR for EXTI0 (PA0)  
void EXTI0_IRQHandler(void) {  
    // 1. Check if EXTI0 triggered the interrupt  
    if (EXTI->PR & EXTI_PR_PR0) {  
        // 2. Clear the pending bit (write 1 to clear)  
        EXTI->PR = EXTI_PR_PR0;  
        // 3. Toggle LED on PA5  
        GPIOA->ODR ^= GPIO_ODR_OD5;  
    }  
}  
```  

---

### **4. Configuring Interrupts**  
#### **Steps to Enable GPIO Interrupt (STM32F4)**:  
1. **Enable GPIO Clock**:  
   ```c
   RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN; // Enable GPIOA  
   ```  
2. **Configure GPIO Pin as Input**:  
   ```c
   GPIOA->MODER &= ~GPIO_MODER_MODER0; // PA0 as input  
   ```  
3. **Enable SYSCFG Clock** (for EXTI mapping):  
   ```c
   RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;  
   ```  
4. **Map PA0 to EXTI0**:  
   ```c
   SYSCFG->EXTICR[0] |= SYSCFG_EXTICR1_EXTI0_PA;  
   ```  
5. **Configure EXTI for Falling Edge**:  
   ```c
   EXTI->FTSR |= EXTI_FTSR_TR0; // Trigger on falling edge  
   EXTI->IMR |= EXTI_IMR_MR0;   // Unmask EXTI0  
   ```  
6. **Enable NVIC Interrupt**:  
   ```c
   NVIC_EnableIRQ(EXTI0_IRQn); // Enable EXTI0 in NVIC  
   ```  

---

### **5. Lab: Trigger an Interrupt on GPIO Pin Change**  
**Task**: Connect a button to PA0 and an LED to PA5. Write code to toggle the LED when the button is pressed.  

#### **Full Code Example**:  
```c
#include "stm32f4xx.h"  

int main(void) {  
    // 1. Enable Clocks  
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;  
    RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;  

    // 2. Configure PA5 (LED) as output  
    GPIOA->MODER |= GPIO_MODER_MODER5_0; // Output mode  

    // 3. Configure PA0 (Button) as input  
    GPIOA->MODER &= ~GPIO_MODER_MODER0;  

    // 4. Map PA0 to EXTI0  
    SYSCFG->EXTICR[0] |= SYSCFG_EXTICR1_EXTI0_PA;  

    // 5. Configure EXTI for falling edge  
    EXTI->FTSR |= EXTI_FTSR_TR0;  
    EXTI->IMR |= EXTI_IMR_MR0;  

    // 6. Enable NVIC interrupt  
    NVIC_EnableIRQ(EXTI0_IRQn);  

    while (1);  
}  

// ISR for EXTI0  
void EXTI0_IRQHandler(void) {  
    if (EXTI->PR & EXTI_PR_PR0) {  
        EXTI->PR = EXTI_PR_PR0; // Clear interrupt  
        GPIOA->ODR ^= GPIO_ODR_OD5; // Toggle LED  
    }  
}  
```  

---

### **6. Homework: Timer Overflow Interrupt**  
**Task**: Configure Timer 2 (TIM2) to trigger an interrupt every 1 second and toggle an LED.  

#### **Solution Steps**:  
1. **Enable TIM2 Clock**:  
   ```c
   RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;  
   ```  
2. **Set Prescaler and Auto-Reload**:  
   - Assume system clock = 16 MHz.  
   - Prescaler = 16000 → 16 MHz / 16000 = 1 kHz.  
   - Auto-reload = 1000 → 1 kHz / 1000 = 1 Hz.  
   ```c
   TIM2->PSC = 16000 - 1;  
   TIM2->ARR = 1000 - 1;  
   ```  
3. **Enable Update Interrupt**:  
   ```c
   TIM2->DIER |= TIM_DIER_UIE;  
   ```  
4. **Enable NVIC Interrupt**:  
   ```c
   NVIC_EnableIRQ(TIM2_IRQn);  
   ```  
5. **Start Timer**:  
   ```c
   TIM2->CR1 |= TIM_CR1_CEN;  
   ```  

#### **ISR for TIM2**:  
```c
void TIM2_IRQHandler(void) {  
    if (TIM2->SR & TIM_SR_UIF) {  
        TIM2->SR &= ~TIM_SR_UIF; // Clear interrupt  
        GPIOA->ODR ^= GPIO_ODR_OD5; // Toggle LED  
    }  
}  
```  

---

### **7. Key Takeaways**  
- **ISRs** must be fast and **non-blocking**.  
- **Vector Table** maps interrupts to handler functions.  
- **NVIC** manages interrupt priorities and enables/disables.  

### **Common Pitfalls**:  
- Forgetting to clear the interrupt flag (causes infinite interrupts).  
- Not enabling peripheral clocks (RCC registers).  

---

**Next**: Day 5 will cover timers, PWM, and clock configuration.
