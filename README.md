@deepseek_ai
### **Week 1: Foundations of Low-Level Firmware**
**Day 1: Introduction to Embedded Systems and Registers**  
- **Topics**:  
  - Embedded systems overview (microcontrollers vs. microprocessors).  
  - C programming for firmware: volatile, const, pointers, and memory addresses.  
  - Understanding registers (GPIO, clocks) and memory-mapped I/O.  
- **Lab**: Read a microcontroller datasheet (e.g., ARM Cortex-M or AVR) to locate register addresses.  
- **Homework**: Write code to configure a GPIO pin as output using direct register access (no libraries).  

**Day 2: Bitwise Math and Manipulation**  
- **Topics**:  
  - Bitwise operators: `&`, `|`, `^`, `~`, `<<`, `>>`.  
  - Setting, clearing, and toggling bits.  
  - Bitmasks and bitfields.  
- **Lab**: Solve bitwise problems (e.g., reverse bits, count set bits).  
- **Homework**: Write a function to configure specific bits in a register without affecting others.  

**Day 3: Memory and Peripherals**  
- **Topics**:  
  - Memory organization (Flash, SRAM, EEPROM).  
  - Understanding peripheral registers (e.g., UART, ADC).  
  - Volatile keyword and compiler optimizations.  
- **Lab**: Map a peripheral register (e.g., UART) using pointers.  
- **Homework**: Write code to initialize a UART peripheral by setting baud rate and control registers.  

**Day 4: Interrupts and Vector Tables**  
- **Topics**:  
  - Interrupt basics (ISRs, NVIC/IVT).  
  - Writing interrupt handlers in C.  
  - Enabling/disabling interrupts.  
- **Lab**: Trigger an interrupt on GPIO pin change.  
- **Homework**: Configure a timer overflow interrupt.  

**Day 5: Timers and Clocks**  
- **Topics**:  
  - System clock configuration (PLL, HSI, HSE).  
  - Timer registers (prescaler, auto-reload, counter).  
  - PWM generation.  
- **Lab**: Generate a 1Hz LED blink using timer interrupts.  
- **Homework**: Write code to generate a PWM signal with adjustable duty cycle.  

**Week 1 Project**: Build a "Blinking LED System" using GPIO, timers, and interrupts. Use direct register access for all configurations.  

---

### **Week 2: Peripheral Drivers and Debugging**  
**Day 6: UART Communication**  
- **Topics**:  
  - UART registers (TX, RX, status flags).  
  - Polling vs. interrupt-driven UART.  
- **Lab**: Transmit "Hello World" via UART using polling.  
- **Homework**: Implement UART receive with interrupts.  

**Day 7: SPI Protocol**  
- **Topics**:  
  - SPI registers (clock polarity, data order).  
  - Full-duplex communication.  
- **Lab**: Read/write to an SPI sensor (e.g., temperature sensor).  
- **Homework**: Write a function to send 16-bit data over SPI.  

**Day 8: I2C Protocol**  
- **Topics**:  
  - I2C registers (start/stop conditions, ACK/NACK).  
  - Addressing and multi-byte transactions.  
- **Lab**: Read data from an I2C EEPROM.  
- **Homework**: Implement I2C bus scanning.  

**Day 9: Analog-to-Digital Conversion (ADC)**  
- **Topics**:  
  - ADC registers (sampling time, resolution).  
  - Single-channel and scan modes.  
- **Lab**: Read analog voltage from a potentiometer.  
- **Homework**: Configure ADC to use DMA for data transfer.  

**Day 10: Debugging Techniques**  
- **Topics**:  
  - Using debuggers (JTAG/SWD).  
  - LED/serial debugging.  
  - Watchdogs and fault handlers.  
- **Lab**: Trigger a hard fault and analyze the stack.  
- **Homework**: Fix a provided buggy ISR.  

**Week 2 Project**: Build a "Sensor Data Logger" that reads analog/digital sensor data and transmits it via UART.  

---

### **Week 3: Advanced Firmware Design**  
**Day 11: Power Management**  
- **Topics**:  
  - Sleep modes (Run, Sleep, Stop).  
  - Clock gating.  
- **Lab**: Measure current consumption in sleep mode.  
- **Homework**: Wake the system from sleep via GPIO interrupt.  

**Day 12: Direct Memory Access (DMA)**  
- **Topics**:  
  - DMA registers (channels, transfer modes).  
  - Memory-to-peripheral transfers.  
- **Lab**: Use DMA to transfer ADC data to memory.  
- **Homework**: Implement UART-to-memory DMA.  

**Day 13: Real-Time Operating Systems (RTOS) Basics**  
- **Topics**:  
  - Tasks, schedulers, and context switching.  
  - Critical sections and semaphores.  
- **Lab**: Create two tasks toggling LEDs at different rates.  
- **Homework**: Fix a priority inversion scenario.  

**Day 14: Firmware Optimization**  
- **Topics**:  
  - Code size vs. speed tradeoffs.  
  - Inline assembly for critical sections.  
  - Compiler flags and linker scripts.  
- **Lab**: Profile code execution time using timers.  
- **Homework**: Optimize a provided delay function.  

**Day 15: Final Project Kickoff**  
- **Topics**:  
  - Project requirements and design review.  
  - Code modularity and documentation.  
- **Final Project**: Build a "Smart Thermostat" combining GPIO, ADC, UART, timers, interrupts, and power management.  

---

### **Final Project Requirements**:  
1. Read temperature via ADC or I2C sensor.  
2. Display data on UART or LCD.  
3. Control an actuator (e.g., fan) via PWM.  
4. Enter low-power mode when idle.  
5. Handle faults using watchdog timers.  

This course balances theory, hands-on labs, and projects to build proficiency in bare-metal firmware development. Adjust the microcontroller (e.g., STM32, AVR, PIC) based on student hardware access.
