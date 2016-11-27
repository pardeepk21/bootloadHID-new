# Name: Makefile
# Project: bootloadHID
# Author: Pardeep Kumar
# Creation Date: 2016-11-27
# Tabsize: 4
# Copyright: (c) 2016 by OBJECTIVE DEVELOPMENT Software GmbH
# License: GNU GPL v2 (see License.txt)
# This Revision: $Id$

###############################################################################
# Configure the following variables according to your AVR. The example below
# is for an ATMega168. Program the device with
#     make fuse    # to set the clock generator, boot section size etc.
#     make flash   # to load the boot loader into flash

DEVICE = atmega168
BOOTLOADER_ADDRESS = 3800
F_CPU = 12000000
FUSEL = 0xdf
FUSEH = 0xd5
FUSEE = 0x00
# Fuse low byte:
# 0xdf = 1 1 0 1   1 1 1 1 
#        ^ ^ \ /   \--+--/
#        | |  |       +------- CKSEL 3..0 (external >8M crystal)
#        | |  +--------------- SUT 1..0 (Start Up Time PWRDN/RESET- 16K CK/14CK + 0ms)
#        | +------------------ CKOUT (Clock output on PORTB0 - Unprogrammed - i.e. No Clock Output from PORTB0)
#        +-------------------- CKDIV8 (Divide Clock by 8 - Unprogrammed i.e. We're Not Dividing Clock by 8)
# Fuse high byte:
# 0xd5 = 1 1 0 1   0 1 0 1 
#        ^ ^ ^ ^   ^ \-+-/
#        | | | |   |   +------ BODLEVEL 2..0 (2.7V)
#        | | | |   + --------- EESAVE (preserve EEPROM over chip erase)
#        | | | +-------------- WDTON (WDT not always on)
#        | | +---------------- NOT USED / DISABLED
#        | +------------------ DWEN (Debug wire disabled)
#        +-------------------- RSTDISBL (reset pin is enabled)
# Fuse high byte:
# 0x00 = 0 0 0 0   0 0 0 0 <-- BOOTRST (boot reset vector at 0x3800) 
#        \--+--/   ^ ^ ^------ BOOTSZ0   
#           |      | +-------- BOOTSZ1
#		    |      + --------- NOT USED / DISABLED
#           +----------------- NOT USED / DISABLED
###############################################################################

AVRDUDE = avrdude -c stk500v2 -P avrdoper -p $(DEVICE)

LDFLAGS += -Wl,--relax,--gc-sections -Wl,--section-start=.text=$(BOOTLOADER_ADDRESS)

# Omit -fno-* options when using gcc 3, it does not support them.
COMPILE = avr-gcc -Wall -Os -fno-move-loop-invariants -fno-tree-scev-cprop -fno-inline-small-functions -Iusbdrv -I. -mmcu=$(DEVICE) -DF_CPU=$(F_CPU) -DDEBUG_LEVEL=0 # -DTEST_MODE
# NEVER compile the final product with debugging! Any debug output will
# distort timing so that the specs can't be met.

OBJECTS =  usbdrv/usbdrvasm.o usbdrv/oddebug.o main.o


# symbolic targets:
all:	main.hex

.c.o:
	$(COMPILE) -c $< -o $@

.S.o:
	$(COMPILE) -x assembler-with-cpp -c $< -o $@
# "-x assembler-with-cpp" should not be necessary since this is the default
# file type for the .S (with capital S) extension. However, upper case
# characters are not always preserved on Windows. To ensure WinAVR
# compatibility define the file type manually.

.c.s:
	$(COMPILE) -S $< -o $@

flash:	all
	$(AVRDUDE) -U flash:w:main.hex:i

readflash:
	$(AVRDUDE) -U flash:r:read.hex:i

fuse:
	$(AVRDUDE) -U hfuse:w:$(FUSEH):m -U lfuse:w:$(FUSEL):m -U efuse:w:$(FUSEE):m

read_fuses:
	$(UISP) --rd_fuses

ifeq ($(OS),Windows_NT)
clean:
		del /f main.hex main.bin *.o usbdrv\*.o main.s usbdrv\oddebug.s usbdrv\usbdrv.s
else
UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Linux)
clean:
		rm -f main.hex main.bin *.o usbdrv/*.o main.s usbdrv/oddebug.s usbdrv/usbdrv.s
endif
endif

# file targets:
main.bin:	$(OBJECTS)
	$(COMPILE) -o main.bin $(OBJECTS) $(LDFLAGS)

ifeq ($(OS),Windows_NT)
main.hex:	main.bin
	del /f main.hex main.eep.hex
	avr-objcopy -j .text -j .data -O ihex main.bin main.hex
	avr-size main.hex
else
UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Linux)
main.hex:	main.bin
	rm -f main.hex main.eep.hex
	avr-objcopy -j .text -j .data -O ihex main.bin main.hex
	avr-size main.hex
endif
endif

disasm:	main.bin
	avr-objdump -d main.bin

cpp:
	$(COMPILE) -E main.c
