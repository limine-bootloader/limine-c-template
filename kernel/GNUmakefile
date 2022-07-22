# This is the name that our final kernel executable will have.
# Change as needed.
override KERNEL := kernel.elf

# Convenience macro to reliably declare overridable command variables.
define DEFAULT_VAR =
    ifeq ($(origin $1), default)
        override $(1) := $(2)
    endif
    ifeq ($(origin $1), undefined)
        override $(1) := $(2)
    endif
endef

# It is highly recommended to use a custom built cross toolchain to build a kernel.
# We are only using "cc" as a placeholder here. It may work by using
# the host system's toolchain, but this is not guaranteed.
$(eval $(call DEFAULT_VAR,CC,cc))

# User controllable CFLAGS.
CFLAGS ?= -O2 -g -Wall -Wextra -Wpedantic -pipe

# User controllable nasm flags.
NASMFLAGS ?= -F dwarf -g

# User controllable linker flags. We set none by default.
LDFLAGS ?=

# User controllable preprocessor flags. We set none by default.
CPPFLAGS ?=

# Internal C flags that should not be changed by the user.
override INTERNALCFLAGS := \
    -I.                    \
    -std=c11               \
    -ffreestanding         \
    -fno-stack-protector   \
    -fno-stack-check       \
    -fno-pie               \
    -fno-pic               \
    -m64                   \
    -march=x86-64          \
    -mabi=sysv             \
    -mno-80387             \
    -mno-mmx               \
    -mno-sse               \
    -mno-sse2              \
    -mno-red-zone          \
    -mcmodel=kernel        \
    -MMD

# Internal linker flags that should not be changed by the user.
override INTERNALLDFLAGS :=     \
    -nostdlib                   \
    -static                     \
    -Wl,-z,max-page-size=0x1000 \
    -Wl,-T,linker.ld

# Internal nasm flags that should not be changed by the user.
override INTERNALNASMFLAGS := \
    -f elf64

# Use find to glob all *.c, *.S, and *.asm files in the directory and extract the object names.
override CFILES := $(shell find ./ -type f -name '*.c')
override ASFILES := $(shell find ./ -type f -name '*.S')
override NASMFILES := $(shell find ./ -type f -name '*.asm')
override OBJ := $(CFILES:.c=.o) $(ASFILES:.S=.o) $(NASMFILES:.asm=.o)
override HEADER_DEPS := $(CFILES:.c=.d) $(ASFILES:.S=.d)

# Default target.
.PHONY: all
all: $(KERNEL)

limine.h:
	curl https://raw.githubusercontent.com/limine-bootloader/limine/trunk/limine.h -o $@

# Link rules for the final kernel executable.
$(KERNEL): $(OBJ)
	$(CC) $(CFLAGS) $(INTERNALCFLAGS) $(OBJ) $(LDFLAGS) $(INTERNALLDFLAGS) -o $@

# Include header dependencies.
-include $(HEADER_DEPS)

# Compilation rules for *.c files.
%.o: %.c limine.h
	$(CC) $(CPPFLAGS) $(CFLAGS) $(INTERNALCFLAGS) -c $< -o $@

# Compilation rules for *.S files.
%.o: %.S limine.h
	$(CC) $(CPPFLAGS) $(CFLAGS) $(INTERNALCFLAGS) -c $< -o $@

# Compilation rules for *.asm (nasm) files.
%.o: %.asm
	nasm $(NASMFLAGS) $(INTERNALNASMFLAGS) $< -o $@

# Remove object files and the final executable.
.PHONY: clean
clean:
	rm -rf $(KERNEL) $(OBJ) $(HEADER_DEPS)

.PHONY: distclean
distclean: clean
	rm -f limine.h
