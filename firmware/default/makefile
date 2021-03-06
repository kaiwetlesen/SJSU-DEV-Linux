# Allow settiing a project name from the environment, default to firmware.
# Only affects the name of the generated binary.
# TODO: Set this from the directory this makefile is stored in
PROJ 			?= firmware

# Points where the SJSUOne librarie sources are located
LIB_DIR			?= lib
# Affects what DBC is generated for SJSUOne board
ENTITY 			?= DBG

# IMPORTANT: Must be accessible via the PATH variable!!!
CC              = arm-none-eabi-gcc
CPPC            = arm-none-eabi-g++
OBJDUMP         = arm-none-eabi-objdump
SIZEC           = arm-none-eabi-size
OBJCOPY         = arm-none-eabi-objcopy

# Internal build directories
OBJ_DIR			= obj
BIN_DIR			= bin
DBC_DIR			= _can_dbc

CFLAGS = -mcpu=cortex-m3 \
    -mthumb -Os -fmessage-length=0 \
    -ffunction-sections -fdata-sections \
    -Wall -Wshadow -Wlogical-op \
    -Wfloat-equal -DBUILD_CFG_MPU=0 \
    -fabi-version=0 \
    -fno-exceptions \
    -I"$(LIB_DIR)/" \
    -I"$(LIB_DIR)/newlib" \
    -I"$(LIB_DIR)/L0_LowLevel" \
    -I"$(LIB_DIR)/L1_FreeRTOS" \
    -I"$(LIB_DIR)/L1_FreeRTOS/include" \
    -I"$(LIB_DIR)/L1_FreeRTOS/portable" \
    -I"$(LIB_DIR)/L1_FreeRTOS/portable/no_mpu" \
    -I"$(LIB_DIR)/L2_Drivers" \
    -I"$(LIB_DIR)/L2_Drivers/base" \
    -I"$(LIB_DIR)/L3_Utils" \
    -I"$(LIB_DIR)/L3_Utils/tlm" \
    -I"$(LIB_DIR)/L4_IO" \
    -I"$(LIB_DIR)/L4_IO/fat" \
    -I"$(LIB_DIR)/L4_IO/wireless" \
    -I"$(LIB_DIR)/L5_Application" \
    -I"L2_Drivers" \
    -I"L3_Utils" \
    -I"L4_IO" \
    -I"L5_Application" \
    -I"$(DBC_DIR)"

DBCBUILD        	= $(DBC_DIR)/generated_can.h
LIBRARIES			= $(shell find "$(LIB_DIR)" -name '*.c' -o -name '*.cpp')
SOURCES				= $(shell find . -name '*.c' -o -name '*.cpp')
COMPILABLES 		= $(LIBRARIES) $(SOURCES)

# $(patsubst %.cpp,%.o, LIST) 		: Replace .cpp -> .o
# $(patsubst %.c,%.o, LIST)			: Replace .c -> .o
# $(patsubst src/%,%, LIST) 		: Replace src/path/file.o -> path/file.o
# $(addprefix $(OBJ_DIR)/, LIST) 	: Add OBJ DIR to path (path/file.o -> obj/path/file.o)
OBJECT_FILES 		= $(addprefix $(OBJ_DIR)/, $(patsubst %.c,%.o, $(patsubst %.cpp,%.o, $(COMPILABLES))))
EXECUTABLE			= $(BIN_DIR)/$(PROJ).elf
HEX					= $(EXECUTABLE:.elf=.hex)
LIST				= $(EXECUTABLE:.elf=.lst)
SIZE				= $(EXECUTABLE:.elf=.siz)
MAP					= $(EXECUTABLE:.elf=.map)

.PHONY: all clean

all: $(DBC_DIR) $(OBJ_DIR) $(BIN_DIR) $(SIZE) $(LIST) $(HEX)

cleaninstall: clean all install

print-%  : ; @echo $* = $($*)

$(HEX): $(EXECUTABLE)
	@echo 'Invoking: Cross ARM GNU Create Flash Image'
	$(OBJCOPY) -O ihex "$<" "$@"
	@echo 'Finished building: $@'
	@echo ' '

$(SIZE): $(EXECUTABLE)
	@echo 'Invoking: Cross ARM GNU Print Size'
	$(SIZEC) --format=berkeley "$<"
	@echo 'Finished building: $@'
	@echo ' '

$(LIST): $(EXECUTABLE)
	@echo 'Invoking: Cross ARM GNU Create Listing'
	$(OBJDUMP) --source --all-headers --demangle --line-numbers --wide "$<" > "$@"
	@echo 'Finished building: $@'
	@echo ' '

$(EXECUTABLE): $(DBCBUILD) $(OBJECT_FILES)
	@echo 'Invoking: Cross ARM C++ Linker'
	mkdir -p "$(dir $@)"
	$(CPPC) -mcpu=cortex-m3 -mthumb -Os -fmessage-length=0 -ffunction-sections -fdata-sections -Wall -Wshadow -Wlogical-op -Wfloat-equal -T $(LIB_DIR)/loader.ld -nostartfiles -Xlinker --gc-sections -Wl,-Map,"$(MAP)" -specs=nano.specs -o "$@" $(OBJECT_FILES)
	@echo 'Finished building target: $@'
	@echo ' '

$(OBJ_DIR)/%.o: %.cpp
	@echo 'Building file: $<'
	@echo 'Invoking: Cross ARM C++ Compiler'
	mkdir -p "$(dir $@)"
	$(CPPC) $(CFLAGS) -std=gnu++11 -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@)" -c -o "$@" "$<"
	@echo 'Finished building: $<'
	@echo ' '

$(OBJ_DIR)/%.o: %.c
	@echo 'Building file: $<'
	@echo 'Invoking: Cross ARM C Compiler'
	mkdir -p "$(dir $@)"
	$(CC) $(CFLAGS) -std=gnu11 -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@)" -c -o "$@" "$<"
	@echo 'Finished building: $<'
	@echo ' '

$(OBJ_DIR)/%.o: $(LIB_DIR)/%.cpp
	@echo 'Building file: $<'
	@echo 'Invoking: Cross ARM C++ Compiler'
	mkdir -p "$(dir $@)"
	$(CPPC) $(CFLAGS) -std=gnu++11 -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@)" -c -o "$@" "$<"
	@echo 'Finished building: $<'
	@echo ' '

$(OBJ_DIR)/%.o: $(LIB_DIR)/%.c
	@echo 'Building file: $<'
	@echo 'Invoking: Cross ARM C Compiler'
	mkdir -p "$(dir $@)"
	$(CC) $(CFLAGS) -std=gnu11 -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@)" -c -o "$@" "$<"
	@echo 'Finished building: $<'
	@echo ' '

$(DBCBUILD):
	python "$(LIB_DIR)/_can_dbc/dbc_parse.py" -i "$(LIB_DIR)/_can_dbc/243.dbc" -s $(ENTITY) > $(DBCBUILD)

$(DBC_DIR):
	mkdir -p $(DBC_DIR)

$(OBJ_DIR):
	@echo 'Creating Objects Folder: $<'
	mkdir $(OBJ_DIR)

$(BIN_DIR):
	@echo 'Creating Binary Folder: $<'
	mkdir $(BIN_DIR)

clean:
	rm -fR $(OBJ_DIR) $(BIN_DIR) $(DBC_DIR)

install: all
	# pyFlash uses environment variable HEXFILE to make the magic happen
	pyFlash-Hyperload.py $(HEX) 
