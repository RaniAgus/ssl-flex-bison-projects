#    Copyright (C) 2021  Agustín Ranieri
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.

# Agregar bibliotecas necesarias acá (por ejemplo, -lm para incluir <math.h>)
LIBS+=-lfl -ly

CC=gcc
LEX=flex
YACC=bison

EXT=out

ifeq ($(OS),Windows_NT)
	EXT=exe
	C_FLAGS+=-m32
endif

C_FLAGS+=-Isrc -Iobj -Wall
Y_FLAGS+=-v -d
L_FLAGS+=

Y_SRCS=$(wildcard src/*.y)
L_SRCS=$(wildcard src/*.l)
C_SRCS=$(wildcard src/*.c)
H_SRCS=$(wildcard src/*.h)

Y_OBJS=$(Y_SRCS:src/%.y=obj/%.tab.o)
L_OBJS=$(L_SRCS:src/%.l=obj/%.yy.o)
C_OBJS=$(C_SRCS:src/%.c=obj/%.o)

BINS=$(Y_SRCS:src/%.y=%.$(EXT))

all: $(BINS)

clean:
	rm -rf obj
	rm -f *.$(EXT)

debug: Y_FLAGS += --debug
debug: L_FLAGS += -d
debug: C_FLAGS += -DDEBUG -g
debug: clean all

# Executable
%.$(EXT): obj obj/%.tab.o obj/%.yy.o $(C_OBJS) $(H_SRCS)
	$(CC) $(C_FLAGS) -o "$@" $(filter %.o, $^) $(LIBS)
	@echo "=================[ Build $@ done! ]================="

# 1. Create directories
obj:
	mkdir -p $@

# 2. Bison objects
obj/%.tab.o: obj/%.tab.c
	$(CC) $(C_FLAGS) -c -o "$@" $<

obj/%.tab.c: src/%.y
	$(YACC) $(Y_FLAGS) -o"$@" $<

# 3. Flex objects
obj/%.yy.o: obj/%.yy.c
	$(CC) $(C_FLAGS) -c -o "$@" $<

obj/%.yy.c: src/%.l obj/%.tab.o
	$(LEX) $(L_FLAGS) -o"$@" $<

# 4. C objects
obj/%.o: src/%.c
	$(CC) $(C_FLAGS) -c -o "$@" $<

.PHONY: all clean debug

.PRECIOUS: $(BINS) $(C_OBJS) $(Y_OBJS) $(Y_OBJS:%.o=%.c) $(L_OBJS) $(L_OBJS:%.o=%.c)

.SECONDARY: $(BINS) $(C_OBJS) $(Y_OBJS) $(Y_OBJS:%.o=%.c) $(L_OBJS) $(L_OBJS:%.o=%.c)