# STM32-Digital-Clock

A precise digital clock built on the STM32 microcontroller platform. This project leverages the micro's internal Real-Time Clock (RTC) peripheral for accurate timekeeping and interfaces with an SSD1306 OLED display via the I2C protocol to render a real-time, low-power user interface.

#Hardware Components:

Microcontroller: STM32F103C8T6, Display: 0.96-inch SSD1306 OLED (128x64 resolution), USB: ST Link V2

#Toolchain:

STM32CubeMX - Used for graphical pinout configuration, clock tree setup, and generating the baseline initialization C code for the I2C and RTC peripherals. 

STM32CubeIDE (version 2.1.1) - Used for writing the core clock logic, managing the OLED driver files, compiling the C codebase, and debugging/flashing the application via the ST-LINK interface.

#System Architecture & Features
1. Internal RTC Peripheral
Instead of relying on software delays or standard timers—which introduce cumulative drift—this project configures the dedicated internal Hardware RTC.
Driven by a 32.768 kHz Crystal (LSE) to ensure accuracy even during low-power modes.
Handles time (Hours, Minutes, Seconds) and date tracking automatically at the hardware level.

2. I2C Display Interfacing
Configured the STM32 I2C peripheral in Master Mode (Standard 100 kHz or Fast Mode 400 kHz).
Implemented a lightweight SSD1306 driver to transmit frame buffer data efficiently, keeping memory overhead minimal.


#Project Structure

Core/
Inc/ ---Header Files (.h)

fonts.h --- Custom font character maps for the OLED

gpio.h / i2c.h / rtc.h --- Peripheral initialization headers

ssd1306.h --- SSD1306 OLED command & function declarations

ssd1306_defines.h --- Screen dimensions and register macro definitions

main.h --- Global macros and pin definitions

Src/ --- Source Files (.c)

main.c --- Main application entry point & superloop

fonts.c --- Font pixel arrays for drawing text/numbers

gpio.c / i2c.c / rtc.c ---Peripheral control & configuration implementations

ssd1306.c --- Custom display driver (I2C pixel buffer manipulation)

stm32f1xx_it.c --- Hardware interrupt service routines (ISR)

Drivers/ --- STM32F1xx HAL (Hardware Abstraction Layer) libraries

Startup/ --- Assembly startup code for MCU boot sequences

clock.ioc --- STM32CubeMX graphical configuration project file

STM32F103C8TX_FLASH.ld --- Linker script for memory mapping (Flash/SRAM bounds)


#Steps to Build the Project

--Pin Configuration in MX
1. In systems core -> sys -> change debug to serial wire   {Turns PA13 and PA14 pin green}
also, sys -> rcc -> set LSE to cystal ceramic resonator
2. In Timer, RTC -> Activate calendar and clock (in mode) [rtc out and tamper should remain disable] {PC 14 and 15 turn green}
also, in the clock configuration panel, change lse connectivity from lsi to lse
3. Same in rtc, in configuration, set date and time (any)
4. In connectivity, change i2c disable to i2c {PB6 and PB7 will turn green}
5. Generate code, ensure it is stm32cube file in the toolchain section
6. Open project (in IDE)

--Software Implementaion in IDE

Once the initialization code is generated and the project opens in STM32CubeIDE, follow these steps to integrate the driver and implement the application logic:

1. Add the Display Driver Files:
   * Copy ssd1306.c, ssd1306_defines.h, ssd1306.h, fonts.c, and fonts.h into your project.
   * Place the .h files inside the Core/Inc/ directory.
   * Place the .c files inside the Core/Src/ directory.

2. Include Headers & Driver Initialization:
   * Open Core/Src/main.c. 
   * In the private include section (`/* USER CODE BEGIN Includes */`), add the driver headers:
     ```c
     #include "ssd1306.h"
     #include "fonts.h"
     ```
   * Inside main(), right after the peripheral initialization functions (`MX_I2C1_Init()`, `MX_RTC_Init()`, etc.), initialize the OLED display:
     ```c
     /* USER CODE BEGIN 2 */
     SSD1306_Init();
     /* USER CODE END 2 */
     ```

3. Implement the Real-Time Update Loop:
   * Inside the `while(1)` superloop, use the RTC HAL functions to fetch the updated time and date structures from the hardware.
   * Format the retrieved data into a string buffer (e.g., using `sprintf`) and pass it to the display driver tracking functions (like `SSD1306_GotoXY()` and `SSD1306_Puts()`).
   * Call `SSD1306_UpdateScreen()` to refresh the physical OLED layout and add a small software delay (e.g., `HAL_Delay(500)`) to avoid unnecessary screen flickering.

4. Build the Project:
   * Compile the project.
   * Ensure the build finishes with `0 errors` and `0 warnings`.

5. Flash the Target Microcontroller:
   * Connect your **ST-LINK V2** debugger to your computer via USB, and connect its SWD pins (GND, CLK, DIO, 3.3V) to the corresponding debug pins on your STM32 board.
   * Click the Run icon ▶️ (or Debug icon 🪲) in the toolbar.
   * In the configuration window, confirm that the debugger interface is set to **Serial Wire (SWD)**, then click OK to flash the compiled binary onto the board.



