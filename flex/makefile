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
LIBS+=-lfl

CC=gcc
LEX=flex

EXT=out

ifeq ($(OS),Windows_NT)
	EXT=exe
	C_FLAGS+=-m32
endif

C_FLAGS+=-Isrc -Iobj -Wall
L_FLAGS+=

L_OBJS=$(patsubst src/%.l, obj/%.yy.o, $(wildcard src/*.l))
C_OBJS=$(patsubst src/%.c, obj/%.o, $(wildcard src/*.c))

BINS=$(patsubst src/%.l, bin/%.$(EXT), $(wildcard src/*.l))

all: $(BINS)

clean:
	rm -rf bin obj

debug: Y_FLAGS += --debug
debug: L_FLAGS += -d
debug: C_FLAGS += -DDEBUG -g
debug: clean all

# Executable
bin/%.$(EXT): bin obj obj/%.yy.o $(C_OBJS)
	$(CC) $(C_FLAGS) -o "$@" $(filter %.o, $^) $(LIBS)
	@echo "=================[ Build $@ done! ]================="

# 1. Create directories
bin obj:
	mkdir -p $@

# 2. C objects
obj/%.o: src/%.c src/%.h
	$(CC) $(C_FLAGS) -c -o "$@" $<

obj/%.o: src/%.c
	$(CC) $(C_FLAGS) -c -o "$@" $<

# 3. Flex objects
obj/%.yy.o: obj/%.yy.c
	$(CC) $(C_FLAGS) -c -o "$@" $<

obj/%.yy.c: src/%.l
	$(LEX) $(L_FLAGS) -o"$@" $<

.PHONY: all clean debug

.PRECIOUS: $(BINS) $(C_OBJS) $(L_OBJS) $(L_OBJS:%.o=%.c)

.SECONDARY: $(BINS) $(C_OBJS) $(L_OBJS) $(L_OBJS:%.o=%.c)