### Round 1

**Prompt:**  
for my code and the files attached, i fail a lot of test cases. can u help me see whats wrong?:

```c
#include <stdio.h>
#include <string.h>

// memory address array
static int pas[1000];

int base(int bp, int L) {
    int arb = bp;
    while (L > 0) {
        arb = pas[arb];
        L--;
    }
    return arb;
}

int main(int argc, char* argv[]) {
    // check if the number of inputs in terminal is correct
    if (argc != 2) {
        printf("\nError: incorrect number of arguments\n");
        return 1;
    }

    // open the input file
    FILE* ptr;
    ptr = fopen(argv[1], "r");

    // check if the file opened correctly
    if (ptr == NULL) {
        printf("\nError: cannot open %s\n", argv[1]);
        return 1;
    }

    // initial registers
    int pc = 200, j = 200;
    int bp = 999;
    int sp = 1000;
    int ic = 0;
    int irOP, irL, irM, a, b;
    char mnemonic[4];

    // output
    printf("\tL\tM\tPC\tBP\tSP\tstack\n");
    printf("Initial values:\t%d\t%d\t%d\n", pc, bp, sp);

    // load instructions into pas[] array
    while (fscanf(ptr, "%d", &pas[j]) != EOF) {
        j++;
    }
    ic = (j - 200) / 3;
    fclose(ptr);

    // check if more instructions in text segment
    if ((200 + 3 * ic) > 1000) {
        printf("\nError: program too large for the text segment\n");
        return 1;
    }

    while (1) {
        // check pc out of text bound
        if ((pc >= sp) || (pc >= 1000)) {
            printf("\nError: program counter left the text segment\n");
            return 1;
        }

        // fetch
        irOP = pas[pc];
        irL = pas[pc + 1];
        irM = pas[pc + 2];

        // advance
        pc += 3;

        // execute
        // use a switch to see which instruction to call
        switch (irOP) {
            case 1: // LIT 0, M
                sp--;

                // check stack overflow
                if (sp < (200 + 3 * ic)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                pas[sp] = irM;
                strcpy(mnemonic, "LIT");
                break;

            case 2: // OPR 0, M
                b = pas[sp];
                pas[sp] = 0;
                sp++;

                a = pas[sp];
                pas[sp] = 0;
                sp++;

                // use another switch to see which OPR sub-operation to call
                switch (irM) {
                    case 0: // RTN
                        sp = bp + 1;
                        bp = pas[sp - 2];
                        pc = pas[sp - 3];
                        strcpy(mnemonic, "RTN");
                        break;

                    case 1: // ADD
                        sp--;
                        pas[sp] = a + b;
                        strcpy(mnemonic, "ADD");
                        break;

                    case 2: // SUB
                        sp--;
                        pas[sp] = a - b;
                        strcpy(mnemonic, "SUB");
                        break;

                    case 3: // MUL
                        sp--;
                        pas[sp] = a * b;
                        strcpy(mnemonic, "MUL");
                        break;

                    case 4: // DIV
                        // test error condition
                        if (b == 0) {
                            printf("\nError: division by zero\n");
                            return 1;
                        }

                        sp--;
                        pas[sp] = a / b;
                        strcpy(mnemonic, "DIV");
                        break;

                    case 5: // EQL
                        sp--;
                        if (a == b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "EQL");
                        break;

                    case 6: // NEQ
                        sp--;
                        if (a != b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "NEQ");
                        break;

                    case 7: // LSS
                        sp--;
                        if (a < b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "LSS");
                        break;

                    case 8: // LEQ
                        sp--;
                        if (a <= b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "LEQ");
                        break;

                    case 9: // GTR
                        sp--;
                        if (a > b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "GTR");
                        break;

                    case 10: // GEQ
                        sp--;
                        if (a >= b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "GEQ");
                        break;

                    default: // test error condition
                        printf("\nError: unknown OPR sub-operation\n");
                        return 1;
                }
                break;

            case 3: // LOD L, M
                sp--;

                // check stack overflow
                if (sp < (200 + 3 * ic)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                // check address out of range
                if (((base(bp, irL) - irM) < 200) || ((base(bp, irL) - irM) > 999)) {
                    printf("\nError: data address out of range\n");
                    return 1;
                }

                pas[sp] = pas[base(bp, irL) - irM];
                strcpy(mnemonic, "LOD");
                break;

            case 4: // STO L, M
                // check address out of range
                if (((base(bp, irL) - irM) < 200) || ((base(bp, irL) - irM) > 999)) {
                    printf("\nError: data address out of range\n");
                    return 1;
                }

                pas[base(bp, irL) - irM] = pas[sp];
                sp++;

                strcpy(mnemonic, "STO");
                break;

            case 5: // CAL L, M
                // check stack overflow
                if ((sp - 3) < (200 + 3 * ic)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                pas[sp - 1] = base(bp, irL);
                pas[sp - 2] = bp;
                pas[sp - 3] = pc;
                bp = sp - 1;
                strcpy(mnemonic, "CAL");
                break;

            case 6: // INC 0, M
                sp -= irM;

                // check stack overflow
                if (sp < (200 + 3 * ic)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                strcpy(mnemonic, "INC");
                break;

            case 7: // JMP 0, M
                pc = irM;
                strcpy(mnemonic, "JMP");
                break;

            case 8: // JPC 0, M
                if (pas[sp] == 0) pc = irM;
                sp++;

                strcpy(mnemonic, "JPC");
                break;

            case 9: // SYS 0, M
                // use another switch to see which SYS operation to call
                switch (irM) {
                    case 1: // write
                        printf("Output result is: %d\n", pas[sp]);
                        sp++;
                        break;

                    case 2: // read
                        printf("Please Enter an Integer: ");
                        sp--;

                        // check stack overflow
                        if (sp < (200 + 3 * ic)) {
                            printf("\nError: stack overflow\n");
                            return 1;
                        }

                        scanf("%d", &pas[sp]);
                        printf("%d\n", pas[sp]);
                        break;

                    case 3: // halt
                        return 0;

                    default:
                        printf("\nError: unknown SYS operation\n");
                        return 1;
                }
                strcpy(mnemonic, "SYS");
                break;

            default: // test error condition
                printf("\nError: unknown opcode\n");
                return 1;
        }

        // print mnemonic, L, M, PC, BP, SP
        printf("%s\t%d\t%d\t%d\t%d\t%d\t", mnemonic, irL, irM, pc, bp, sp);

        // print stack contents
        int k;
        for (int i = 999; i >= sp; i--) {
            k = bp;
            // print "|" when base address detected, excluding outermost
            while ((k >= sp) && (k != 999)) {
                if (i == k) {
                    printf("| ");
                    break;
                }
                // find dynamic link below bp
                k = pas[k - 1];
            }
            printf("%d ", pas[i]);
        }
        printf("\n");
    }
}
```

**AI output:**  
here is a gentle look at what is causing test cases to fail in your virtual machine. we can walk through each issue step by step so you know exactly what needs fixing.

* **missing required header comment**: the test script checks for the exact required header block from the pdf. without it, or if author still says your name here, the autograder flags an automatic zero.
* **cal instruction missing pc assignment**: inside `case 5: // CAL`, you properly save the static link, dynamic link, and return address, and set `bp = sp - 1;`, but you forget to jump to the target procedure . you need to set `pc = irM;`.
* **opr 0 / rtn dynamic link restoration**: in your `RTN` implementation:
  ```c
  sp = bp + 1;
  bp = pas[sp - 2];
  pc = pas[sp - 3];
  ```
  because you set `sp = bp + 1` first, `sp - 2` evaluates to `(bp + 1) - 2 = bp - 1`. according to the specification table, `BP = pas[SP - 2]` and `PC = pas[SP - 3]` must be read relative to the *new* `SP` or saved before moving `SP`. specifically, `DL` is located at `bp - 1` and `RA` is at `bp - 2`. in your code, `pas[sp - 3]` evaluates to `bp - 2`, which works for `pc`, but doing this clearly before altering `sp` prevents subtle stack boundary bugs.
* **error timing and no trace lines on error**: the handout specifies that any failing instruction must produce *no* trace line; you must catch errors and exit before printing a trace. currently, in your fetch step:
  ```c
  if ((pc >= sp) || (pc >= 1000))
  ```
  `pc` out of bounds must check whether `pc` is outside the loaded text segment, specifically `if (pc < 200 || pc >= 200 + 3 * ic)`. checking `pc >= sp` triggers false positives when the stack grows downward.
* **stack overflow bound**: stack overflow occurs when `sp <= 200 + 3 * ic - 1` (that is, when `sp` reaches or enters the last address holding code). checking `sp < (200 + 3 * ic)` causes off-by-one errors on boundary tests.
* **data address out of range checks**: in `LOD` and `STO`, the valid data segment is the stack segment (from address 999 down to the code boundary). the check should ensure the target address `addr = base(bp, irL) - irM` is within `[sp, 999]` (or valid stack bounds) rather than `>= 200`.
* **stack trace vertical bar tracking**: in your printing loop, you trace vertical bars by walking `pas[k - 1]`. this only works if your activation records are intact. make sure you trace the dynamic chain correctly from `bp` down to `999` and print `| ` right before every activation record base except the outermost.
* **missing newline formatting on errors**: each error must print an empty line followed by `Error: <message>\n`. make sure every error string has the exact wording from table 4.

here is your verification checklist to guide your updates:

* add the required comment block at the very top of `vm.c` with your name in `Author:`.
* add `pc = irM;` inside `case 5: // CAL`.
* update `case 7: // JMP` and verify `pc` updates cleanly without extra increments.
* update `RTN` to:
  ```c
  sp = bp + 1;
  pc = pas[bp - 2];
  bp = pas[bp - 1];
  ```
* adjust the fetch bound to check if `pc < 200 || pc >= (200 + 3 * ic)`.
* run `./test_vm.sh vm.c` on eustis to confirm all 28 cases pass.

**What I did with it:**  
I used this feedback to identify why my procedure calls were not jumping to their targets by adding `pc = irM;` in `CAL`. I also noted the advice about separating my program counter bounds from the stack pointer.

---

### Round 2

**Prompt:**  
how do i do the error for pc counter leaving bounds bc i got an error

**AI output:**  
the check needs to verify whether `pc` is pointing to an instruction inside the loaded text segment right at the fetch step before reading the instruction registers.

because instructions are loaded starting at address 200 and each instruction takes 3 words, the loaded text segment occupies addresses from 200 up to `200 + 3 * ic - 1`. in addition, instructions are read in 3-word triples (`pc`, `pc + 1`, `pc + 2`), so `pc` must point to the start of a valid instruction triple within those bounds.

1. **place the check at the start of the fetch cycle**  
   right at the top of your `while(1)` loop, before accessing `pas[pc]`, check whether `pc` has fallen below 200 or reached/exceeded the end of the loaded text segment:
   ```c
   // check if pc is outside the loaded text segment
   if (pc < 200 || pc >= (200 + 3 * ic) || (pc - 200) % 3 != 0) {
       printf("\nError: program counter left the text segment\n");
       return 1;
   }
   ```

2. **remove the sp comparison from the pc check**  
   delete `pc >= sp` from your bounds check. comparing `pc` to `sp` belongs exclusively to stack overflow detection, not the program counter check.

3. **verify exact error format and exit code**  
   ensure that on detecting this error, the program prints a leading blank line, the exact string `Error: program counter left the text segment\n`, and immediately returns a non-zero status without tracing the instruction.

to verify the fix, run `./test_vm.sh vm.c` on eustis and check that test case `e08_pc_out_of_bounds` (or the corresponding error case in `test_cases/cases.txt`) passes with exit status non-zero.

**What I did with it:**  
I placed the `pc` boundary condition check right at the beginning of the fetch phase inside the main execution loop and removed `pc >= sp` so stack growth would not trigger it.

---

### Round 3

**Prompt:**  
so i did that but i end up exiting with the error instead of running a test case that should let me pass it normally so im not sure. the test case is this one:
```text
7 0 215
6 0 4
1 0 42
4 1 3
2 0 0
6 0 4
5 0 203
3 0 3
9 0 1
9 0 3
```

**AI output:**  
the error happens because `OPR` is popping two values (`a` and `b`) **before** checking what sub-operation it is running.

in your code:
```c
case 2: // OPR 0, M
    b = pas[sp];
    pas[sp] = 0;
    sp++;

    a = pas[sp];
    pas[sp] = 0;
    sp++;

    switch (irM) {
        case 0: // RTN
            ...
```

for arithmetic and relational operations (like `ADD`, `SUB`, `EQL`), popping two operands is expected. however, **`RTN` (OPR 0, 0) does not take two operands**. 

here is what happens during `c04_cal_rtn_single.txt`:
1. when the procedure is called, its return address `RA = 221` is saved in the stack at address `993`.
2. when the procedure reaches `RTN`, `sp` is at `992`.
3. your code immediately pops `b` at `992` and sets it to `0`, then pops `a` at `993` and sets `pas[993] = 0`.
4. you just overwrote the stored return address with `0`.
5. when `RTN` executes `pc = pas[...]`, it reads that wiped value and sets `pc = 0`.
6. on the very next loop iteration, `pc = 0 < 200`, which immediately triggers `Error: program counter left the text segment`.

**1. move operand popping inside the arithmetic/relational cases**  
only sub-operations `1` through `10` should pop `a` and `b`:
```c
case 2: // OPR 0, M
    switch (irM) {
        case 0: // RTN
            sp = bp + 1;
            bp = pas[sp - 2];
            pc = pas[sp - 3];
            strcpy(mnemonic, "RTN");
            break;

        case 1: // ADD
            b = pas[sp++];
            a = pas[sp++];
            pas[--sp] = a + b;
            strcpy(mnemonic, "ADD");
            break;

        case 2: // SUB
            b = pas[sp++];
            a = pas[sp++];
            pas[--sp] = a - b;
            strcpy(mnemonic, "SUB");
            break;
            
        // repeat operand popping for cases 3 through 10...
```

**2. trace `SYS 0 3` before exiting**  
the specification requires `SYS 0 3` to be printed as the last traced instruction line before terminating with status `0`.

if you `return 0;` inside `case 3:` before reaching `printf`, the trace line is never printed. instead, set a flag like `int halt = 0;`:
* inside `case 3: // halt`: set `halt = 1;`
* at the bottom of the loop, right after printing the trace line:
  ```c
  if (halt) {
      return 0;
  }
  ```
this ensures `SYS 0 3` is printed and stops the loop before `pc` reaches the end of the text segment on the next cycle.

**What I did with it:**  
I learned why my return address was being overwritten with 0 during `RTN` and understood that `SYS 0 3` needed to print its trace line before the virtual machine stopped executing.

---

### Round 4

**Prompt:**  
okay i see. can i just do this before the switch for irM:
```c
if ((irM >= 1) && (irM <= 10)) {
    b = pas[sp];
    pas[sp] = 0;
    sp++;

    a = pas[sp];
    pas[sp] = 0;
    sp++;
}
```

**AI output:**  
yes, that works cleanly and solves the problem directly. 

by guarding operand extraction with `if ((irM >= 1) && (irM <= 10))`:
* `RTN` (`irM == 0`) is untouched, preventing it from overwriting the return address and corrupting the base pointer.
* invalid sub-operations outside `[0, 10]` will bypass the pops and fall through directly into your `default:` branch to report `Error: unknown OPR sub-operation`.
* operations `1` through `10` will receive `b` and `a` in the exact order required (`a` as the second item popped, `b` as the first).

just remember to also handle the `halt` flag so `SYS 0 3` prints its trace line before exiting:
```c
case 2: // OPR 0, M
    if ((irM >= 1) && (irM <= 10)) {
        b = pas[sp];
        pas[sp] = 0;
        sp++;

        a = pas[sp];
        pas[sp] = 0;
        sp++;
    }

    switch (irM) {
        case 0: // RTN
            sp = bp + 1;
            bp = pas[sp - 2];
            pc = pas[sp - 3];
            strcpy(mnemonic, "RTN");
            break;
        // cases 1 to 10 remain unchanged...
```

**What I did with it:**  
I added the `if ((irM >= 1) && (irM <= 10))` conditional block before my inner `switch` in `case 2` so `RTN` wouldn't pop any operands from the stack.

---

### Round 5

**Prompt:**  
in CAL do i put the pc change first or last

**AI output:**  
you must put the `pc` change **last**.

in step 2 of the fetch-execute cycle, `pc` already advanced by 3 before the instruction started executing. that advanced value of `pc` is the return address pointing to the instruction right after the call. 

if you overwrite `pc = irM` first, saving `pas[sp - 3] = pc` will store the procedure's start address rather than the return point, causing `RTN` to loop back to the callee forever.

the exact order inside `case 5: // CAL` is:
```c
case 5: // CAL L, M
    // check stack overflow before writing
    if ((sp - 3) < (200 + 3 * ic)) {
        printf("\nError: stack overflow\n");
        return 1;
    }

    pas[sp - 1] = base(bp, irL); // 1. static link
    pas[sp - 2] = bp;            // 2. dynamic link
    pas[sp - 3] = pc;            // 3. return address (saves the already-advanced PC)
    bp = sp - 1;                 // 4. set new base pointer
    pc = irM;                    // 5. jump to target address last

    strcpy(mnemonic, "CAL");
    break;
```

**What I did with it:**  
I placed `pc = irM;` as the last statement in my `CAL` implementation to make sure `pas[sp - 3]` accurately stores the incremented return address.

---

### Round 6

**Prompt:**  
idk its acting weird idk:

```c
#include <stdio.h>
#include <string.h>

// memory address array
static int pas[1000];

int base(int bp, int L) {
    int arb = bp;
    while (L > 0) {
        arb = pas[arb];
        L--;
    }
    return arb;
}

int main(int argc, char* argv[]) {
    // check if the number of inputs in terminal is correct
    if (argc != 2) {
        printf("\nError: incorrect number of arguments\n");
        return 1;
    }

    // open the input file
    FILE* ptr;
    ptr = fopen(argv[1], "r");

    // check if the file opened correctly
    if (ptr == NULL) {
        printf("\nError: cannot open %s\n", argv[1]);
        return 1;
    }

    // initial registers
    int pc = 200, j = 200;
    int bp = 999;
    int sp = 1000;
    int ic = 0, halt = 0;
    int irOP, irL, irM, a, b;
    char mnemonic[4];

    // output
    printf("\tL\tM\tPC\tBP\tSP\tstack\n");
    printf("Initial values:\t%d\t%d\t%d\n", pc, bp, sp);

    // load instructions into pas[] array
    while (fscanf(ptr, "%d", &pas[j]) != EOF) {
        j++;
    }
    ic = (j - 200) / 3;
    fclose(ptr);

    // check if more instructions in text segment
    if ((200 + 3 * ic) > 1000) {
        printf("\nError: program too large for the text segment\n");
        return 1;
    }

    while (1) {
        // check pc out of text bound
        if ((pc < 200) || (pc >= (200 + 3 * ic))) {
            printf("\nError: program counter left the text segment\n");
            return 1;
        }

        // fetch
        irOP = pas[pc];
        irL = pas[pc + 1];
        irM = pas[pc + 2];

        // advance
        pc += 3;

        // execute
        // use a switch to see which instruction to call
        switch (irOP) {
            case 1: // LIT 0, M
                sp--;

                // check stack overflow
                if (sp <= (200 + 3 * ic - 1)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                pas[sp] = irM;
                strcpy(mnemonic, "LIT");
                break;

            case 2: // OPR 0, M
                if ((irM >= 1) && (irM <= 10)) {
                    b = pas[sp];
                    pas[sp] = 0;
                    sp++;

                    a = pas[sp];
                    pas[sp] = 0;
                    sp++;
                }

                // use another switch to see which OPR sub-operation to call
                switch (irM) {
                    case 0: // RTN
                        sp = bp + 1;
                        bp = pas[bp - 1];
                        pc = pas[bp - 2];
                        strcpy(mnemonic, "RTN");
                        break;

                    case 1: // ADD
                        sp--;
                        pas[sp] = a + b;
                        strcpy(mnemonic, "ADD");
                        break;

                    case 2: // SUB
                        sp--;
                        pas[sp] = a - b;
                        strcpy(mnemonic, "SUB");
                        break;

                    case 3: // MUL
                        sp--;
                        pas[sp] = a * b;
                        strcpy(mnemonic, "MUL");
                        break;

                    case 4: // DIV
                        // test error condition
                        if (b == 0) {
                            printf("\nError: division by zero\n");
                            return 1;
                        }

                        sp--;
                        pas[sp] = a / b;
                        strcpy(mnemonic, "DIV");
                        break;

                    case 5: // EQL
                        sp--;
                        if (a == b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "EQL");
                        break;

                    case 6: // NEQ
                        sp--;
                        if (a != b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "NEQ");
                        break;

                    case 7: // LSS
                        sp--;
                        if (a < b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "LSS");
                        break;

                    case 8: // LEQ
                        sp--;
                        if (a <= b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "LEQ");
                        break;

                    case 9: // GTR
                        sp--;
                        if (a > b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "GTR");
                        break;

                    case 10: // GEQ
                        sp--;
                        if (a >= b) pas[sp] = 1;
                        else pas[sp] = 0;
                        strcpy(mnemonic, "GEQ");
                        break;

                    default: // test error condition
                        printf("\nError: unknown OPR sub-operation\n");
                        return 1;
                }
                break;

            case 3: // LOD L, M
                sp--;

                // check stack overflow
                if (sp <= (200 + 3 * ic - 1)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                // check address out of range
                if (((base(bp, irL) - irM) < sp) || ((base(bp, irL) - irM) > 999)) {
                    printf("\nError: data address out of range\n");
                    return 1;
                }

                pas[sp] = pas[base(bp, irL) - irM];
                strcpy(mnemonic, "LOD");
                break;

            case 4: // STO L, M
                // check address out of range
                if (((base(bp, irL) - irM) < sp) || ((base(bp, irL) - irM) > 999)) {
                    printf("\nError: data address out of range\n");
                    return 1;
                }

                pas[base(bp, irL) - irM] = pas[sp];
                sp++;

                strcpy(mnemonic, "STO");
                break;

            case 5: // CAL L, M
                // check stack overflow
                if ((sp - 3) <= (200 + 3 * ic - 1)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                pas[sp - 1] = base(bp, irL);
                pas[sp - 2] = bp;
                pas[sp - 3] = pc;
                bp = sp - 1;
                pc = irM;
                strcpy(mnemonic, "CAL");
                break;

            case 6: // INC 0, M
                sp -= irM;

                // check stack overflow
                if (sp <= (200 + 3 * ic - 1)) {
                    printf("\nError: stack overflow\n");
                    return 1;
                }

                strcpy(mnemonic, "INC");
                break;

            case 7: // JMP 0, M
                pc = irM;
                strcpy(mnemonic, "JMP");
                break;

            case 8: // JPC 0, M
                if (pas[sp] == 0) pc = irM;
                sp++;

                strcpy(mnemonic, "JPC");
                break;

            case 9: // SYS 0, M
                // use another switch to see which SYS operation to call
                switch (irM) {
                    case 1: // write
                        printf("Output result is: %d\n", pas[sp]);
                        sp++;
                        break;

                    case 2: // read
                        printf("Please Enter an Integer: ");
                        sp--;

                        // check stack overflow
                        if (sp <= (200 + 3 * ic - 1)) {
                            printf("\nError: stack overflow\n");
                            return 1;
                        }

                        scanf("%d", &pas[sp]);
                        printf("%d\n", pas[sp]);
                        break;

                    case 3: // halt
                        halt = 1;
                        break;

                    default:
                        printf("\nError: unknown SYS operation\n");
                        return 1;
                }
                strcpy(mnemonic, "SYS");
                break;

            default: // test error condition
                printf("\nError: unknown opcode\n");
                return 1;
        }

        // print mnemonic, L, M, PC, BP, SP
        printf("%s\t%d\t%d\t%d\t%d\t%d\t", mnemonic, irL, irM, pc, bp, sp);

        // print stack contents
        int k;
        for (int i = 999; i >= sp; i--) {
            k = bp;
            // print "|" when base address detected, excluding outermost
            while ((k >= sp) && (k != 999)) {
                if (i == k) {
                    printf("| ");
                    break;
                }
                // find dynamic link below bp
                k = pas[k - 1];
            }
            printf("%d ", pas[i]);
        }
        printf("\n");
        if (halt) return 0;
    }
}
```

**AI output:**  
The issue is inside your `RTN` case:
```c
case 0: // RTN
    sp = bp + 1;
    bp = pas[bp - 1];
    pc = pas[bp - 2];
    strcpy(mnemonic, "RTN");
    break;
```

Look closely at the order:
1. `bp = pas[bp - 1];` updates `bp` to the caller's base pointer (for example, `999`).
2. On the very next line, `pc = pas[bp - 2];` uses the **new** `bp` (`999 - 2 = 997`)!
3. `pas[997]` has never been set, so it reads `0`.
4. `pc` gets set to `0`, and on the next iteration of the `while` loop, `pc < 200` triggers `Error: program counter left the text segment`.

Save `pc` **before** you overwrite `bp` (or use temporary variables):
```c
case 0: // RTN
    sp = bp + 1;
    pc = pas[sp - 3]; // or pas[bp - 2] before updating bp
    bp = pas[sp - 2]; // or pas[bp - 1]
    strcpy(mnemonic, "RTN");
    break;
```

---

There is also a second latent bug in your `LOD` and `STO` checks:
```c
// in LOD and STO:
if (((base(bp, irL) - irM) < sp) || ((base(bp, irL) - irM) > 999))
```

In `LOD`, you decrement `sp--` before checking the address. Checking `< sp` or checking against `sp` instead of the stack boundary will cause valid stack reads to fail or trigger false errors on other test cases. 

According to Section 5, `data address out of range` means the computed address points outside the stack segment (i.e. into the text segment or system reserved area). Compute the target address first, and verify it stays above the text segment and within memory bounds:
```c
case 3: // LOD L, M
    sp--;
    if (sp <= (200 + 3 * ic - 1)) {
        printf("\nError: stack overflow\n");
        return 1;
    }

    int lod_addr = base(bp, irL) - irM;
    if (lod_addr < (200 + 3 * ic) || lod_addr > 999) {
        printf("\nError: data address out of range\n");
        return 1;
    }

    pas[sp] = pas[lod_addr];
    strcpy(mnemonic, "LOD");
    break;

case 4: // STO L, M
    int sto_addr = base(bp, irL) - irM;
    if (sto_addr < (200 + 3 * ic) || sto_addr > 999) {
        printf("\nError: data address out of range\n");
        return 1;
    }

    pas[sto_addr] = pas[sp];
    sp++;
    strcpy(mnemonic, "STO");
    break;
```

Fixing the order of `pc` and `bp` in `RTN` will immediately resolve the `pc` error on `c04_cal_rtn_single.txt`.

**What I did with it:**  
I fixed the line ordering in `RTN` so `pc` was restored before `bp` was updated, which kept `pc` from reading garbage data and triggering out-of-bounds errors. I also adjusted my data address range checks in `LOD` and `STO`.

---

### Round 7

**Prompt:**  
okay ty i see now. it works for that case. for the next one tho, i run into an issue with the printing "|" bc the data is correct but my program doesnt print a "|" when it should for the following file:
```text
7 0 224
6 0 4
5 0 212
2 0 0
6 0 4
1 0 77
4 2 3
2 0 0
6 0 4
5 0 203
3 0 3
9 0 1
9 0 3
```

**AI output:**  
The missing `|` is caused by the loop condition `while((k >= sp) && (k != 999))`.

Look at what happens right after `CAL` executes:
1. `CAL` sets `bp = sp - 1`.
2. Because `sp` has not been decremented yet (the callee hasn't run its `INC` yet), **`bp < sp`** (for instance, `bp = 991` while `sp = 992`).
3. When `i` begins looping down from `999` to print the stack, `k` starts at `bp = 991`.
4. Your condition checks `k >= sp` (`991 >= 992`), which evaluates to **false on the very first check**.
5. The `while` loop aborts immediately without ever following `k = pas[k - 1]` up to the parent frames (`995`), completely skipping the `|` for the outer frames.

The dynamic chain points **upward** toward higher addresses (`991 -> 995 -> 999`), so checking `k >= sp` incorrectly cuts off the chain whenever the newest frame base sits below `sp`.

Change the inner loop condition to simply follow the chain until it reaches the outermost frame base `999`:
```c
// print stack contents
for (int i = 999; i >= sp; i--) {
    int k = bp;
    // walk up dynamic links until the outermost frame (999)
    while (k > 0 && k != 999) {
        if (i == k) {
            printf("| ");
            break;
        }
        k = pas[k - 1]; // follow dynamic link
    }
    printf("%d ", pas[i]);
}
printf("\n");
```

**What I did with it:**  
I modified the dynamic link while loop in the stack trace printer to `while (k > 0 && k != 999)` so it would traverse all the way up the stack frames and print the `|` divider accurately.

---

### Round 8

**Prompt:**  
tysm i passed almost all the cases i have a 95 on the scoring rn tysm again

**AI output:**  
getting a 95/100 on the autograder means the machine's core fetch-execute cycle, activation record dynamics, and control flow are practically solid. 

to clean up the remaining 5 points, check the exact diffs generated in the test folder:
* inspect the specific failed case by running:
  ```bash
  cat failed_cases/<case_name>/diff.txt
  ```
* check `failed_cases/<case_name>/info.txt` to verify whether the deduction came from an exit status mismatch (e.g., returning 0 instead of non-zero on an error case) or a tiny token mismatch.
* verify relational operators in `OPR`: confirm `LEQ` (case 8) is strictly `<=` and `GEQ` (case 10) is strictly `>=`.
* check the required header block at the very top of `vm.c` to confirm `Author:` has your name filled in instead of the placeholder text.

**What I did with it:**  
I checked the failed test folder diffs and verified my relational operator edge cases to get the remaining points on the autograder.

---

### Round 9

**Prompt:**  
yay i did it its complete

**AI output:**  
congratulations on getting the virtual machine fully running. navigating dynamic links, the activation record stack, and the fetch-execute cycle in raw c is no small feat, and pulling together a full 100 on the autograder takes real persistence.

before submitting to webcourses, run through this quick final verification checklist:
* verify the file name is strictly `vm.c` in all lowercase.
* check the header comment block at the very top of `vm.c` to ensure `Author:` contains your full name rather than the default template text.
* run the clean compile check on eustis without any warnings:
  ```bash
  gcc -Wall -Wextra -std=c11 -O2 vm.c -o vm
  ```
* run `./test_vm.sh vm.c` one final time to confirm all 28 test cases and code requirement checks report clean passes.
* upload only the required files to webcourses (including the signed ai usage disclosure form and conversation log if required by the assignment guidelines).

**What I did with it:**  
I checked my filename, verified the header comment had my name, and recompiled cleanly on Eustis with the required flags.
