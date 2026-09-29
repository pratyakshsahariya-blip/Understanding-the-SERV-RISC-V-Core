# Understanding-the-SERV-RISC-V-Core
  In this repository we will be understanding the architecture and internal working of the SERV (Serial RISC-V) core.
  
  The goal is not just to use the core, but to go through its RTL implementation and understand **how a RISC-V processor can be built from the ground up**, especially with SERV's unique serial architecture.

## About SERV :

  **SERV** stands for **SERial RISC-V**. It is a small, resource-efficient RISC-V processor core designed by **Olof Kindgren**.
  
  Unlike a conventional RISC-V CPU that processes a complete word in parallel, SERV processes data **one bit at a time**. This makes the design extremely small and simple, at the cost of performance.
  
  SERV is therefore a great project for learning:
  
    - RISC-V processor architecture
    - CPU datapaths and control logic
    - Instruction execution
    - Register files
    - ALUs
    - Program counters
    - Memory interfaces
    - RTL design
    - Verilog/SystemVerilog
    - FPGA implementation
    - Bit-serial processor architectures

## What are we understanding :

  Going through the SERV source code and trying to understand the complete path of an instruction through the processor.
  
  Some of the questions I am exploring are:
  
    - How is a RISC-V instruction fetched?
    - How is the instruction decoded?
    - How are registers read and written?
    - How does the ALU perform operations one bit at a time?
    - How does the program counter work?
    - How are branches and jumps handled?
    - How does the core communicate with memory?
    - How are loads and stores implemented?
    - How does the control logic coordinate all these operations?
    - Why was a serial architecture chosen?
    - What are the trade-offs between area, speed, and complexity?

## Architecture :

  At a high level, we are studying SERV as a collection of interacting blocks:

  '''
                   +----------------------+
                   |      RISC-V          |
                   |     Instruction      |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   |   Instruction        |
                   |      Decode          |
                   +----------+-----------+
                              |
                              v
          +-------------------+-------------------+
          |                                       |
          v                                       v
  +---------------+                       +---------------+
  | Register File |                       |  Control /    |
  |               |                       |   Sequencing  |
  +-------+-------+                       +-------+-------+
          |                                       |
          +-------------------+-------------------+
                              |
                              v
                   +----------------------+
                   |   Serial Datapath    |
                   |                      |
                   |  ALU / Shifters /    |
                   |  Address Generation  |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   |    Memory Interface  |
                   +----------------------+

   '''
  
  The actual SERV implementation is more specialized than this simplified diagram, and one of the main goals of this repository is to understand how the different RTL modules fit together.

## Why a Serial CPU?
  A normal CPU datapath might operate on 32 bits in parallel.
  
  SERV takes a different approach.
  
  Instead of performing an operation on all 32 bits simultaneously, the datapath processes the bits serially.

   '''
   
     For example, conceptually:
     
     32-bit operation
     
     Parallel CPU:
       +---+---+---+---+---+---+---+---+
       |31 |30 |29 |...| 3 | 2 | 1 | 0 |
       +---+---+---+---+---+---+---+---+
                    |
                    v
              Process in parallel
     
     
     SERV:
       bit 0 -> bit 1 -> bit 2 -> ... -> bit 31
                     |
                     v
              Process serially
   '''
    
  This significantly reduces the amount of hardware required.
  
  The trade-off is that an operation takes multiple clock cycles.
  
  Understanding this trade-off is one of the key things from studying SERV.

## Learning Approach :
  
  The general approach is:
  
    Understand the RISC-V ISA concepts required by the core.
    
    Identify the major RTL modules.
    
    Understand the signals between those modules.
    
    Follow the execution of an instruction cycle by cycle.
    
    Understand how the serial datapath processes individual bits.
    
    Trace arithmetic, logical, load/store, and branch instructions.
    
    Understand how the program counter changes.
    
    Understand how memory accesses are generated.
    
    Relate the RTL implementation back to the RISC-V specification.
    
    Document my findings as I go.
    
 Things I Plan to Document :
   1. Instruction Fetch
      Understanding how SERV obtains an instruction from memory and how the instruction reaches the execution logic.
   
   2. Instruction Decode
      Understanding how the fields of a RISC-V instruction are interpreted:
   
      +---------+---------+---------+---------+---------+---------+
      | funct7  |  rs2    |  rs1    | funct3  |   rd    | opcode |
      +---------+---------+---------+---------+---------+---------+
      
      I will connect these instruction fields to the corresponding control signals in the RTL.
   
   3. Register File
      Understanding:
   
        How registers are stored
        
        How registers are read
        
        How registers are written
        
        How the zero register is handled
        
        How serial processing interacts with the register file
   
   4. Serial ALU
      One of the main areas I want to understand is how common operations are implemented using a bit-serial datapath.
   
      For example:
   
          Operand A
              |
              v
          +-------+
          |       |
          |  ALU  | ---> Result bit
          |       |
          +-------+
              ^
              |
          Operand B
    
      Rather than calculating the complete result in one cycle, the datapath works through the bits over multiple cycles.
   
   5. Program Counter
      Investigating how the PC is updated for:
   
        Sequential execution
        
        Conditional branches
        
        Jumps
        
        Jump-and-link instructions
        
        Exceptions or other control-flow changes where applicable
     
   6. Load and Store Instructions
      Understanding how memory addresses are generated and how data moves between the processor and memory.
      
      Understand how multi-byte values are handled despite the serial nature of the core.
   
   7. Control and Sequencing
      A major part of understanding SERV is understanding when each operation happens.
      
      Trace the internal control signals and state transitions to understand how a single RISC-V instruction is broken into multiple steps.
      
      Example Instruction Tracing
      I plan to document instructions in a format similar to:
      
      Instruction:
          ADD x3, x1, x2
      
      Goal:
          x3 = x1 + x2
      
      Questions to trace:
      
          1. How is the instruction fetched?
          2. How is ADD identified?
          3. How are x1 and x2 selected?
          4. How does the register file provide the operands?
          5. How does the serial ALU perform the addition?
          6. How many cycles are required?
          7. How is the result written back to x3?
          8. What happens to the program counter?
      
      The idea is to follow an instruction from fetch → decode → execute → memory (if required) → write-back.
   
   RISC-V ISA

     SERV implements a small RISC-V ISA subset, making it particularly interesting for understanding the relationship between an ISA and its hardware implementation.
     
     As part of this project, we should also study the relevant RISC-V instructions and map them to the RTL implementation.
     
     The focus will be on understanding the hardware rather than simply memorizing the instruction set.
     
   Why SERV?
   
     SERV is particularly interesting to me because it demonstrates that a CPU does not necessarily need a large, complicated datapath.
     
     A processor can be constructed using a very small amount of hardware by trading parallelism and performance for simplicity and area efficiency.
     
     That makes SERV a useful design to study for understanding the fundamentals of CPU architecture.
   
   Repository Status
   🚧 Work in Progress
   
   This repository is primarily a learning and documentation project.
   
   My understanding may evolve as I dig deeper into the RTL, and some explanations may be updated or corrected as I verify them against the implementation.
