---
title: RV64单周期CPU
date: 2024-03-06 08:04:28
tags:
- original
- homework
categories:
- ICS
---

### 整体架构

![架构图](https://ref.xht03.online/202412101917700.png)

我设计的RV64-CPU总共有14个模块：

- PC：更新下一条指令地址。
- PCincrement：将PC加4，作为PCNext的可能值。
- PCNext：根据Jump和CndCode信号，判断PC是跳转还是累加，计算出PCNext值。
- InstMem：指令内存。根据PC值取出指令并解析。
- RegFile：寄存器文件。执行读取和写回操作。
- ImmGen：立即数生成器。将立即数补位成64位。
- Control：控制模块。判断指令类型，并输出控制信号。
- ALU_A：选择ALU端口A的数据来源。
- ALU_B：选择ALU端口B的数据来源。
- ALU_control：选择ALU功能。
- ALU：执行算术逻辑运算。
- DataMem：数据内存。执行写入和读取内存操作。
- Conditional：计算Conditional Code。
- RegBack：选择写回寄存器的数据来源。

其中控制信号有以下：

- Branch：是否是Branch类指令
- Jump：是否是`jal`指令
- JumpReg：是否是`jalr`指令
- MemRead：是否读取内存
- MemWrite：是否写入内存
- ALUop：指令类型
- ALUSrc：ALU_B数据来源
- RegWrite：是否写回寄存器
- Halt：停止执行

我没有设置MemtoReg信号。这是因为：目前，MemtoReg信号和MemRead信号一致，读取内存一定是存进寄存器里。



### [RISC-V指令集 ](https://www.cs.sfu.ca/~ashriram/Courses/CS295/assets/notebooks/RISCV/RISCV_CARD.pdf)

我设计的CPU支持所有RV32I Base Integer Instruction（除`ecall`、`ebreak`外）。

![指令集](https://ref.xht03.online/202412101917808.png)



### 代码实现

#### PC

> 在时钟上升沿更新PC值。

```verilog
module PC(
    input logic clk,
    input logic [63:0] PCnext,
    output logic [63:0] PCaddress
);
    always_ff @(posedge clk) begin
        PCaddress <= PCnext;
    end
endmodule
```



#### PCincrement

> 计算PC累加值，作为`PCNext`的可能值。

```verilog
module PCincrement(
    input logic [63:0] PCaddress,
    output logic [63:0] PCincrement
);
    always_comb begin
        PCincrement = PCaddress + 4;
    end
endmodule

```



#### PCNext

> 根据`Conditional Code`、`Jump`、`JumpReg`信号，计算`PCNext`值。

```verilog
module PCNext(
    input logic [63:0] PCaddress,
    input logic [63:0] PCincrement,
    input logic [63:0] Imm,
    input logic [63:0] Rs1,         // jalr会用到rs1
    input logic Jump,               // jal
    input logic JumpReg,            // jalr
    input logic CndCode,            // branch
    input logic Halt,
    output logic [63:0] PCnext
);
    always_comb begin
        if(CndCode) PCnext = PCaddress + Imm;
        else if(Jump) PCnext = PCaddress + Imm;
        else if(JumpReg) PCnext = Rs1 + Imm;
        else if(Halt) PCnext = PCaddress;
        else PCnext = PCincrement;
    end
endmodule
```



#### Control

> 根据`OpCode`判断指令类型，并产生控制信号，指导：ALU运算、寄存器读写、内存读写、PC更新。

| ALUop |       指令类型        |
| :---: | :-------------------: |
|  000  |        R-type         |
|  001  |        I1-type        |
|  010  | I2-type（Load类指令） |
|  011  |        S-tpye         |
|  100  |        B-type         |
|  101  |        J-type         |
|  110  |        U-type         |

```verilog
module Control(
    input logic [6:0] OpCode,
    output logic Branch,
    output logic Jump,
    output logic JumpReg,
    output logic MemRead,
    output logic MemWrite,
    output logic [2:0] ALUop,       // 指令类型
    output logic ALUSrc,            // 0:Rs2 1:imm
    output logic RegWrite,
    output logic Halt               // 1:停机 0:正常运行
);
    /*
    *ALUop:
    *000:R-type 001:I1-type 010:I2-type 011:S-type
    *100:B-type 101:J-type 	110:U-type
    */
    
    always_comb begin
        case(OpCode)
            7'b0110011: begin       // R-type
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b000; ALUSrc = 0;
                RegWrite = 1;
                Halt = 0;
            end
            7'b0010011: begin       // I1-type
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b001; ALUSrc = 1;
                RegWrite = 1;
                Halt = 0;
            end
            7'b0000011: begin       // I2-type
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 1; MemWrite = 0;
                ALUop = 3'b010; ALUSrc = 1;
                RegWrite = 1;
                Halt = 0;
            end
            7'b0100011: begin       // S-type
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 1;
                ALUop = 3'b011; ALUSrc = 1;
                RegWrite = 0;
                Halt = 0;
            end
            7'b1100011: begin       // B-type
                Branch = 1; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b100; ALUSrc = 0;
                RegWrite = 0;
                Halt = 0;
            end
            7'b1101111: begin       // J-type (jal)
                Branch = 0; Jump = 1; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b101; ALUSrc = 0;
                RegWrite = 0;
                Halt = 0;
            end
            7'b1100111: begin       // jarl
                Branch = 0; Jump = 0; JumpReg = 1;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b101; ALUSrc = 1;
                RegWrite = 1;
                Halt = 0;
            end
            7'b0110111: begin       // U-type(lui)
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b110; ALUSrc = 1;
                RegWrite = 1;
                Halt = 0;
            end
            7'b0010111: begin       // U-type(auipc)
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b110; ALUSrc = 1;
                RegWrite = 1;
                Halt = 0;
            end
            default: begin
                Branch = 0; Jump = 0; JumpReg = 0;
                MemRead = 0; MemWrite = 0;
                ALUop = 3'b000; ALUSrc = 0;
                RegWrite = 0;
                Halt = 1;
            end
        endcase
    end
endmodule

```



#### InstMem

> 根据`PCaddress`读取指令，并拆解为`OpCode`、`Rs1`、`Rs2`、`Rd`。`Inst`是完整指令，用于生成立即数。

> 为了后续便于验证正确性，这里预先写入两条指令。

```verilog
module InstMem(
    input logic [63:0] PCaddress,
    output logic [6:0] OpCode,
    output logic [4:0] Rs1,
    output logic [4:0] Rs2,
    output logic [4:0] Rd,
    output logic [31:0] Inst
);
    logic [7:0] mem[0:1023];
    logic [31:0] instruction;
    
    initial begin
        mem[0] = 8'b10110111;        // b7  ( lui x5, 0x12345)
        mem[1] = 8'b00000010;       //  02
        mem[2] = 8'b01101000;       //  68
        mem[3] = 8'b00010010;       //  12
        mem[4] = 8'b10110011;       //  b3  ( add x7, x5, x6)
        mem[5] = 8'b00000011;       //  03
        mem[6] = 8'b01010011;       //  53
        mem[7] = 8'b00000000;       //  00
    end

    always_comb begin
        instruction = {mem[PCaddress+3], mem[PCaddress+2], mem[PCaddress+1], mem[PCaddress]};
        OpCode = instruction[6:0];
        Rs1 = instruction[19:15];
        Rs2 = instruction[24:20];
        Rd = instruction[11:7];
        Inst = instruction;
    end
endmodule
```



#### RegFile

> 对寄存器文件进行读写操作。

```verilog
module RegFile(
    input logic clk,
    input logic [4:0] ReadReg1,
    input logic [4:0] ReadReg2,
    input logic [4:0] WriteReg,
    input logic RegWrite,   // 是否写入寄存器
    input logic [63:0] WriteData,
    output logic [63:0] ReadData1,
    output logic [63:0] ReadData2
);
    logic [63:0] regs[0:31];     // 32个64-bit寄存器

    always_ff @(posedge clk) begin      // write是时序的
        if(RegWrite) begin
            regs[WriteReg] <= WriteData;
        end
    end

    always_comb begin		// read是组合的
        ReadData1 = regs[ReadReg1];
        ReadData2 = regs[ReadReg2];
    end
endmodule
```



#### ImmGen

> 根据`Inst`判断指令类型，根据不同指令，采取不同方式补位立即数。

> `funct7_30`是 funct7 的1个 bit ，指令中的第30个 bit 。funct7 中其余部分都是0，真正有区分作用的仅`funct7_30`一位。（区分`srai`和`srli`时）

```verilog
module ImmGen(
    input logic [31:0] Inst,
    output logic [63:0] Imm
);
    logic [6:0] opcode;
    logic [2:0] funct3;
    logic funct7_30;

    assign opcode = Inst[6:0];
    assign funct3 = Inst[14:12];
    assign funct7_30 = Inst[30];

    always_comb begin
        case(opcode)
            7'b0010011: begin                                   // I-type
                if(funct3 == 3'b101 && funct7_30 == 1) begin
                    Imm ={{(64 - 5){Inst[24]}},Inst[24:20]};
                end else begin
                    Imm = {{(64 - 12){Inst[31]}}, Inst[31:20]};
                end
            end
            7'b0000011: Imm = {{(64 - 12){Inst[31]}}, Inst[31:20]};    // I-type(load)
            7'b0100011: Imm = {{(64 - 12){Inst[31]}}, Inst[31:25], Inst[11:7]};    // S-type
            7'b1100011: Imm = {{(64 - 13){Inst[31]}}, Inst[31], Inst[7], Inst[30:25], Inst[11:8], 1'b0};    // B-type
            7'b1101111: Imm = {{(64 - 21){Inst[31]}}, Inst[31], Inst[19:12], Inst[20], Inst[30:21], 1'b0};    // J-type(jal)
            7'b1100111: Imm = {{(64 - 12){Inst[31]}}, Inst[31:20]};    // jalr
            7'b0110111: Imm = {{(64 - 32){Inst[31]}}, Inst[31:12], 12'b0};    // U-type(lui)
            7'b0010111: Imm = {{(64 - 32){Inst[31]}}, Inst[31:12], 12'b0};    // U-type(auipc)
            default: Imm = 64'b0;
        endcase
    end
endmodule
```



#### ALU_control

> 根据指令类型，选择ALU具体功能`ALUfunc`。

```verilog
module ALU_control(
    input logic [2:0] ALUop,
    input logic [2:0] funct3,
    input logic funct7_30,
    output logic [2:0] ALUfunc
);
    always_comb begin
        case(ALUop)
            3'b000: begin
                case(funct3)
                    3'b000: begin
                        case(funct7_30)
                            1'b0: ALUfunc = 3'b000;     // add
                            1'b1: ALUfunc = 3'b001;     // sub
                        endcase
                    end
                    3'b001: ALUfunc = 3'b101;           // sll(<<)
                    3'b010: ALUfunc = 3'b001;           // slt(做减法运算)
                    3'b011: ALUfunc = 3'b001;           // sltu(做减法运算)
                    3'b100: ALUfunc = 3'b100;           // xor
                    3'b101: begin
                        case(funct7_30)
                            1'b0: ALUfunc = 3'b111;     // srl(>>>)
                            1'b1: ALUfunc = 3'b110;     // sra(>>)
                        endcase
                    end
                    3'b110: ALUfunc = 3'b011;           // or
                    3'b111: ALUfunc = 3'b010;           // and
                endcase
            end
            3'b001: begin
                case(funct3)
                    3'b000: ALUfunc = 3'b000;           // add
                    3'b001: ALUfunc = 3'b101;           // sll(<<)
                    3'b010: ALUfunc = 3'b001;           // slt(做减法运算)
                    3'b011: ALUfunc = 3'b001;           // sltu(做减法运算)
                    3'b100: ALUfunc = 3'b100;           // xor
                    3'b101: begin
                        case(funct7_30)
                            1'b0: ALUfunc = 3'b111;     // srl(>>>)
                            1'b1: ALUfunc = 3'b110;     // sra(>>)
                        endcase
                    end
                    3'b110: ALUfunc = 3'b011;           // or
                    3'b111: ALUfunc = 3'b010;           // and
                endcase
            end
            3'b010: ALUfunc = 3'b000;                   // load类指令 add
            3'b011: ALUfunc = 3'b000;                   // store类指令 add
            3'b100: ALUfunc = 3'b001;                   // branch类指令 sub
            3'b101: ALUfunc = 3'b000;                   // jump类指令 add
            3'b110: ALUfunc = 3'b000;                   // U-type指令 add
            default: ALUfunc = 3'b000;
        endcase
    end
endmodule
```



#### ALU_A

> 选择`AluA`的数据来源。

```verilog
module ALU_A(
    input logic [3:0] ALUop,
    input logic [6:0] OpCode,
    input logic [63:0] ReadData1,
    input logic [63:0] PCaddress,
    output logic [63:0] AluA
);
    always_comb begin
        case(ALUop)
            3'b000: AluA = ReadData1;   // R-type
            3'b001: AluA = ReadData1;   // I1-type
            3'b010: AluA = ReadData1;   // I2-type
            3'b011: AluA = ReadData1;   // S-type
            3'b100: AluA = ReadData1;   // B-type
            3'b101: AluA = PCaddress;   // J-type
            3'b110: begin
                if(OpCode == 7'b0110111) begin
                    AluA = 64'b0;                       // U-type(lui)
                end else if(OpCode == 7'b0010111)begin
                    AluA = PCaddress;                   // U-type(auipc)
                end
            end
            default: AluA = 64'b0;
        endcase
    end
endmodule
```



#### ALU_B

> 选择`AluB`的数据来源。

```verilog
module ALU_B(
    input logic [6:0] OpCode,
    input logic ALUSrc,
    input logic [63:0] ReadData2,
    input logic [63:0] Imm,
    output logic [63:0] AluB
);
    always_comb begin
        if(ALUSrc) begin
            if(OpCode == 7'b1101111 || OpCode == 7'b1100111) begin      // J-type(jal, jalr)
                AluB = 4;
            end else begin
                AluB = Imm;
            end
        end else begin
            AluB = ReadData2;
        end
    end
endmodule

```



#### ALU

> 执行算术逻辑运算。

```verilog
module ALU(
    input logic [63:0] A,
    input logic [63:0] B,
    input logic [2:0] ALUfunc,
    output logic [63:0] result,
    output logic Zero,
    output logic SignedLess,
    output logic UnsignedLess
);
    assign Zero = (result == 64'b0);
    assign SignedLess = ($signed(A) < $signed(B));
    assign UnsignedLess = (A < B);

    always_comb begin
        case(ALUfunc)
            3'b000: result = A + B;
            3'b001: result = A - B;
            3'b010: result = A & B;
            3'b011: result = A | B;
            3'b100: result = A ^ B;
            3'b101: result = A << B;
            3'b110: result = A >> B;
            3'b111: result = A >>> B;
        endcase
    end
endmodule
```



#### DataMem

> 执行对数据内存的读写操作。

```verilog
module DataMem(
    input logic [63:0] Address,
    input logic [63:0] WriteData,
    input logic MemWrite,
    input logic MemRead,
    input logic [2:0] funct3,
    output logic [63:0] ReadData
);
    logic [7:0] mem[0:1023];

    always_comb begin
        if(MemWrite) begin
            case(funct3)
                3'h0: mem[Address] = WriteData[7:0];
                3'h1: {mem[Address], mem[Address+1]} = WriteData[15:0];
                3'h2: {mem[Address], mem[Address+1], mem[Address+2], mem[Address+3]} = WriteData[31:0];
            endcase
        end
        if(MemRead) begin
            case(funct3)
                3'h0: ReadData = {{(64 - 8){mem[Address][7]}}, mem[Address]};
                3'h1: ReadData = {{(64 - 16){mem[Address][7]}}, mem[Address], mem[Address+1]};
                3'h2: ReadData = {{(64 - 32){mem[Address][7]}}, mem[Address], mem[Address+1], mem[Address+2], mem[Address+3]};
                3'h4: ReadData = {{(64 - 8){1'b0}}, mem[Address]};
                3'h5: ReadData = {{(64 - 16){1'b0}}, mem[Address], mem[Address+1]};
            endcase
        end
    end
endmodule
```



#### Conditional

> 根据ALU计算得到的符号位`Zere`、`SignedLess`、`UnsignedLess`，判断是否跳转，得到`CndCode`。

```verilog
module Conditional(
    input logic Branch,
    input logic Zero,
    input logic SignedLess,
    input logic UnsignedLess,
    input logic [2:0] funct3,
    output logic CndCode        // 是否满足条件跳转
);
    always_comb begin
        if(Branch) begin
            case(funct3)
                3'h0: if(Zero) CndCode = 1'b1;  // beq
                3'h1: if(!Zero) CndCode = 1'b1;   // bne
                3'h4: if(SignedLess) CndCode = 1'b1;   // blt
                3'h5: if(!SignedLess) CndCode = 1'b0;   // bge
                3'h6: if(UnsignedLess) CndCode = 1'b1;   // bltu
                3'h7: if(!UnsignedLess) CndCode = 1'b1;   // bgeu
                default: CndCode = 1'b0;
            endcase
        end else begin
            CndCode = 1'b0;
        end
    end
endmodule
```



#### RegBack

> 选择写回寄存器的数据来源。

```verilog
module RegBack(
    input logic MemRead,
    input logic [63:0] ReadData,    // 从内存中读出的数据
    input logic [63:0] ALUresult,
    output logic [63:0] WriteData
);
    always_comb begin
        if(MemRead)begin
            WriteData = ReadData;
        end else begin
            WriteData = ALUresult;
        end
    end
endmodule
```



#### arch

> 在顶层模块，完成连线。

```verilog
module arch(
    input logic CLK
);
    logic [63:0] PCaddress;
    logic [63:0] PCnext;
    logic [63:0] PCincrement;
    
    initial begin
        PCaddress = 64'b0;
    end

    PC pc(
        .clk(CLK),
        .PCnext(PCnext),
        .PCaddress(PCaddress)
    );

    PCincrement pcincrement(
        .PCaddress(PCaddress),
        .PCincrement(PCincrement)
    );

    logic [6:0] OpCode;
    logic [4:0] Rs1;
    logic [4:0] Rs2;
    logic [4:0] Rd;
    logic [31:0] Inst;
    logic [63:0] Imm;

    InstMem instmem(
        .PCaddress(PCaddress),
        .OpCode(OpCode),
        .Rs1(Rs1),
        .Rs2(Rs2),
        .Rd(Rd),
        .Inst(Inst)
    );

    logic RegWrite;
    logic [63:0] WriteData;
    logic [63:0] ReadData1;
    logic [63:0] ReadData2;

    RegFile regfile(
        .clk(CLK),
        .ReadReg1(Rs1),
        .ReadReg2(Rs2),
        .WriteReg(Rd),
        .RegWrite(RegWrite),
        .WriteData(WriteData),
        .ReadData1(ReadData1),
        .ReadData2(ReadData2)
    );

    ImmGen immgen(
        .Inst(Inst),
        .Imm(Imm)
    );

    logic Branch;
    logic Jump;
    logic JumpReg;
    logic MemRead;
    logic MemWrite;
    logic [2:0] ALUop;
    logic ALUSrc;
    logic Halt;

    Control control(
        .OpCode(OpCode),
        .Branch(Branch),
        .Jump(Jump),
        .JumpReg(JumpReg),
        .MemRead(MemRead),
        .MemWrite(MemWrite),
        .ALUop(ALUop),
        .ALUSrc(ALUSrc),
        .RegWrite(RegWrite),
        .Halt(Halt)
    );

    logic [2:0] ALUfunc;

    ALU_control alu_control(
        .ALUop(ALUop),
        .funct3(Inst[14:12]),
        .funct7_30(Inst[30]),
        .ALUfunc(ALUfunc)
    );

    logic [63:0] AluA;
    logic [63:0] AluB;
    logic [63:0] ALUresult;
    logic Zero;
    logic SignedLess;
    logic UnsignedLess;

    ALU_A alu_a(
        .ALUop(ALUop),
        .OpCode(OpCode),
        .ReadData1(ReadData1),
        .PCaddress(PCaddress),
        .AluA(AluA)
    );

    ALU_B alu_b(
        .OpCode(OpCode),
        .ALUSrc(ALUSrc),
        .ReadData2(ReadData2),
        .Imm(Imm),
        .AluB(AluB)
    );

    ALU alu(
        .A(AluA),
        .B(AluB),
        .ALUfunc(ALUfunc),
        .result(ALUresult),
        .Zero(Zero),
        .SignedLess(SignedLess),
        .UnsignedLess(UnsignedLess)
    );

    logic [63:0] ReadData;      // 从内存中读出的数据

    DataMem datamem(
        .Address(ALUresult),
        .WriteData(ReadData2),
        .MemWrite(MemWrite),
        .MemRead(MemRead),
        .funct3(Inst[14:12]),
        .ReadData(ReadData)
    );

    RegBack regback(
        .MemRead(MemRead),
        .ReadData(ReadData),
        .ALUresult(ALUresult),
        .WriteData(WriteData)
    );

    logic CndCode;

    Conditional conditional(
        .Branch(Branch),
        .Zero(Zero),
        .SignedLess(SignedLess),
        .UnsignedLess(UnsignedLess),
        .funct3(Inst[14:12]),
        .CndCode(CndCode)
    );

    PCNext pcnext(
        .PCaddress(PCaddress),
        .PCincrement(PCincrement),
        .Imm(Imm),
        .Rs1(ReadData1),
        .Jump(Jump),
        .JumpReg(JumpReg),
        .CndCode(CndCode),
        .Halt(Halt),
        .PCnext(PCnext)
    );

endmodule
```



### 正确性验证

为检验正确性，我向 InstMem 中写入了两条指令：

- `lui x5, 0x12345	`：0x126802b7

- `add x7, x5, x6`：0x005303b3	

然后我们在 Vivado 中进行 synthesis 和 stimulation 。

```verilog
module sim();
    logic clk;
    arch test(.CLK(clk));

    initial begin
        repeat (20) begin
            clk = 0;
            #5;
            clk = 1;
            #5;
        end
        $finish;
    end
    
    initial begin
        $dumpfile("test.vcd");
        $dumpvars;
    end
endmodule
```

我们可以看到：指令都被正确解析。

![波形图1](https://ref.xht03.online/202412101917711.png)

我们查看Regfile中的5号、7号寄存器。它们都被正确地写入和读取。

![波形图2](https://ref.xht03.online/202412101918999.png)



### 注记



