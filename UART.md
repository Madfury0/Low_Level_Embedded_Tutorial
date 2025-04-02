### **Week 2, Day 1: UART Communication**  
**Objective**: Learn to configure UART for asynchronous serial communication using direct register access, implement polling and interrupt-driven data transmission/reception, and debug via UART.  

---

### **1. UART Fundamentals**  
#### **Key Concepts**:  
1. **UART Protocol**:  
   - Asynchronous, full-duplex serial communication.  
   - **Frame format**: Start bit (1), data bits (5-9), parity bit (optional), stop bits (1-2).  
   - **Baud Rate**: Speed (bits/second), e.g., 9600, 115200.  

2. **UART Registers** (STM32F4 USART2 Example):  
   - **Base Address**: `0x40004400` (USART2).  
   - **Data Register (DR)**: `USART2_BASE + 0x04` – Read/write data.  
   - **Status Register (SR)**: `USART2_BASE + 0x00` – Flags (TXE, RXNE, BUSY).  
   - **Baud Rate Register (BRR)**: `USART2_BASE + 0x08` – Sets baud rate divisor.  
   - **Control Registers (CR1/CR2/CR3)**: Enable UART, interrupts, and configure settings.  

3. **Baud Rate Calculation**:  
   - **Formula**:  
     \[
     \text{BRR} = \frac{f_{\text{clock}}}{16 \times \text{baud rate}}
     \]  
   - Example: For \( f_{\text{clock}} = 16\,\text{MHz} \) and 115200 baud:  
     \[
     \text{BRR} = \frac{16,000,000}{16 \times 115200} \approx 8.68 \rightarrow \text{BRR} = 0x8B \, (\text{8 integer, 11 fractional})
     \]  

---

### **2. Polling vs. Interrupt-Driven UART**  
1. **Polling**:  
   - CPU actively waits for flags (e.g., `TXE` or `RXNE`).  
   - Simple but blocks execution.  

2. **Interrupt-Driven**:  
   - CPU handles data in the background.  
   - Requires enabling interrupts in **CR1** and **NVIC**.  

---

### **3. Lab: Transmit "Hello World" via Polling**  
**Task**: Use USART2 to send "Hello World" at 115200 baud.  

#### **Code Steps** (STM32F4):  
1. **Enable Clocks**:  
   ```c
   // Enable GPIOA (PA2=TX, PA3=RX) and USART2 clocks
   RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
   RCC->APB1ENR |= RCC_APB1ENR_USART2EN;
   ```  

2. **Configure GPIO Pins** (Alternate Function AF7):  
   ```c
   // PA2 (TX) as alternate function
   GPIOA->MODER   |=  GPIO_MODER_MODER2_1;          // Alternate mode
   GPIOA->AFR[0]  |=  (7 << (2 * 4));               // AF7 for USART2_TX
   ```  

3. **Configure UART**:  
   ```c
   // Set baud rate (16 MHz clock, 115200 baud)
   USART2->BRR = 0x8B; // 8.68 → BRR = 0x08B (8 << 4) | 11
   USART2->CR1 |= USART_CR1_TE | USART_CR1_UE;      // Enable TX & UART
   ```  

4. **Transmit Data**:  
   ```c
   char msg[] = "Hello World\r\n";
   for (int i = 0; msg[i] != '\0'; i++) {
       while (!(USART2->SR & USART_SR_TXE)); // Wait until TX buffer empty
       USART2->DR = msg[i];
   }
   ```  

---

### **4. Homework: UART Receive with Interrupts**  
**Task**: Modify the code to receive characters via UART and echo them back using interrupts.  

#### **Solution Code**:  
1. **Enable RX and Interrupts**:  
   ```c
   // Enable RX and RXNE interrupt in CR1
   USART2->CR1 |= USART_CR1_RE | USART_CR1_RXNEIE;  
   
   // Enable USART2 interrupt in NVIC
   NVIC_EnableIRQ(USART2_IRQn);
   ```  

2. **Interrupt Handler**:  
   ```c
   void USART2_IRQHandler(void) {
       if (USART2->SR & USART_SR_RXNE) { // Check RX flag
           char data = USART2->DR;       // Read received data
           while (!(USART2->SR & USART_SR_TXE)); // Wait for TX ready
           USART2->DR = data;            // Echo back
       }
   }
   ```  

---

### **5. Key Takeaways**  
- **Baud Rate**: Must match on transmitter/receiver.  
- **Status Flags**: Always check `TXE` before sending and `RXNE` before reading.  
- **Interrupts**: Efficient for non-blocking I/O but require careful flag management.  

### **Common Pitfalls**:  
- Forgetting to enable the UART clock (`USART_CR1_UE`).  
- Incorrect GPIO alternate function configuration.  
- Not clearing status flags in interrupts (automatically cleared on read/write in STM32).  

---

### **6. Real-World Example: Debugging with UART**  
Use UART to print sensor data or debug messages:  
```c
void debug_print(char *str) {
    for (int i = 0; str[i] != '\0'; i++) {
        while (!(USART2->SR & USART_SR_TXE));
        USART2->DR = str[i];
    }
}
```  

---

**Next**: Day 2 will cover SPI protocol and sensor interfacing.
