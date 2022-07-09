# Target processor architecture.
ARCH ?= x86_64

# This is the name that our final kernel executable will have.
# Change as needed.
override KERNEL := kernel.elf

# Convenience macro to reliably declare overridable command variables.
define DEFAULT_VAR =
    ifeq ($(origin $1),default)
        override $(1) := $(2)
    endif
    ifeq ($(origin $1),undefined)
        override $(1) := $(2)
    endif
endef

# It is highly recommended to use a custom built cross toolchain to build a kernel.
# We are only using "cc" as a placeholder here. It may work by using
# the host system's toolchain, but this is not guaranteed.
$(eval $(call DEFAULT_VAR,CC,cc))

# User controllable CFLAGS.
CFLAGS ?= -O2 -g -Wall -Wextra -Wpedantic -pipe

# User controllable linker flags. We set none by default.
LDFLAGS ?=

# Internal C flags that should not be changed by the user.
override INTERNALCFLAGS := \
    -I.                    \
    -std=c11               \
    -ffreestanding         \
    -fno-stack-protector   \
    -fno-stack-check       \
    -fno-pie               \
    -fno-pic               \
    -MMD

# Internal linker flags that should not be changed by the user.
override INTERNALLDFLAGS :=     \
    -nostdlib                   \
    -static                     \
    -Wl,-z,max-page-size=0x1000 \
    -Wl,-T,linker.ld

# Set archtecture specific variables (and check that the architecture is supported).
ifeq ($(ARCH),x86_64)
    override INTERNALCFLAGS += \
        -m64                   \
        -march=x86-64          \
        -mabi=sysv             \
        -mno-80387             \
        -mno-mmx               \
        -mno-sse               \
        -mno-sse2              \
        -mno-red-zone          \
        -mcmodel=kernel
    override INTERNALLDFLAGS +=    \
        -Wl,-m,elf_x86_64          \
        -Wl,--oformat=elf64-x86-64
else
    $(error Architecture $(ARCH) not supported)
endif

# Use find to glob all *.c files in the directory and extract the object names.
override CFILES := $(shell find ./ -type f -name '*.c')
override OBJ := $(CFILES:.c=.o)
override HEADER_DEPS := $(CFILES:.c=.d)

# Default target.
.PHONY: all
all: $(KERNEL)

limine.h:
	curl https://raw.githubusercontent.com/limine-bootloader/limine/trunk/limine.h -o $@

# Link rules for the final kernel executable.
$(KERNEL): $(OBJ)
	$(CC) $(OBJ) $(LDFLAGS) $(INTERNALLDFLAGS) -o $@

# Compilation rules for *.c files.
-include $(HEADER_DEPS)
%.o: %.c limine.h
	$(CC) $(CFLAGS) $(INTERNALCFLAGS) -c $< -o $@

# Remove object files and the final executable.
.PHONY: clean
clean:
	rm -rf $(KERNEL) $(OBJ) $(HEADER_DEPS)

.PHONY: distclean
distclean: clean
	rm -f limine.h
