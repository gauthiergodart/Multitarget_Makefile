#******************************************************************************
# Copyright (C) 2017 by Alex Fosdick - University of Colorado
#
# Redistribution, modification or use of this software in source or binary
# forms is permitted as long as the files maintain this copyright. Users are 
# permitted to modify this and use it to learn about the field of embedded
# software. Alex Fosdick and the University of Colorado are not liable for any
# misuse of this material. 
#
#*****************************************************************************

#------------------------------------------------------------------------------
# Makefile for multitarget build system
#
# Use: make [TARGET] [PLATFORM-OVERRIDES]
#
# Build Targets:
# 	  <FILE>.i		Builds <FILE>.i preprocessed file 
#	  <FILE>.asm 	Builds <FILE>.asm assembly file 
#	  <FILE>.o 		Builds <FILE>.o object file
#	  compile-all 	Compile all object files, but do not link. Also generate
#					dependency files
#	  build 		Builds and links all the files. Also generate .map file
#					and dependency files
#	  clean 		Clear generated files
#
# Platform:
#      HOST:		Host machine, using GCC
#      MSP432:		Target system, being TI MSP432 Launchpad, 
#					using arm-none-eabi-gcc
#
#------------------------------------------------------------------------------

include sources.mk

# _______________ COMMON FLAGS ________________

# Target
TARGET = c1m2
# Compile flags
CFLAGS = $(CFLAGS_ARCH) -Wall -Werror -g -std=c99 -O0 -D$(PLATFORM) 
# Preprocessor flags
CPPFLAGS = -MT $@ -MD -MP
# Linking flags
LDFLAGS = -Wl,-Map=$(TARGET).map $(LDFLAGS_ARCH) 

# ____________ PLATFORM OVERRIDES  ____________

ifeq ($(PLATFORM),MSP432)	
	# Source files, dependency files and includes
	SRC = $(MSP_SOURCES:.c=.o)
	DEPFILES = $(MSP_SOURCES:.c=.d)
	INCLUDES = $(MSP_INCLUDES)
	# Architecture Flags
	CPU = cortex-m4
	ARCH = armv7e-m
	SPECS = nosys.specs
	# Linker Flag
	LINKER_FILE = ../msp432p401r.lds
	# Compiler flags and define
	CC = arm-none-eabi-gcc
	LD = arm-none-eabi-ld
	DUMP = arm-none-eabi-objdump
	SIZE = arm-none-eabi-size
	CFLAGS_ARCH := -mcpu=$(CPU) -mthumb -march=$(ARCH) --specs=$(SPECS) -mfloat-abi=hard -mfpu=fpv4-sp-d16
	LDFLAGS_ARCH := -T $(LINKER_FILE)	
endif

ifeq ($(PLATFORM),HOST)
	# Source files, dependency files and includes
	SRC = $(HOST_SOURCES:.c=.o)
	DEPFILES = $(HOST_SOURCES:.c=.d)
	INCLUDES = $(HOST_INCLUDES)
	# Compiler flags and define
	CC = gcc
	LD = ld
	DUMP = objdump
	SIZE = size
endif

# ___________________ RULES ___________________

# Generating .d files
$(DEPFILES):

$(TARGET).out: $(SRC)
	$(CC) $(SRC) $(CFLAGS) $(LDFLAGS) $(INCLUDES) -o $@

.PHONY: compile-all
compile-all: $(SRC)
	$(CC) -c $(CFLAGS) $(INCLUDES) $^

.PHONY: build
build: $(TARGET).out
	$(SIZE) $(TARGET).out

.PHONY: clean
clean:
	rm -f *.i *.s *.asm *.o *.d *.out *.map

%.i: %.c
	$(CC) -E $(CFLAGS) $(INCLUDES) -o $@ $^

%.asm: %.o 
	$(DUMP) -S --disassemble $< > $@

%.asm: %.out
	$(DUMP) -S --disassemble $< > $@

%.o: %.c %.d
	$(CC) $(CPPFLAGS) -c $(CFLAGS) $(INCLUDES) -o $@ $< 