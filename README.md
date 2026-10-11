# 528HW2Anderson

## Review Questions
### 1) What is a hardware timer? What kinds of tasks do embedded systems commonly use hardware timers for?
A hardware timer is a register that is set up to increment on each clock cycle, allowing it to record how many clock cycles have passed and therefore how much time. They are commonly used to control PWM output, time when an external signal occurred, or send out regularly timed interrupts.

### 2) What is a clock signal used for in embedded systems? List the primary clock sources available on the MSP432P401R microcontroller and briefly describe each clock source.
Clock signals are used to increment timers and synchronize operations. The MSP432P401R microcontroller has two primary clock sources, the subsystem master clock and an auxiliary clock. Both are built into the microcontroller and the master clock has a higher frequency.

### 3) Consider a 16-bit timer (e.g. Timer_A configured in up mode). If it counts upward, what is the maximum count value before it rolls over? Show your work.
0000_0000_0000_0000 -> (2^16) states -> starts a 0 so max value = states -1 = **65535**

### 4) If a 24-bit timer counts upward from zero at 10 MHz, what is its overflow period? Show your work.
Overflows in 2^24 cycles. Completes 10^7 cycles per second. Cancel cycles by dividing by frequency -> [(2^24) / (10^7)][cycles*seconds/cycles] = **1.6777216 seconds**

### 5) Describe the SysTick timer on the MSP432P401R microcontroller. Provide two example applications for SysTick in embedded systems. Which clock sources can the SysTick timer use and how can they be selected?
SysTick is a 24-bit count-down timer that can be used as a Real-Time Operating System (RTOS) tick timer or as a simple counter. It generates an interrupt when it reaches 0 and reloads to a programmed value.    
Some examples include: A simple counter used to measure time to completion and time used / A high-speed alarm timer using the system clock.    
It can use the core clock (set 1 in CLKSOURCE) or an external clock source.

### 6) How many registers does the SysTick timer have on the MSP432P401R microcontroller and what is the purpose of each? Name them and provide a short description per register.
SysTick has 4 registers:
**STCSR**: Control and Status Register - Enables SysTick features.
**STRVR**: Reload Value Register - Defines the value which is counted down from.
**STCVR**: Current Value Register - Defines the current value of SysTick.
**STCR**: Calibration Value Register - Enables software to scale to any required speed using divide and multiply.

### 7) Describe the Timer_A peripheral on the MSP432P401R microcontroller. List the main operating modes of Timer_A and briefly explain each.
The Timer_A peripheral is 16-bit timer with up to seven capture/compare registers. It has four modules and it is capable of generating interrupts from each of the capture/compare registers.
Timer_A has 4 modes:
**Stop**: Halts the timer
**Up**: The timer repeatedly counts from zero to the value of TAxCCR0
**Continous**: The timer repeatedly counts from zero to 0xFFFF
**Up/Down**: The timer repeatedly counts from zero up to the value of TAxCCR0 and back down to zero

### 8) Briefly state the purpose of the following Timer_A registers.
**TAxCTL**
Configures many aspects of the Timer_A module.
**TAxCCTLn**
Controls many aspects of how the Timer_A module captures.
**TAxR**
Tracks the current value of the Timer_A module.
**TAxCCRn**
Controls whether to compare or capture.
**TaxIV**
Holds the interrupt flags.
**TAxEX0**
Selects the divider for the input clock.

### 9) What is PWM? Provide at least three example applications that use PWM in embedded systems.
PWM is Pulse Width Modulation and it regulates average delivered power by switching a digital signal between on and off states. The output power is directly proportional to its duty cycle—the fraction of each period the signal remains high.    
PWM can be used to control the state of systems such as motors, reduce the intensity of an LED, or create audio frequencies.

### 10) How many Timer_A modules are available and how many PWM outputs can each module provide?
There are 4 modules and each has 7 PWM modules.

### 11) Describe the steps needed to initialize P2.4 as a PWM output driven by Timer_A0 CCR1 (TA0.1).
Configure GPIO for the pin. Set the timer Capture/Compare register to the specified period_constant. Set clock frequency with CTL and EX0 registers. Configure the output mode with the CCTL registers. Set the duty cycle of the PWM signal with the CCR register. Select the clock source with the CTL register.

Include C code such that:
**The P2.4 pin is configured to use its primary module function (Timer_A output)**
P2->SEL0 |= 0x10;
P2->SEL1 &= ~(0x10);
P2->DIR |= 0x10;
**SMCLK is selected as the clock source for Timer_A**
TIMER_A0->CTL &= ~(0x0300);
TIMER_A0->CTL |= (0x0200);
**The timer clock is divided by 2**
TIMER_A0->CTL &= ~(0xC0);
TIMER_A0->CTL |= (0x40);
**Up/down mode is selected**
TIMER_A0->CTL |= (0x30);
**The output mode is configured as Toggle / Reset by setting the appropriate bits of the OUTMOD field in the CCTL[1] register**
TIMER_A0->CCTL[1] &= ~(0xE0);
TIMER_A0->CCTL[1] |= (0x40);
**The PWM signal has a period of 20 ms and a duty cycle of 50%**
TIMER_A0->CCR[1] = 0x00FF;

### 12) Describe the steps needed to configure Timer_A0 to generate a periodic interrupt every 2 ms with an interrupt priority level of 2. Note that Timer_A0 has an interrupt number of 8.
Configure GPIO for the pin. Stop the Timer by clearing the mode control (MC) bits in the CTL register. Select SMCLK (12 MHz) as the timer clock source by setting the TASSEL bits and setting ID. Enable interrupt request of the corresponding Capture/Compare interrupt flag in the CCTL register using the CCIE bit. Store the period in the CCR0 register. Sets TAIDEX. Set the interrupt priority level to 2 using the IPR2 register of NVIC. Enable Interrupt 8 in NVIC by setting Bit 8 of the ISER register. Set the TACLR bit and enable Timer A using the mode control (MC) bits in the CTL register.

Include C code such that:
**SMCLK is selected as the clock source for Timer_A with a prescaler value of 1 (which means the ID field will be 00b)**
TIMER_A0->CTL &= ~(0xC0);
**The TAIDEX field in the TAxEX0 register has a value of 000b (divide by 1)**
TIMER_A0->EX0 = 0x0000
**Up mode is selected**
TIMER_A0->CTL &= ~(0x30);
TIMER_A0->CTL |= (0x10);
