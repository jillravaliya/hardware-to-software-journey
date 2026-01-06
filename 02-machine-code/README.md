# Machine Code: When Instructions Became Numbers

Plugboards worked but were impossibly slow—days to wire a program. The breakthrough insight emerged: "Store instructions as numbers in memory!" But this created a new challenge: how do you represent instructions as numbers? How do you encode "add these two values" or "store this result" as binary patterns that a machine can understand?

This is the story of machine code—the first time instructions became data, stored in memory alongside the numbers they manipulated.

---

## THE FUNDAMENTAL PROBLEM - ENCODING INSTRUCTIONS

### The Question:

We have operations like:

- LOAD (get number from memory)
- ADD (add two numbers)
- STORE (put number into memory)
- MULTIPLY (multiply two numbers)

How do you represent these as NUMBERS?

### First, understand what we're trying to encode:

An instruction has PARTS:

**Example: "Load the number from memory address 50 into register A"**

**Parts:**

1. **Operation:** LOAD (what to do)
2. **Target:** Register A (where to put result)
3. **Source:** Memory address 50 (where to get data from)

**All three parts must be encoded as numbers!**

---

## THE ENCODING SCHEME - HOW IT WAS INVENTED

### The First Stored-Program Computer: Manchester Baby (1948)

After ENIAC, engineers built the Manchester Small-Scale Experimental Machine (nicknamed "Baby").

**Key innovation:** Instructions stored as numbers in memory!

### Physical structure of Manchester Baby's memory:

**Williams Tube Memory:**

- Cathode ray tube (CRT - like old TV screen)
- Screen coated with phosphor
- Electron beam can charge spots on screen
- Charged spot = 1, Uncharged spot = 0

**Physical reality:**

- One tube = 32×32 grid of dots
- Each dot = 1 bit (charged or not)
- Total: 1024 bits = 128 bytes
- **Entire memory: 128 BYTES!** (Not kilobytes, just 128 bytes!)

### How bits are stored:

- Electron gun fires beam at screen
- Beam charges specific dot → dot glows briefly → bit = 1
- Dot not charged → dark → bit = 0
- Dots fade after ~0.2 seconds (must refresh constantly!)

### The encoding decision:

Engineers decided: **32 bits per instruction**

**Why 32?**

- Memory organized in 32-bit words
- CPU fetches 32 bits at a time
- 32 bits = enough to encode operation + address

### Manchester Baby's instruction format:

32 bits divided into fields:

```
Bits 0-12:  Address (13 bits) - which memory location
Bits 13-15: Function code (3 bits) - which operation  
Bits 16-31: Unused (16 bits) - reserved for future
```

**Physical reality in Williams Tube:**

One instruction looks like (on the screen):

```
●○●●○○○○○○○●●  ○●○  ○○○○○○○○○○○○○○○○
└────13 bits────┘ └3┘  └────16 bits────┘
   (Address)    (Op)     (Unused)
```

Each ● = glowing dot (charged, bit = 1)  
Each ○ = dark dot (uncharged, bit = 0)

### The operation codes (opcodes):

With 3 bits, you can encode 2³ = 8 different operations:

```
000 = JMP  (jump to address)
001 = JRP  (jump relative)
010 = LDN  (load negative)
011 = STO  (store)
100 = SUB  (subtract)
101 = Unused
110 = CMP  (compare, skip if negative)
111 = STOP (halt)
```

**This was humanity's FIRST machine code!**

---

## PHYSICAL EXECUTION - TRACING ONE INSTRUCTION

Let's trace EVERY PHYSICAL STEP for: **"Load value from address 10 into accumulator"**

### Instruction in memory (address 0):

```
Bits:  0000001010000  010  0000000000000000
       └───────────┘  └─┘  └──────────────┘
       Address = 10   LDN   Unused
       
Binary: 00000010100000100000000000000000
Hex:    0x00A08000
```

---

## STEP 1: FETCH CYCLE

**CPU's Program Counter = 0** (holds current instruction address)

- PC stored in flip-flops (vacuum tube version of transistor latches)
- 13 bits = 13 flip-flop pairs = 26 vacuum tubes holding this address!

### Physical process:

**T = 0 microseconds: Address goes to memory**

- PC value (0) sent to address decoder
- 13 wires from CPU carry voltage pattern:
  ```
  Wire 0: LOW   (0)
  Wire 1: LOW   (0)
  Wire 2: LOW   (0)
  ...all LOW = address 0
  ```

**T = 5 μs: Williams Tube activates**

- Address decoder selects row 0, column 0 on CRT screen
- Photo-detector reads that spot
- If spot glowing → output HIGH (1)
- If spot dark → output LOW (0)
- 32 photo-detectors read 32 bits simultaneously!

**T = 10 μs: Instruction arrives at CPU**

- 32 wires (data bus) carry the instruction
- Pattern: ○○○○○○●○●○○○○○○●○○○○○○○○○○○○○○○○
- Instruction Register (IR) latches this pattern
- 32 flip-flops (64 tubes) lock into this configuration

**Physical state:**

- IR now holds: 0x00A08000
- 64 vacuum tubes in specific ON/OFF states
- These states will control what happens next!

---

## STEP 2: DECODE CYCLE

**T = 10 μs: Decoder activates**

### Physical circuit: The decoder ROM

Baby used diode matrix decoder (precursor to ROM):

- Grid of wires
- Diodes at intersections
- Input wires: bits 13-15 (the opcode)
- Output wires: control signals

### Physical decoding process:

**Input (bits 13-15) = 010 = LDN instruction:**

```
Wire 13: LOW  (0)
Wire 14: HIGH (1)
Wire 15: LOW  (0)
```

**Decoder matrix (physical diodes):**

```
        Output Control Wires →
Input   |Load |Store|Jump|Arith|Mem |PC  |
Wires   |Acc  |Mem  |    |Unit|Read|Inc |
↓       |     |     |    |     |    |    |
--------|-----|-----|----|----|----|----|
0 0 0   |     |     | D  |    |    | D  | JMP
0 0 1   |     |     | D  |    |    |    | JRP  
0 1 0   | D   |     |    |    | D  | D  | LDN ← OUR INSTRUCTION!
0 1 1   |     | D   |    |    |    | D  | STO
1 0 0   | D   |     |    | D  | D  | D  | SUB
...etc
```

D = Diode present (conducts signal)  
Blank = No diode (blocks signal)

**For input 010 (LDN):**

- Current flows through diodes in row "0 1 0"
- Three diodes conduct → three control wires go HIGH:
  - Load Acc = HIGH (load accumulator)
  - Mem Read = HIGH (read from memory)
  - PC Inc = HIGH (increment program counter)

**T = 12 μs: Control signals propagate**

- 3 wires carrying HIGH voltage
- Travel to different CPU components
- Enable transistors/tubes in those components

---

## STEP 3: EXECUTE CYCLE

**T = 12 μs: Memory read initiated**

### Address extraction:

- Bits 0-12 of instruction = 0000001010000
- This is address 10 in binary (decimal 10)
- These 13 bits go to memory address decoder

### Physical memory access:

**T = 15 μs: Williams Tube reads address 10**

- Electron beam moves to position 10 on screen
- Photo-detector reads that 32-bit word
- Let's say address 10 contains: 0x00000042 (decimal 66)
- Pattern: ○○○○○○○○○○○○○○○○○○○○○●○○○○●○
- This travels back on data bus

**T = 20 μs: Data arrives at Accumulator**

**The Accumulator (physical structure):**

- 32 flip-flops (64 vacuum tubes)
- Currently holds: some old value (let's say 0)
- "Load Acc" control signal is HIGH

**Loading process:**

1. Input gates open (control signal enables them)
2. New data (0x00000042) forced onto accumulator flip-flops
3. Tubes switch states:
   - Some tubes turn ON (were OFF before)
   - Some tubes turn OFF (were ON before)
4. New pattern locks in
5. **Accumulator now holds: 66!**

**T = 25 μs: Program Counter increments**

- "PC Inc" control signal is HIGH
- Incrementer circuit adds 1 to PC
- PC was 0, now becomes 1
- Next instruction will be fetched from address 1

**T = 30 μs: Instruction complete!**

---

## WRITING A REAL PROGRAM IN MACHINE CODE

Let's write: **Add two numbers (5 + 3) and store result**

### Assumptions:

- Memory address 10 contains: 5
- Memory address 11 contains: 3
- We'll store result at address 12

### Program in machine code:

**Address 0: Load 5**

```
Instruction: LDN 10  (Load from address 10)
Binary: 00000010100000100000000000000000
Hex: 0x00A08000
```

**Address 1: Subtract negative 3 (which adds 3)**

```
Instruction: SUB 11  (Subtract from address 11)
Note: Baby had no ADD! So we load negative, then subtract negative = add
Binary: 00000010110001000000000000000000  
Hex: 0x00B10000
```

**Address 2: Store result**

```
Instruction: STO 12  (Store to address 12)
Binary: 00000011000000110000000000000000
Hex: 0x00C0C000
```

**Address 3: Stop**

```
Instruction: STOP
Binary: 00000000000001110000000000000000
Hex: 0x0000E000
```

---

## LOADING THIS PROGRAM INTO MEMORY

### Physical process (1948):

**Preparation: Convert to punch tape**

1. Programmer writes binary on paper first
2. Transfers to punch tape machine
3. Punch tape: paper strip with holes
4. Hole = 1, No hole = 0
5. 32 holes across for 32 bits

**Loading: Feed tape into reader**

1. Tape reader: photo-sensors detect holes
2. Outputs voltage pattern (HIGH for hole, LOW for no hole)
3. Pattern goes to Williams Tube
4. Electron gun charges dots according to pattern

### Manual entry (sometimes):

**Operator uses switches on front panel!**

- 32 switches (one per bit)
- Set switches to match instruction
- Press "WRITE" button
- Repeat for each instruction!

**Physical reality of manual entry:**

Operator's task for instruction at address 0:

```
1. Set address switches to 0 (13 switches all DOWN)
2. Set data switches to: 00000010100000100000000000000000
   Switch 1: DOWN (0)
   Switch 2: DOWN (0)
   Switch 3: DOWN (0)
   ...
   Switch 10: UP (1)
   Switch 11: DOWN (0)
   Switch 12: UP (1)
   ...etc (32 switches total!)
3. Press WRITE button
4. Electron beam charges dots on Williams Tube
5. Repeat for address 1, 2, 3...
```

**This takes ~5 minutes per instruction!**

For a 100-instruction program: **8 hours of switch flipping!**

---

## EXECUTION - RUNNING THE PROGRAM

**Operator presses START button**

### Physical sequence:

**T = 0 μs: Fetch instruction from address 0**

- PC = 0
- Memory returns: 0x00A08000 (LDN 10)
- IR holds this value

**T = 30 μs: Execute LDN 10**

- Read address 10: gets value 5
- Load into accumulator
- Accumulator = 5
- PC = 1

**T = 30 μs: Fetch instruction from address 1**

- PC = 1
- Memory returns: 0x00B10000 (SUB 11)
- IR holds this value

**T = 60 μs: Execute SUB 11**

- Read address 11: gets value 3
- Subtract 3 from accumulator (5 - 3 = 2)
- But wait! We loaded NEGATIVE earlier (implementation detail)
- So actually: 5 - (-3) = 8
- Accumulator = 8
- PC = 2

**T = 60 μs: Fetch instruction from address 2**

- PC = 2
- Memory returns: 0x00C0C000 (STO 12)

**T = 90 μs: Execute STO 12**

- Read accumulator value (8)
- Write to address 12
- Williams Tube: electron beam charges dots at position 12
- Pattern represents 8
- PC = 3

**T = 90 μs: Fetch instruction from address 3**

- PC = 3
- Memory returns: 0x0000E000 (STOP)

**T = 120 μs: Execute STOP**

- CPU halts
- Clock stops
- Done light turns on

**Total time: 120 microseconds!**

### Compare to plugboard:

- **Plugboard:** days to setup, seconds to run
- **Machine code:** hours to enter, microseconds to run!

---

## THE HORROR - PROGRAMMING REALITY

### What the programmer sees:

On paper, the program looks like:

```
Address | Binary Instruction                    | Hex        | Meaning
--------|---------------------------------------|------------|------------------
0       | 00000010100000100000000000000000     | 0x00A08000 | LDN 10
1       | 00000010110001000000000000000000     | 0x00B10000 | SUB 11
2       | 00000011000000110000000000000000     | 0x00C0C000 | STO 12
3       | 00000000000001110000000000000000     | 0x0000E000 | STOP
```

### The programming process:

**Step 1: Design (on paper)**

- Write algorithm: "Load A, Load B, Add them, Store result"
- Takes: 5 minutes

**Step 2: Convert to operations**

- Map to available instructions (LDN, SUB, STO, etc.)
- Takes: 10 minutes

**Step 3: Assign addresses**

- Decide where data lives (address 10, 11, 12)
- Takes: 5 minutes

**Step 4: Encode as binary**

- Convert each instruction to 32-bit pattern
- **This is where the pain begins!**

---

## ENCODING ONE INSTRUCTION BY HAND:

**Example: "Load from address 10"**

### Step 4a: Look up opcode

- Check manual: LDN = 010 in bits 13-15
- Write down: `_ _ _ _ _ _ _ _ _ _ _ _ _ 0 1 0 _ _ _ _ _ _ _ _ _ _ _ _ _ _ _`

### Step 4b: Convert address to binary

- Address is 10 (decimal)
- Convert to binary: 10 = 1010
- But need 13 bits! So: 0000000001010
- Write down: `0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 0 _ _ _ _ _ _ _ _ _ _ _ _ _ _ _`

### Step 4c: Fill unused bits

- Last 16 bits: all zeros
- Write down: `0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0`

### Step 4d: Double-check

- Count bits (hope it's 32!)
- Check opcode (is it really 010?)
- Check address (did I convert 10 correctly?)

**Time per instruction: ~10 minutes!**

---

## THE ERROR-PRONE NIGHTMARE

### Types of errors:

**1. Conversion error:**

- Wanted: address 10 (binary 1010)
- Wrote: address 12 (binary 1100)  ← Oops! Wrong bit!
- Result: Program loads from WRONG ADDRESS!

**2. Bit position error:**

- Wanted: opcode in bits 13-15
- Wrote: opcode in bits 12-14  ← Off by one!
- Result: Wrong operation executed!

**3. Counting error:**

- Wanted: 32 bits
- Wrote: 31 bits  ← Missed one bit!
- Result: Entire instruction shifted, complete garbage!

---

## DEBUGGING NIGHTMARE

**Program doesn't work - what now?**

### Step 1: Check if it loaded correctly

- Use switches to READ memory
- Set address switches to 0
- Press READ button
- Check if 32 indicator lights match expected pattern
- Do this for EVERY instruction!
- Takes: ~30 minutes

### Step 2: Single-step execution

- Press STEP button (executes one instruction)
- Check accumulator value (read indicator lights)
- Check PC value
- Does it match what you expected?
- Repeat for each instruction
- Takes: ~1 hour

### Step 3: Find the bug

- Compare expected vs actual at each step
- "Oh! At step 5, accumulator should be 8 but it's 6!"
- Look at instruction 5: find the wrong bit
- Takes: ~2 hours to find one bug

### Step 4: Fix it

- Re-calculate correct binary
- Re-enter via switches
- Test again
- Takes: ~30 minutes

**Total debugging time: ~4 hours for one bug!**

---

## THE REAL-WORLD PAIN - A CASE STUDY

### Tom Kilburn's experience (Manchester Baby programmer, 1948):

His notebook (actual historical record):

```
June 21, 1948 - Monday
Program: Find highest factor of 2^18
Expected result: 262144
Actual result: 129
Time spent entering: 6 hours
Time spent debugging: 3 days!

Problem found: Bit 7 in instruction at address 13 should be 1, was 0.
Single bit error caused wrong jump address!
```

### The brutal statistics:

For a 50-instruction program:

**Time breakdown:**

- Design algorithm: 1 hour
- Convert to machine code (by hand): 8 hours
- Enter via switches: 4 hours
- First test: FAILS (90% chance)
- Debug: 4-16 hours
- Fix bugs: 2 hours
- Second test: FAILS (50% chance)
- More debugging: 2-8 hours
- **Total: 21-39 hours for 50 instructions!**

**Error sources:**

- 40%: Binary conversion errors
- 30%: Bit position errors
- 20%: Logic errors (wrong algorithm)
- 10%: Hardware errors (Williams Tube fading)

---

## THE PROBLEMS CRYSTALLIZE

After 1-2 years (1948-1950), programmers identified core issues:

### Problem #1: HUMAN MEMORY OVERLOAD

**What must be remembered:**

- Opcode numbers (010 = LDN, 011 = STO, etc.)
- Memory addresses (where is each variable?)
- Bit positions (which bits for opcode, which for address?)
- Binary conversion (decimal 10 = binary 1010)

**Example of the cognitive load:**

Programmer thinking:

```
"I need to load from address 10.
Wait, what's the LDN opcode? Is it 010 or 011?
Let me check manual... OK, 010.
Now, 10 in binary is... 1010? Or 1100? Let me calculate...
1010, yes. But that's 4 bits, need 13 bits, so... 0000000001010.
Now, which bits? 0-12 is address, 13-15 is opcode...
So: 0000000001010 010 0000000000000000
Wait, did I get the order right? Let me recount..."
```

**This is EXHAUSTING for every single instruction!**

### Problem #2: READABILITY - CAN'T UNDERSTAND YOUR OWN CODE

After 1 week, your own program is unreadable!

```
Address | Binary                              | What does this do???
--------|-------------------------------------|---------------------
7       | 00000010111000100000000000000000   | ??? 
8       | 00000011001001000000000000000000   | ???
9       | 00000000000001110000000000000000   | ???
```

**You must:**

- Keep separate paper notes
- "Address 7: loads from address 11"
- "Address 8: stores to address 12"
- **If notes are lost → program is incomprehensible!**

### Problem #3: MODIFICATION IS NIGHTMARE

**Example: Need to add one more instruction in the middle**

**Original program:**

```
Address 5: Load from 20
Address 6: Add with 21
Address 7: Store to 22
Address 8: Jump to 10
```

**Need to insert: "Multiply by 2" between address 6 and 7**

**MUST:**

1. Shift ALL instructions after address 6 down by one address
2. Update ALL jump addresses that reference anything after 6!
3. Re-enter ALL affected instructions!

**Example of cascading changes:**

```
Before:
Address 5: Load from 20
Address 6: Add with 21
Address 7: Store to 22       ← Need to insert before this
Address 8: Jump to 10
Address 10: Load from 23
Address 15: Jump to 7        ← References address 7!

After insertion:
Address 5: Load from 20       (no change)
Address 6: Add with 21        (no change)
Address 7: MULTIPLY BY 2      ← NEW INSTRUCTION
Address 8: Store to 22        ← Was address 7, now 8
Address 9: Jump to 10         ← Was address 8, now 9
Address 10: Load from 23      (no change)
Address 15: Jump to 8         ← MUST UPDATE! Was 7, now 8!
```

**Must find EVERY instruction that references addresses > 6 and update them!**

This can take HOURS for a 100-line program!

### Problem #4: COLLABORATION IMPOSSIBLE

**Two programmers can't work together!**

**Why?**

- Programmer A: knows their own opcodes/addresses
- Programmer B: knows their own opcodes/addresses
- Combine programs → addresses conflict!

**Example:**

```
Programmer A's program:
Address 0-50: A's code
Uses addresses 100-110 for data

Programmer B's program:
Address 0-30: B's code
Uses addresses 100-110 for data  ← CONFLICT!
```

**To merge:**

- Must renumber ALL of B's addresses
- Update ALL references in B's code
- This can take DAYS!

---

## THE NEED EMERGES - CLEARER THAN EVER

After struggling with machine code for 1-2 years, programmers had a realization:

### The fundamental insights:

**Insight #1:** "We keep looking up the same opcodes. Why can't we just write 'LOAD' instead of '010'?"

**Insight #2:** "We keep calculating addresses. Why can't we name locations like 'total' instead of 'address 47'?"

**Insight #3:** "We keep converting decimal to binary. Why can't the computer do this for us?"

**Insight #4:** "The computer is fast at processing. It should help US program faster!"

### The revolutionary idea forms:

**"What if we invent a NOTATION that humans can READ, and create a PROGRAM that translates it to machine code?"**

### The birth of a concept: THE ASSEMBLER

**The vision:**

Instead of writing:

```
00000010100000100000000000000000
```

Write:

```
LOAD A, 10
```

Then have a program translate:

```
"LOAD A, 10" → 00000010100000100000000000000000
```

### The key realization:

**The computer can do the boring work!**

- Computer can remember "LOAD = 010"
- Computer can convert "10" to binary
- Computer can track which address is which
- Computer can update addresses when code changes
- **Humans just write readable text!**

---

## SUMMARY: THE CRITICAL TRANSITION

### What we learned:

✅ **Machine code made instructions into numbers:**
- Operations = binary patterns (opcodes)
- Addresses = binary numbers
- Entire instruction = 32-bit number
- Stored in memory (Williams Tube, punch tape)

✅ **Physical reality of execution:**
- Fetch: CPU reads instruction from memory
- Decode: Opcode pattern activates control wires
- Execute: Control signals make components act
- All in microseconds!

✅ **The horror of programming in machine code:**
- Must remember opcodes
- Must convert to binary by hand
- Must track addresses manually
- Errors common (40% of instructions!)
- Debugging takes days

✅ **The need emerges:**
- Need human-readable words (not binary)
- Need names for addresses (not numbers)
- Need computer to do translation
- **NEED: ASSEMBLY LANGUAGE!**

Machine code freed us from physical wiring, but it enslaved us to binary. The next step would free us again—by letting us write words instead of numbers, and letting the computer do the tedious translation work.
