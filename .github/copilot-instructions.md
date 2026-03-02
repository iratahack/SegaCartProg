# Project Guidelines

## Code Style
- Language: AVR C for ATmega328p (16 MHz). See [Programmer/src/main.c](Programmer/src/main.c) for typical patterns.
- Functions are typically snake_case; macros use UPPER_CASE. See [Programmer/src/xmodem.c](Programmer/src/xmodem.c).
- Keep strings in flash using the `printf` macro that wraps `PSTR()` in [Programmer/src/main.c](Programmer/src/main.c#L9-L10).
- Prefer direct register/port manipulation for IO (DDR/PORT/PIN) as in [Programmer/src/main.c](Programmer/src/main.c#L114-L151).

## Architecture
- Core firmware lives in [Programmer/src](Programmer/src): hardware init, UI, flash ops in [Programmer/src/main.c](Programmer/src/main.c).
- Mapper abstraction is `Mapper_t` in [Programmer/src/mappers.h](Programmer/src/mappers.h) with SEGA and Iratahack implementations in [Programmer/src/mappers.c](Programmer/src/mappers.c).
- Serial XMODEM transfer logic is in [Programmer/src/xmodem.c](Programmer/src/xmodem.c); UART and timing are in [Programmer/src/uart.c](Programmer/src/uart.c) and [Programmer/src/timer.c](Programmer/src/timer.c).
- CRC helpers are in [Programmer/src/CRC32.c](Programmer/src/CRC32.c).

## Build and Test
- Toolchain: `gcc-avr`, `avr-libc`, `avrdude`.
- Build firmware: `cd Programmer/src && make` (outputs `SMSFlasher.bin`) per [Programmer/src/Makefile](Programmer/src/Makefile).
- Flash device: `cd Programmer/src && make download` (uses `avrdude` on `/dev/ttyUSB0` at 115200) per [Programmer/src/Makefile](Programmer/src/Makefile#L35-L36).
- 74HC595 test firmware: `cd Programmer/74hc595 && make` and `make download` per [Programmer/74hc595/Makefile](Programmer/74hc595/Makefile).
- Runtime serial UI is at 2,000,000 baud (8N1) per [Programmer/src/uart.c](Programmer/src/uart.c).

## Project Conventions
- RAM is limited (ATmega328p 2 KB); use `PSTR()`/`PROGMEM` tables for constants (see [Programmer/src/xmodem.c](Programmer/src/xmodem.c) and [Programmer/src/CRC32.c](Programmer/src/CRC32.c)).
- Address bus is SPI-driven with 74HC595 and latched via `RCLK` (PC0); control signals `_CE/_RD/_WR` are on PC1-PC3. See [Programmer/src/main.c](Programmer/src/main.c#L12-L15).
- Data bus uses PD2-PD7 and PB0-PB1 with explicit input/output switching; follow the existing pin mapping in [Programmer/src/main.c](Programmer/src/main.c#L114-L137).
- Mapper detection expects SEGA slot equality vs Iratahack slot differences; keep `translateAddress()` and detection logic consistent with [Programmer/src/mappers.c](Programmer/src/mappers.c).

## Integration Points
- Host transfer uses XMODEM; example flows are in [Programmer/README.md](Programmer/README.md#L39-L58).
- Flash command sequences follow SST/AMD unlock/program/erase conventions in [Programmer/src/main.c](Programmer/src/main.c).

## Security
- No authentication on the serial menu; the firmware can erase/program attached flash directly. Be cautious with any new commands that change flash state.
