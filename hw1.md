for my code and the files attached, i fail a lot of test cases. can u help me see whats wrong?: 
'''
#include<stdio.h>

#include<string.h>



//memory address array

static int pas[1000];



int base(int bp, int L)

{

int arb = bp;

while (L > 0) {

arb = pas[arb];

L--;

}

return arb;

}



int main(int argc, char* argv[])

{

//check if the number of inputs in terminal is correct

if(argc != 2)

{

printf("\nError: incorrect number of arguments\n");

return 1;

}



//open the input file

FILE* ptr;

ptr = fopen(argv[1], "r");



//check if the file opened correctly

if(ptr == NULL)

{

printf("\nError: cannot open %s\n", argv[1]);

return 1;

}



//initial registers

int pc = 200, j = 200;

int bp = 999;

int sp = 1000;

int ic = 0;

int irOP, irL, irM, a, b;

char mnemonic[4];



//output

printf("\tL\tM\tPC\tBP\tSP\tstack\n");

printf("Initial values:\t%d\t%d\t%d\n", pc, bp, sp);



//load instructions into pas[] array

while(fscanf(ptr, "%d", &pas[j]) != EOF) {

j++;

}

ic = (j - 200) / 3;

fclose(ptr);



//check if more instructions in text segment

if((200 + 3 * ic) > 1000) {

printf("\nError: program too large for the text segment\n");

return 1;

}



while(1) {

//check pc out of text bound

if((pc >= sp) || (pc >= 1000)) {

printf("\nError: program counter left the text segment\n");

return 1;

}



//fetch

irOP = pas[pc];

irL = pas[pc + 1];

irM = pas[pc + 2];



//advance

pc += 3;



//execute

//use a switch to see which instruction to call

switch (irOP) {

case 1: //LIT 0, M

sp--;



//check stack overflow

if(sp < (200 + 3 * ic)) {

printf("\nError: stack overflow\n");

return 1;

}



pas[sp] = irM;

strcpy(mnemonic, "LIT");

break;



case 2: //OPR 0, M

b = pas[sp];

pas[sp] = 0;

sp++;



a = pas[sp];

pas[sp] = 0;

sp++;



//use another switch to see which OPR sub-operation to call

switch (irM) {

case 0: //RTN

sp = bp + 1;

bp = pas[sp - 2];

pc = pas[sp - 3];

strcpy(mnemonic, "RTN");

break;



case 1: //ADD

sp--;

pas[sp] = a + b;

strcpy(mnemonic, "ADD");

break;



case 2: //SUB

sp--;

pas[sp] = a - b;

strcpy(mnemonic, "SUB");

break;



case 3: //MUL

sp--;

pas[sp] = a * b;

strcpy(mnemonic, "MUL");

break;



case 4: //DIV

//test error condition

if(b == 0) {

printf("\nError: division by zero\n");

return 1;

}



sp--;

pas[sp] = a / b;

strcpy(mnemonic, "DIV");

break;



case 5: //EQL

sp--;

if(a == b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "EQL");

break;



case 6: //NEQ

sp--;

if(a != b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "NEQ");

break;



case 7: //LSS

sp--;

if(a < b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "LSS");

break;



case 8: //LEQ

sp--;

if(a <= b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "LEQ");

break;



case 9: //GTR

sp--;

if(a > b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "GTR");

break;



case 10: //GEQ

sp--;

if(a >= b) pas[sp] = 1;

else pas[sp] = 0;

strcpy(mnemonic, "GEQ");

break;



default: //test error condition

printf("\nError: unknown OPR sub-operation\n");

return 1;

}

break;



case 3: //LOD L, M

sp--;



//check stack overflow

if(sp < (200 + 3 * ic)) {

printf("\nError: stack overflow\n");

return 1;

}



//check address out of range

if(((base(bp, irL) - irM) < 200) || ((base(bp, irL) - irM) > 999)) {

printf("\nError: data address out of range\n");

return 1;

}



pas[sp] = pas[base(bp, irL) - irM];

strcpy(mnemonic, "LOD");

break;



case 4: //STO L, M

//check address out of range

if(((base(bp, irL) - irM) < 200) || ((base(bp, irL) - irM) > 999)) {

printf("\nError: data address out of range\n");

return 1;

}



pas[base(bp, irL) - irM] = pas[sp];

sp++;



strcpy(mnemonic, "STO");

break;



case 5: //CAL L, M

//check stack overflow

if((sp - 3) < (200 + 3 * ic)) {

printf("\nError: stack overflow\n");

return 1;

}



pas[sp - 1] = base(bp, irL);

pas[sp - 2] = bp;

pas[sp - 3] = pc;

bp = sp - 1;

strcpy(mnemonic, "CAL");

break;



case 6: //INC 0, M

sp -= irM;



//check stack overflow

if(sp < (200 + 3 * ic)) {

printf("\nError: stack overflow\n");

return 1;

}



strcpy(mnemonic, "INC");

break;



case 7: //JMP 0, M

pc = irM;

strcpy(mnemonic, "JMP");

break;



case 8: //JPC 0, M

if(pas[sp] == 0) pc = irM;

sp++;



strcpy(mnemonic, "JPC");

break;



case 9: //SYS 0, M

//use another switch to see which SYS operation to call

switch (irM) {

case 1: //write

printf("Output result is: %d\n", pas[sp]);

sp++;

break;



case 2: //read

printf("Please Enter an Integer: ");

sp--;



//check stack overflow

if(sp < (200 + 3 * ic)) {

printf("\nError: stack overflow\n");

return 1;

}



scanf("%d", &pas[sp]);

printf("%d\n", pas[sp]);

break;



case 3: //halt

return 0;



default:

printf("\nError: unknown SYS operation\n");

return 1;

}

strcpy(mnemonic, "SYS");

break;



default: //test error condition

printf("\nError: unknown opcode\n");

return 1;

}


//print mnemonic, L, M, PC, BP, SP

printf("%s\t%d\t%d\t%d\t%d\t%d\t", mnemonic, irL, irM, pc, bp, sp);


//print stack contents

int k;

for(int i = 999; i >= sp; i--) {

k = bp;

//print "|" when base address detected, excluding outermost

while((k >= sp) && (k != 999)) {

if(i == k) {

printf("| ");

break;

}

//find dynamic link below bp

k = pas[k - 1];

}

printf("%d ", pas[i]);

}

printf("\n");

}

} 

'''
