### **Week 1, Day 5: Timers, Clocks, and PWM Generation**  
**Objective**: Configure system clocks (HSI/HSE/PLL), use timers to generate precise delays, and create PWM signals via direct register access.  

---

### **1. System Clock Configuration**  
#### **Key Concepts**:  
1. **Clock Sources**:  
   - **HSI (High-Speed Internal)**: Default 8-16 MHz RC oscillator (inexact, low power).  
   - **HSE (High-Speed External)**: Crystal oscillator (e.g., 8-25 MHz, precise).  
   - **PLL (Phase-Locked Loop)**: Multiplies HSI/HSE to higher frequencies (e.g., 84 MHz on STM32F4).  

2. **Clock Tree**:  
   - Clocks feed peripherals (timers, GPIO, UART) via prescalers.  
   - Example: STM32F4 clock tree routes HSE → PLL → SYSCLK → AHB/APB buses.  

3. **RCC (Reset and Clock Control) Registers**:  
   - `RCC_CR`: Enable HSE/HSI/PLL.  
   - `RCC_CFGR`: Configure clock sources and prescalers.  

#### **Example: Configure HSE and PLL for 84 MHz SYSCLK** (STM32F4):  
```c
// 1. Enable HSE (8 MHz external crystal)  
RCC->CR |= RCC_CR_HSEON;  
while (!(RCC->CR & RCC_CR_HSERDY));  

// 2. Configure PLL (HSE as source, 84 MHz output)  
RCC->PLLCFGR = (8 << 0)  |  // PLLM = 8 (HSE / 8 = 1 MHz)  
               (168 << 6) |  // PLLN = 168 (1 MHz * 168 = 168 MHz)  
               (0 << 16)  |  // PLLP = 2 (168 MHz / 2 = 84 MHz)  
               RCC_PLLCFGR_PLLSRC_HSE;  

// 3. Enable PLL and wait for readiness  
RCC->CR |= RCC_CR_PLLON;  
while (!(RCC->CR & RCC_CR_PLLRDY));  

// 4. Switch SYSCLK to PLL  
RCC->CFGR |= RCC_CFGR_SW_PLL;  
while ((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_PLL);  
```  

---

### **2. Timer Registers and PWM**  
#### **Timer Basics**:  
- **Counter**: Increments/decrements at a clock rate.  
- **Prescaler (PSC)**: Divides input clock (e.g., 84 MHz → 1 MHz).  
- **Auto-Reload Register (ARR)**: Sets period (counter resets after reaching ARR).  
- **Capture/Compare Register (CCR)**: Sets PWM duty cycle.  

#### **PWM Modes**:  
- **Mode 1**: Counter < CCR → output high, else low.  
- **Mode 2**: Counter < CCR → output low, else high.  

---

### **3. Lab: 1Hz LED Blink with Timer Interrupts**  
**Task**: Use Timer 2 to toggle an LED every 0.5 seconds (1Hz blink).  

#### **Code Steps** (STM32F4, 84 MHz APB1 clock):  
1. **Enable Timer Clock**:  
   ```c
   RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;  
   ```  
2. **Configure Timer**:  
   ```c
   // Prescaler = 8399 (84 MHz / 8400 = 10 kHz)  
   TIM2->PSC = 8399;  
   // Auto-reload = 5000 (10 kHz / 5000 = 2 Hz → toggle every 0.5s)  
   TIM2->ARR = 5000 - 1;  
   ```  
3. **Enable Update Interrupt**:  
   ```c
   TIM2->DIER |= TIM_DIER_UIE;  
   NVIC_EnableIRQ(TIM2_IRQn);  
   ```  
4. **Start Timer**:  
   ```c
   TIM2->CR1 |= TIM_CR1_CEN;  
   ```  

#### **ISR**:  
```c
void TIM2_IRQHandler(void) {  
    if (TIM2->SR & TIM_SR_UIF) {  
        TIM2->SR &= ~TIM_SR_UIF; // Clear interrupt  
        GPIOA->ODR ^= GPIO_ODR_OD5; // Toggle PA5  
    }  
}  
```  

---

### **4. Homework: Adjustable PWM Duty Cycle**  
**Task**: Generate a PWM signal on PA6 (TIM3_CH1) with a duty cycle adjustable via code.  

#### **Solution Code**:  
```c
// 1. Enable GPIOA and TIM3 clocks  
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;  
RCC->APB1ENR |= RCC_APB1ENR_TIM3EN;  

// 2. Configure PA6 as Alternate Function (TIM3_CH1)  
GPIOA->MODER |= GPIO_MODER_MODER6_1; // Alternate function mode  
GPIOA->AFR[0] |= (2 << 24); // AF2 for TIM3_CH1  

// 3. Configure TIM3 for PWM  
TIM3->PSC = 8399; // 84 MHz / 8400 = 10 kHz  
TIM3->ARR = 100 - 1; // 10 kHz / 100 = 100 Hz PWM  
TIM3->CCR1 = 30; // 30% duty cycle (CCR1 = 30)  

// 4. Configure PWM Mode 1  
TIM3->CCMR1 |= TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_2; // PWM mode 1  
TIM3->CCMR1 |= TIM_CCMR1_OC1PE; // Enable preload  

// 5. Enable Output and Start Timer  
TIM3->CCER |= TIM_CCER_CC1E;  
TIM3->CR1 |= TIM_CR1_CEN;  
```  

---

### **5. Key Takeaways**  
- **Clock Configuration**: Critical for timing accuracy (use HSE/PLL for precision).  
- **Timer Math**:  
  - PWM Frequency = Timer Clock / ((PSC + 1) * (ARR + 1))  
  - Duty Cycle = (CCR / (ARR + 1)) * 100%  
- **PWM**: Use capture/compare registers to control pulse width.  

### **Common Pitfalls**:  
- Forgetting to set GPIO alternate function modes for PWM outputs.  
- Incorrect timer clock source selection (APB1 vs. APB2).  

---

**Next Week**: Week 2 dives into peripheral drivers (UART, SPI, I2C) and debugging techniques.
