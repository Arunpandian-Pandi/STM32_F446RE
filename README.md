# verify_driver

A minimal STM32F446RETx firmware project that demonstrates controlling a GPIO
pin by accessing peripheral registers directly, including the GPIO bit set/reset
register (BSRR).

## What it does

The application in `Src/main.c`:

1. Enables the GPIOA peripheral clock through the RCC register.
2. Configures PA5 as a general-purpose output.
3. Sets PA5 through BSRR, waits in a software delay loop, then resets PA5
   through BSRR and waits again. Repeating these steps blinks the pin.

### BSRR set and reset

The 32-bit BSRR register controls GPIO output pins without a read-modify-write
of the output data register (ODR):

- Bits 0-15 set the corresponding GPIO pins. Writing bit 5 (`1 << 5`) sets PA5.
- Bits 16-31 reset the corresponding GPIO pins. Writing bit 21 (`1 << (16 + 5)`)
  resets PA5.

Each write changes only the selected pin, so the set/reset operation does not
disturb other GPIO output bits. In this example, the two BSRR writes alternate
to turn PA5 on and off.

On an STM32 Nucleo-F446RE board, the user LED (LD2) is connected to PA5, so it
should blink when the firmware is running. On other hardware, connect an LED
with a suitable current-limiting resistor to PA5, or adapt the pin and GPIO
configuration for your board.

The delay is a simple busy-wait loop; its duration depends on compiler settings
and clock configuration and is not a precise timing source.

## Target and tools

- **MCU:** STM32F446RETx
- **IDE/build system:** STM32CubeIDE managed project
- **Toolchain:** GNU Arm Embedded (`arm-none-eabi-`)
- **Linker script:** `STM32F446RETX_FLASH.ld` for the Flash build

## Build and run

1. Open STM32CubeIDE.
2. Import this directory using **File > Import > Existing Projects into
   Workspace**.
3. Select the `verify_driver` project and build the **Debug** or **Release**
   configuration.
4. Connect a compatible STM32F446RE board over ST-LINK.
5. Start a debug session or use the IDE's Run command to program the target.

The STM32CubeIDE project configuration and startup assembly are included in
this repository. CMSIS and STM32F4 HAL driver sources are also present under
`Drivers/`; the current `main.c` example uses direct register access rather
than HAL initialization or HAL GPIO functions.

## Project layout

| Path | Purpose |
| --- | --- |
| `Src/main.c` | Application entry point and GPIO toggle example |
| `Src/syscalls.c`, `Src/sysmem.c` | C library system-call and heap support |
| `Startup/` | STM32F446RETx startup code and interrupt vector |
| `Drivers/CMSIS/` | ARM CMSIS core and STM32 device definitions |
| `Drivers/STM32F4xx_HAL_Driver/` | STM32F4 HAL and low-level driver sources |
| `Drivers/BSP/` | ST board support package sources |
| `Inc/` | Application include directory |
| `STM32F446RETX_FLASH.ld` | Linker script for the Flash target |
| `STM32F446RETX_RAM.ld` | Linker script for the RAM target |

## Notes

- The GPIO register addresses and pin configuration in `Src/main.c` are
  specific to the STM32F4 peripheral layout and the selected PA5 output.
- The project includes ST and ARM software components with their own license
  terms. Refer to the license files under `Drivers/` for those components.
