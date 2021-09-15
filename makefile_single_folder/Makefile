# Version 1.0
#main : main.c func1.c func2.c
#	gcc main.c func1.c func2.c -o main

# Verison 2.0
#main : main.o func1.o func2.o
#	gcc main.o func1.o func2.o -o main
#
#main.o : main.c
#	gcc -c main.c
#
#func1.o : func1.c
#	gcc -c func1.c
#
#func2.o : func2.c
#	gcc -c func2.c

# Verison 3.0
#obj := main.o func1.o func2.o
#target := main
#CC = gcc
#
#$(target) : $(obj)
#	$(CC) $^ -o $@
#
#%.o : %.c
#	$(CC) -c $< -o $@
#
#clean:
#	rm *.o $(target) -f

# Version 4.0
src := $(wildcard ./*.c)
obj := $(patsubst %.c, %.o, $(src))
target := main
CC := gcc

$(target) : $(obj)
	$(CC) -o $@ $^
	
%.o : %.c
	$(CC) -c $^

debug :
	@echo $(src)
	@echo $(obj)

clean:
	rm *.o $(target) -f