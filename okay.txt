#include <stdlib.h>
#include <stdio.h>

#define MEMORY_SIZE 4096
#define WORD_SIZE 16
#define REGISTER_COUNT 16

unsigned short memory[MEMORY_SIZE];
unsigned short registers[REGISTER_COUNT];
unsigned short pc = 0;
unsigned short flags = 0;

// Define opcodes
#define ADD_OPCODE 0x0
#define SUB_OPCODE 0x1
// Add more opcodes as needed

// Define instructions
void add(unsigned short dest, unsigned short src1, unsigned short src2) {
    registers[dest] = registers[src1] + registers[src2];
}

void sub(unsigned short dest, unsigned short src1, unsigned short src2) {
    registers[dest] = registers[src1] - registers[src2];
}

// Add functions for other instructions

void loadInstructions(const char* filename) {
    FILE* file = fopen(filename, "r");
    if (file == NULL) {
        fprintf(stderr, "Error: Unable to open file %s.\n", filename);
        exit(1);
    }

    unsigned short instruction;
    int i = 0;

    while (fscanf(file, "%hx", &instruction) != EOF) {
        if (i >= MEMORY_SIZE) {
            fprintf(stderr, "Error: Too many instructions. Exceeding MEMORY_SIZE.\n");
            fclose(file);
            exit(1);
        }
        memory[i++] = instruction;
    }

    // Append the termination instruction if there's space
    if (i < MEMORY_SIZE) {
        memory[i] = 0xDEAD;
    }

    fclose(file);
}


void printRegisterDump() {
    for (int i = 0; i < REGISTER_COUNT; i++) {
        printf("Register  %d: 0x%04X\n", i, registers[i]);
    }
    printf("Register  PC: 0x%04X\n", pc);
}

void printMemoryDump() {
    for (int i = 0; i < MEMORY_SIZE; i += 16) {
        if (i % 2 == 0) { // Check if the last digit of the address is 0
            printf("0x%04X:", i);
            for (int j = 0; j < 8; j++) {
                printf(" [%04X]", memory[i + j]);
            }
            printf("\n");
        }
    }
}


int main(int argc, char* argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        return 1;
    }
    // Initialize memory and registers

    loadInstructions(argv[1]);

    while (1) {
        unsigned short instruction = memory[pc];
        unsigned short opcode = (instruction >> 12) & 0xF;
        unsigned short dest = (instruction >> 8) & 0xF;
        unsigned short src1 = (instruction >> 4) & 0xF;
        unsigned short src2 = instruction & 0xF;

        // Execute instructions based on opcode
        switch (opcode) {
            case ADD_OPCODE:
                add(dest, src1, src2);
                break;
            case SUB_OPCODE:
                sub(dest, src1, src2);
                break;
            // Add cases for other instructions
        }

        pc++;
        if (pc >= MEMORY_SIZE) {
            printf("");
            break;
        }
    }

    printRegisterDump();
    printMemoryDump();

    return 0;
}
