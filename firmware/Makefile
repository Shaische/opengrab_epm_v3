#
# Copyright (c) 2015 Zubax Robotics, zubax.com
# Distributed under the MIT License, available in the file LICENSE.
# Author: Pavel Kirienko <pavel.kirienko@zubax.com>
#

CPPSRC := $(wildcard src/*.cpp)                  \
          $(wildcard src/*/*.cpp)

CSRC   := $(wildcard lpc_chip_11cxx_lib/src/*.c) \
          $(wildcard src/sys/*.c)

DEF = -DFW_VERSION_MAJOR=1                       \
      -DFW_VERSION_MINOR=0                       \
      -DHW_VERSION_MAJOR=3

INC = -Isrc                                      \
      -isystem lpc_chip_11cxx_lib/inc

ifdef BOARD_OLIMEX_LPC_P11C24
    DEF += -DBOARD_OLIMEX_LPC_P11C24
endif

# ProDropper support
PRODROPPER ?= 0
ifneq ($(PRODROPPER),0)
    $(info Building for ProDropper)
    DEF += -DPRODROPPER=1
endif

#
# UAVCAN library
#

DEF += -DUAVCAN_TINY=1 -DUAVCAN_CPP_VERSION=UAVCAN_CPP11

include libuavcan/libuavcan/include.mk
CPPSRC += $(LIBUAVCAN_SRC)
INC += -I$(LIBUAVCAN_INC)

include libuavcan/libuavcan_drivers/lpc11c24/driver/include.mk
CPPSRC += $(LIBUAVCAN_LPC11C24_SRC)
INC += -I$(LIBUAVCAN_LPC11C24_INC)

$(info $(shell $(LIBUAVCAN_DSDLC) $(UAVCAN_DSDL_DIR)))
INC += -Idsdlc_generated

#
# Git commit hash
#

GIT_HASH := $(shell git rev-parse --short HEAD)
ifeq ($(words $(GIT_HASH)),1)
    DEF += -DGIT_HASH=0x$(GIT_HASH)
endif

#
# Build configuration
#

BUILDDIR = build
OBJDIR = $(BUILDDIR)/obj
DEPDIR = $(BUILDDIR)/dep

DEF += -DNDEBUG -DCHIP_LPC11CXX -DCORE_M0 -DTHUMB_NO_INTERWORKING -U__STRICT_ANSI__

FLAGS = -mthumb -mcpu=cortex-m0 -mno-thumb-interwork -flto -Os -g3 -Wall -Wextra -Werror -Wundef -ffunction-sections \
        -fdata-sections -fno-common -fno-exceptions -fno-unwind-tables -fno-stack-protector -fomit-frame-pointer \
        -Wfloat-equal -Wconversion -Wsign-conversion -Wmissing-declarations

C_CPP_FLAGS = $(FLAGS) -MD -MP -MF $(DEPDIR)/$(@F).d

CFLAGS = $(C_CPP_FLAGS) -std=c99

CPPFLAGS = $(C_CPP_FLAGS) -pedantic -std=c++14 -fno-rtti -fno-threadsafe-statics

LDFLAGS = $(FLAGS) -nodefaultlibs -lc -lgcc -nostartfiles -Tlpc11c24.ld -Xlinker --gc-sections \
          -Wl,-Map,$(BUILDDIR)/output.map

# Link with nano newlib. Other toolchains may not support this option, so it can be safely removed.
LDFLAGS += --specs=nano.specs

COBJ   = $(addprefix $(OBJDIR)/, $(notdir $(CSRC:.c=.o)))
CPPOBJ = $(addprefix $(OBJDIR)/, $(notdir $(CPPSRC:.cpp=.o)))
OBJ = $(COBJ) $(CPPOBJ)

VPATH = $(sort $(dir $(CSRC)) $(dir $(CPPSRC)))

ELF = $(BUILDDIR)/firmware.elf
BIN = $(BUILDDIR)/firmware.bin
HEX = $(BUILDDIR)/firmware.hex

#
# Rules
#

TOOLCHAIN ?= arm-none-eabi-
CC   = $(TOOLCHAIN)gcc
CPPC = $(TOOLCHAIN)g++
AS   = $(TOOLCHAIN)gcc
LD   = $(TOOLCHAIN)g++
CP   = $(TOOLCHAIN)objcopy
SIZE = $(TOOLCHAIN)size

all: $(OBJ) $(ELF) $(BIN) $(HEX) size

$(OBJ): | $(BUILDDIR)

$(BUILDDIR):
	@mkdir -p $(BUILDDIR)
	@mkdir -p $(DEPDIR)
	@mkdir -p $(OBJDIR)

$(BIN): $(ELF)
	@echo
	$(CP) -O binary $(ELF) $@

$(HEX): $(ELF)
	@echo
	$(CP) -O ihex $(ELF) $@

$(ELF): $(OBJ)
	@echo
	$(LD) $(OBJ) $(LDFLAGS) -o $@

$(COBJ): $(OBJDIR)/%.o: %.c
	@echo
	$(CC) -c $(DEF) $(INC) $(CFLAGS) $< -o $@

$(CPPOBJ): $(OBJDIR)/%.o: %.cpp
	@echo
	$(CPPC) -c $(DEF) $(INC) $(CPPFLAGS) $< -o $@

clean:
	rm -rf $(BUILDDIR) dsdlc_generated

size: $(ELF)
	@if [ -f $(ELF) ]; then echo; $(SIZE) $(ELF); echo; fi;

.PHONY: all clean size $(BUILDDIR)

# Include the dependency files, should be the last of the makefile
-include $(shell mkdir $(DEPDIR) 2>/dev/null) $(wildcard $(DEPDIR)/*)
