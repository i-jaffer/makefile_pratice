DIR_INC = ./inc
DIR_SRC = ./src
DIR_OBJ = ./obj

SRC := $(wildcard $(DIR_SRC)/*.c)
OBJ := $(patsubst %.c, ${DIR_OBJ}/%.o, $(notdir $(SRC)))
TARGET := main
CC := gcc
CFLAGS := -I $(DIR_INC)

$(TARGET) : $(OBJ)
	$(CC) $(OBJ) -o $@

$(DIR_OBJ)/%.o : $(DIR_SRC)/%.c
	$(CC) -c $< $(CFLAGS) -o $@

debug :
	@echo $(SRC)
	@echo $(OBJ)
	@echo $(CC) $(OBJ) -o $(TARGET)

clean :
	-rm $(TARGET) $(OBJ)