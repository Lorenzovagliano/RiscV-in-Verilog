## 1 Introduction

RISC-V is an open-source instruction set architecture (ISA) originally developed at the University of California, Berkeley. The main task proposed in this practical work was to include new operations and modules in a five-stage RISC-V implementation done in the hardware description language Verilog, running on the Google Colab platform. In this work, the instructions we implemented were the multiplication (`mul`), division (`div`), bitwise logical "and" with an immediate value (`andi`), and branch on equal (`beq`) operations.

## 2 Development

### 2.1 Assembly

In the Assembly section, the encodings for each of the new implemented operations were included. Regarding the `mul` and `div` instructions, we defined them as R-type operations in the instruction format. We stipulated `opcode`, `funct3`, and `funct7` values for both. The `beq` and `andi` instructions already had these pre-established values and were not altered. The following table shows the respective values for each operation:

| Instruction | Type | Opcode   | Funct3 | Funct7  |
|-------------|------|----------|--------|---------|
| mul         | R    | 0110011  | 001    | 0100011 |
| div         | R    | 0110011  | 010    | 0100011 |
| andi        | I    | 0010011  | 111    |         |
| beq         | B    | 1100011  | 000    |         |

### 2.2 Control Unit

The control unit serves to decode the fetched instruction, determining which operation should be performed and which operands are involved. Within the case that read the operation code, the opcodes for R-type and I-type instructions were already included, respectively with the tags `add` and `addi`. Thus, we included a new case to detect B-type operations, allowing the recognition of the `beq` instruction, reproduced below:

```verilog
7'b1100011: begin // beq
  aluop <= 2'b1;
  regwrite <= 1'b0;
  branch_eq <= 1'b1;
  ImmGen <= {{19{inst[31]}},inst[31], inst[7], inst[30:25], inst[11:8],1'b0};
end
```

In the case where `aluop` is `1100011`, the control unit will recognize the B-type operation, assign the values of `aluop` and `branch_eq` as 1, and `regwrite` as 0, since there is no writing to the registers. Furthermore, it will calculate the immediate value used in the displacement.

### 2.3 ALU Control and ALU

In this section of the program, some changes were made. They can be divided into three main parts:

- **`case(ctl)`**: It was necessary to add the multiplication and division operations. These operations were assigned to the values `4'd10` and `4'd11`, respectively.
- **`case(funct[3:0])`**: Used for R-type instructions like `mul` and `div`. Two new cases were created, values `4'd9` and `4'd10`, which are computed by concatenating the second most significant bit of `funct7` with `funct3`. These correspond to `mul` and `div` respectively. These then map to values `4'd10` and `4'd11` used by the ALU.
- **`case(funct[2:0])`**: Focused on `andi`. Since this is an I-type instruction, it does not use `funct7`. A mapping was added between `funct3 == 3'd7` and ALU function `4'd0`, which is the logical AND.

### 2.4 Compiling and Executing

It was observed that if the number of cycles was too low (e.g., 20), some operations would not complete. Thus, the clock value was increased to 60, which resolved these issues.

## 3 Tests

All tests were executed on the Google Colab platform and can be reproduced. For the `beq` instruction, an issue arose due to incorrect hexadecimal conversion by the Colab compiler. Using the Venus simulator resolved this, indicating the problem is in the compiler, not the instruction.

### 3.1 `mul`

**Instructions:**
```assembly
nop
addi x1, x0, 6
addi x2, x0, 2
mul x3, x1, x2
end: nop
```

**Output:**
```
x0 = 0
x1 = 6
x2 = 2
x3 = c (12 in hexadecimal)
```

### 3.2 `div`

**Instructions:**
```assembly
nop
addi x1, x0, 6
addi x2, x0, 2
div x3, x1, x2
end: nop
```

**Output:**
```
x0 = 0
x1 = 6
x2 = 2
x3 = 3
```

### 3.3 `andi`

**Instructions:**
```assembly
nop
addi x1, x0, 1
addi x2, x0, 2
andi x3, x1, 2
andi x4, x2, 3
end: nop
```

**Output:**
```
x0 = 0
x1 = 1
x2 = 2
x3 = 0
x4 = 2
```

### 3.4 `beq`

**Instructions:**
```assembly
nop
addi x1, x0, 4
addi x2, x0, 4
beq x2, x1, end
sub x2, x2, x1
end: addi x3, x0, 7
```

**Output:**
```
x0 = 0
x1 = 4
x2 = 4
x3 = 7
```

## 4 Conclusion

The four requested instructions — `mul`, `div`, `andi`, and `beq` — were successfully implemented and tested. This required understanding Verilog and the architecture of the base RISC-V processor. Key changes were made in the control unit and ALU to support the new operations. We also debugged issues including compiler-related bugs (notably for `beq`) and logic errors during development. Ultimately, the objectives were achieved.
