ifeq ($(strip $(DEVKITPPC)),)
$(error "Set DEVKITPPC in your environment.")
endif

PREFIX = $(DEVKITPPC)/bin/powerpc-eabi-

CFLAGS = -mcpu=750 -m32 -mlittle -mhard-float -mno-eabi -mno-sdata
CFLAGS += -ffreestanding -ffunction-sections -fdata-sections
CFLAGS += -I../baselibc/include
CFLAGS += -Wall -Wextra -O2 -g
ASFLAGS =
LDFLAGS = -mcpu=750 -m32 -mlittle -n -nostartfiles -nodefaultlibs -Wl,-gc-sections

DEFINES =  -DWITH_STDIO -DBASELIBC_INTERNAL -D__BIG_ENDIAN_SYSTEM__=1
LDFLAGS += -nostartfiles -nodefaultlibs -L./baselibc -L./gccle

LDSCRIPT = of.ld
TARGET = stage2.elf

FILES = $(wildcard source/*.S) $(wildcard source/*.c)
OBJSx = $(FILES:source/%.S=build/%.o)
OBJS = $(OBJSx:source/%.c=build/%.o)
LIBS = -lcbase -lgcc

AR = $(PREFIX)ar
AS = $(PREFIX)as
CC = $(PREFIX)gcc
CXX = $(PREFIX)g++
LD = $(PREFIX)ld
OBJCOPY = $(PREFIX)objcopy
RANLIB = $(PREFIX)ranlib
STRIP = $(PREFIX)strip

ifeq ($(NOMAPFILE),)
LDFLAGS += -Wl,-Map,$(TARGET).map
endif

ifneq ($(LDSCRIPT),)
LDFLAGS += -Wl,-T$(LDSCRIPT)
endif

DEPDIR = .deps

all: $(TARGET)
	@[ -d $(DIR_BUILD) ] || mkdir $(DIR_BUILD)

$(TARGET): $(OBJS)
	@echo "  LINK      $@"
	@$(CC) $(LDFLAGS) $(OBJS) $(LIBS) -o $@
	@cp $(TARGET) $(TARGET:%.elf=%_unstripped.elf)
	@strip $(TARGET)

ifneq ($(LDSCRIPT),)
$(TARGET): $(LDSCRIPT)
endif

build/%.o: source/%.c
	@echo "  COMPILE   $<"
	@mkdir -p $(DEPDIR)
	@$(CC) $(CFLAGS) $(DEFINES) -Wp,-MMD,$(DEPDIR)/$(*F).d,-MQ,"$@",-MP -c $< -o $@

build/%.o: source/%.s
	@echo "  ASSEMBLE  $<"
	@$(CC) $(CFLAGS) $(DEFINES) $(ASFLAGS) -c $< -o $@

build/%.o: source/%.S
	@echo "  ASSEMBLE  $<"
	@$(CC) $(CFLAGS) $(DEFINES) $(ASFLAGS) -c $< -o $@

clean:
	rm -rf $(DEPDIR)
	rm -f $(TARGET) $(TARGET:%.elf=%_unstripped.elf) $(TARGET).map $(OBJS)

-include $(DEPDIR)/*

.PHONY: clean
