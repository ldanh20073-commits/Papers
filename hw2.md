---

## Round 1

### Prompt
hi i was wondering if you could help me approach my program with the file attached. i have the lexeme and token matching working but im not sure what to write for the output file and also my token list is having trouble

```c
#include<stdio.h>
#include<string.h>

//checks and returns if word is a single reserved word
int isSingleKeyWord(char character, char single[], int numSingle) {
    for(int i = 0; i < numSingle; i++) {
        if(character == single[i]) {
            return 1;
        }
    }
    return 0;
}

//checks if a string is only digits
int isOnlyDigits(char lexT[]) {
    //if string is empty, not only digits
    if(lexT[0] == '\0') return 0;
    int k = 0;
    while(lexT[k] != '\0') {
        if(!((lexT[k] >= '0') && (lexT[k] <= '9'))) return 0;
        k++;
    }
    return 1;
}

//checks if a code 1 token is in name token array
int notInName(char lexT[], int numTokens, char nameTokens[][10]) {
    for(int i = 0; i < numTokens; i++) {
        if(strcmp(lexT, nameTokens[i]) == 0) return 0;
    }
    return 1;
}

int main(int argc, char *argv[]) {
    //check if argc is valid
    if(argc != 2) {
        printf("Usage: ./lex <input file>\n");
        return 1;
    }
    //open the input file
    FILE* ptr = fopen(argv[1], "r");
    //check if input file is valid
    if(ptr == NULL) {
        printf("Error: unable to open input file '%s'\n", argv[1]);
        return 1;
    }
    //variables to track line, column, and text
    int line = 0;
    int column = 0;
    int maxTokenLen = 10;
    int maxTokens = 1000;
    //array to store name tokens
    char nameTokens[maxTokens][maxTokenLen];
    int nameElements = 0;
    //token array for symbols (excludes letter and digit case: codes 1, 2)
    char singleKeys[] = {'+', '-', '*', '/', '(', ')', '=', ',', '.', '<', '>', ';', ':', '!'};
    int numSingle = 14;
    int numMulti = 22;
    //2D array to store input text and parse
    char readChar;
    char rTemp;
    char tokens[maxTokens][maxTokenLen];
    int numTokens = 0;
    int index = 0;
    char temp[maxTokenLen];
    //arrays to store the line and column for name tokens
    int lineArr[maxTokens];
    int colArr[maxTokens];
    int numLine = 0;
    int numCol = 0;
    //ISSUE: figure out how to store line and col while reading file in while loop below
    //output
    printf("\nSource Program:\n\n");
    int largestLine = 0;
    int largestCol = 0;
    //scan through and print the input file
    while(fscanf(ptr, "%c", &readChar) != EOF) {
        printf("%c", readChar);
        column++;
        //check for new line in input file
        if(readChar == '\n') {
            line++;
            if(column > largestCol) {
                largestCol = column;
            }
            column = 0;
        }
        //check if char is an alphanumeric char
        if(((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')) || ((readChar >= '0') && (readChar <= '9'))) {
            temp[index] = readChar;
            index++;
        }
        //check if char is a white space and add if new word is being scanned
        else if(((readChar == ' ') || (readChar == '\n') || (readChar == '\t') || (readChar == '\r')) && (index > 0)) {
            //end building the current word
            temp[index] = '\0';
            strcpy(tokens[numTokens], temp);
            numTokens++;
            index = 0;
        }
        //each if char is a reserved single token
        else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
            //add current word to token array
            if(index > 0) {
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                numTokens++;
                index = 0;
            }
            //check if next char makes a longer operator
            if((readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
                fscanf(ptr, "%c", &rTemp);
                //check if next operator is '='
                if(rTemp == '=') {
                    printf("%c", rTemp);
                    column++;
                    //add long operator to token array
                    temp[0] = readChar;
                    temp[1] = '=';
                    temp[2] = '\0';
                    strcpy(tokens[numTokens], temp);
                    numTokens++;
                }
                //store reserved char into array as well
                else {
                    //check if rTemp was not a '='
                    if(rTemp != '=') {
                        //put scanned char back into input stream
                        ungetc(rTemp, ptr);
                    }
                    //store a single char
                    temp[0] = readChar;
                    temp[1] = '\0';
                    strcpy(tokens[numTokens], temp);
                    numTokens++;
                }
            }
            //store a single char
            else {
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                numTokens++;
            }
        }
    }
    //add last word to token array once scanning finishes
    if(index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        numTokens++;
        index = 0;
    }
    printf("\n");
    printf("\nLexeme Table:\n\n");
    printf("lexeme\t\ttoken\n");
    char lexT[maxTokenLen];
    int code = 0;
    int tokenCode[maxTokens];
    int numCodes = 0;
    //loop through token array and print token and code
    for(int i = 0; i < numTokens; i++) {
        strcpy(lexT, tokens[i]);
        if(strcmp(lexT, "+") == 0) code = 3;
        else if(strcmp(lexT, "-") == 0) code = 4;
        else if(strcmp(lexT, "*") == 0) code = 5;
        else if(strcmp(lexT, "/") == 0) code = 6;
        else if(strcmp(lexT, "==") == 0) code = 7;
        else if(strcmp(lexT, "!=") == 0) code = 8;
        else if(strcmp(lexT, "<") == 0) code = 9;
        else if(strcmp(lexT, "<=") == 0) code = 10;
        else if(strcmp(lexT, ">") == 0) code = 11;
        else if(strcmp(lexT, ">=") == 0) code = 12;
        else if(strcmp(lexT, "(") == 0) code = 13;
        else if(strcmp(lexT, ")") == 0) code = 14;
        else if(strcmp(lexT, ",") == 0) code = 15;
        else if(strcmp(lexT, ";") == 0) code = 16;
        else if(strcmp(lexT, ".") == 0) code = 17;
        else if(strcmp(lexT, "=") == 0) code = 18;
        else if(strcmp(lexT, ":=") == 0) code = 19;
        else if(strcmp(lexT, "begin") == 0) code = 20;
        else if(strcmp(lexT, "end") == 0) code = 21;
        else if(strcmp(lexT, "if") == 0) code = 22;
        else if(strcmp(lexT, "fi") == 0) code = 23;
        else if(strcmp(lexT, "then") == 0) code = 24;
        else if(strcmp(lexT, "while") == 0) code = 25;
        else if(strcmp(lexT, "elihw") == 0) code = 26;
        else if(strcmp(lexT, "do") == 0) code = 27;
        else if(strcmp(lexT, "od") == 0) code = 28;
        else if(strcmp(lexT, "odd") == 0) code = 29;
        else if(strcmp(lexT, "call") == 0) code = 30;
        else if(strcmp(lexT, "const") == 0) code = 31;
        else if(strcmp(lexT, "var") == 0) code = 32;
        else if(strcmp(lexT, "procedure") == 0) code = 33;
        else if(strcmp(lexT, "write") == 0) code = 34;
        else if(strcmp(lexT, "read") == 0) code = 35;
        else if(strcmp(lexT, "else") == 0) code = 36;
        else if(isOnlyDigits(lexT)) code = 2;
        else {
            //check if lexT is in name token array already
            if(notInName(lexT, nameElements, nameTokens)) {
                strcpy(nameTokens[nameElements], lexT);
                tokenCode[numCodes] = nameElements;
                numCodes++;
                nameElements++;
            }
            code = 1;
        }
        tokenCode[numCodes] = code;
        numCodes++;
        printf("%s\t\t%d\n", lexT, code);
    }
    printf("\n");
    printf("\nName Table:\n\n");
    printf("index name\t\tline column\n");
    for(int i = 0; i < nameElements; i++) {
        printf("%d %s\t\t\t%d %d\n", i, nameTokens[i], line, column);
    }
    printf("\n");
    printf("\nToken List:\n\n");
    //print out the token codes in tokenCode[]
    for(int i = 0; i < numCodes; i++) {
        printf("%d ", tokenCode[i]);
    }
    printf("\n");
}
```

### AI Output
the issues in the token list and file writing come from how the scanner structures token data and how positions are tracked.

#### 1. why the token list is having trouble
in your classification loop:
```c
else {
    if(notInName(lexT, nameElements, nameTokens)) {
        strcpy(nameTokens[nameElements], lexT);
        tokenCode[numCodes] = nameElements; // here!
        numCodes++;
        nameElements++;
    }
    code = 1;
    tokenCode[numCodes] = code;
    numCodes++;
}
```
there are three separate bugs causing the token list to drift:
1. **reversed token/index order:** for identifiers (code 1), the specification requires printing code 1 first, followed by its name table index (e.g., `1 0`). in your loop, you store `nameElements` first, then store `code = 1` resulting in `0 1`.
2. **missing index on repeat occurrences:** when an identifier already exists in `nameTokens`, `notInName(...)` returns false. your code skips the block and never inserts the existing index into `tokenCode`, so repeat identifiers output only `1` instead of `1 <index>`.
3. **numbers require their value in the token list:** numbers ($code = 2$) must be followed by their literal digits (e.g., `2 5` for the number 5). storing only `code = 2` leaves the digits out of the token stream.

#### 2. writing tokens.txt and nametable.txt
both files must be written in the working directory, even when an error occurs (holding whatever was scanned up to that point):
* **tokens.txt:** one token per line.
  * symbols / reserved words: `<code_number>` alone.
  * identifiers: `1 <index>`.
  * numbers: `2 <digits>`.
* **nametable.txt:** one entry per line formatted as `<index> <name> <line> <column>` separated by single spaces.

#### 3. tracking line and column accurately
in your current code, `printf("%d %s\t\t\t%d %d\n", i, nameTokens[i], line, column);` prints whatever final line and column counter the file scan ended on.
lines and columns must record the start position of the first character where the token began:
* lines start at 1; columns start at 1.
* `\n`: increment line, reset column = 1.
* `\r`: advance nothing (do not increment column or line).
* `\t` and regular characters: advance column by 1.

to fix this, record `start_line = line` and `start_col = column` at the moment `index == 0` when starting a token.

#### 4. recommended token data structure
instead of parallel raw string arrays, represent tokens as structures:
```c
typedef struct {
    char lexeme[64];
    int token_type;
    int line;
    int col;
    int name_index; // used if token_type == 1
} Token;

typedef struct {
    char name[64];
    int line;
    int col;
} NameEntry;
```

#### concrete next steps
1. **fix line/column counters:** initialize `line = 1` and `column = 1` before the reading loop. do not increment column on `\r`.
2. **track start coordinates:** record `token_col = column` and `token_line = line` at character 0 of every lexeme.
3. **record name table indices:** when encountering an identifier, search `nameTokens`. if found, record its existing index; if not, add it with `start_line` and `start_col` and record the new index.
4. **file output routine:** create a function `write_output_files(Token *tokens, int num_tokens, NameEntry *names, int name_count)` that opens `tokens.txt` and `nametable.txt` using `fopen` with mode `"w"` and loops over the collected structs.

### What I did with it
I realized why my token list was missing repeat indices and printing numbers incorrectly, and I noted the file writing rules and line/column 1-based indexing requirements.

---

## Round 2

### Prompt
im not sure how to structure it bc i want to be able to keep the array stuff i've been using without wiping it or using structs

### AI Output
you can keep your exact 2d array setup without structs by using parallel arrays to store each token's type, value, and position.

#### parallel arrays to add at the top
keep your existing arrays and add parallel tracking arrays:
```c
// existing arrays
char tokens[maxTokens][maxTokenLen];
int numTokens = 0;

// token metadata arrays (same index as tokens)
int tokenTypes[maxTokens];
int tokenValOrIndex[maxTokens]; // stores name index for idents, or digit/value if needed
int tokenLine[maxTokens];
int tokenCol[maxTokens];

// name table arrays
char nameTokens[maxTokens][maxTokenLen];
int nameLine[maxTokens];
int nameCol[maxTokens];
int nameElements = 0;
```

#### tracking position during scanning
initialize counters to 1 before reading:
```c
int line = 1;
int column = 1;
int startLine = 1;
int startCol = 1;
```
update position rules on character reads:
* `\r`: do not advance column or line.
* `\n`: advance `line++`, reset `column = 1`.
* all other characters: advance `column++`.

when `index == 0` right before adding a character to `temp`, capture the starting position:
```c
if (index == 0) {
    startLine = line;
    startCol = column;
}
```
when you save a token into `tokens[numTokens]`:
```c
tokenLine[numTokens] = startLine;
tokenCol[numTokens] = startCol;
```

#### updating the token classification loop
adjust your matching loop to record indices and name table data properly:
```c
for (int i = 0; i < numTokens; i++) {
    strcpy(lexT, tokens[i]);
    // reserved word and operator checks (code 3 to 36)...
    if (strcmp(lexT, "+") == 0) code = 3;
    // ... all other keywords / operators
    else if (isOnlyDigits(lexT)) {
        code = 2;
    }
    else {
        // identifier
        code = 1;
        int foundIndex = -1;
        for (int k = 0; k < nameElements; k++) {
            if (strcmp(lexT, nameTokens[k]) == 0) {
                foundIndex = k;
                break;
            }
        }
        // if not seen before, add new entry to name table
        if (foundIndex == -1) {
            foundIndex = nameElements;
            strcpy(nameTokens[nameElements], lexT);
            nameLine[nameElements] = tokenLine[i];
            nameCol[nameElements] = tokenCol[i];
            nameElements++;
        }
        tokenValOrIndex[i] = foundIndex;
    }
    tokenTypes[i] = code;
}
```

#### printing standard output and writing the files
after classifying all tokens:
```c
// 1. print lexeme table
printf("\nLexeme Table:\n\nlexeme\t\ttoken\n");
for (int i = 0; i < numTokens; i++) {
    printf("%s\t\t%d\n", tokens[i], tokenTypes[i]);
}

// 2. print name table
printf("\nName Table:\n\nindex\tname\t\tline\tcolumn\n");
for (int i = 0; i < nameElements; i++) {
    printf("%d\t%s\t\t%d\t%d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
}

// 3. print token list
printf("\nToken List:\n\n");
for (int i = 0; i < numTokens; i++) {
    if (tokenTypes[i] == 1) {
        printf("%d %d ", tokenTypes[i], tokenValOrIndex[i]);
    } else if (tokenTypes[i] == 2) {
        printf("%d %s ", tokenTypes[i], tokens[i]);
    } else {
        printf("%d ", tokenTypes[i]);
    }
}
printf("\n");

// 4. write tokens.txt
FILE *f_tok = fopen("tokens.txt", "w");
if (f_tok != NULL) {
    for (int i = 0; i < numTokens; i++) {
        if (tokenTypes[i] == 1) {
            fprintf(f_tok, "%d %d\n", tokenTypes[i], tokenValOrIndex[i]);
        } else if (tokenTypes[i] == 2) {
            fprintf(f_tok, "%d %s\n", tokenTypes[i], tokens[i]);
        } else {
            fprintf(f_tok, "%d\n", tokenTypes[i]);
        }
    }
    fclose(f_tok);
}

// 5. write nametable.txt
FILE *f_name = fopen("nametable.txt", "w");
if (f_name != NULL) {
    for (int i = 0; i < nameElements; i++) {
        fprintf(f_name, "%d %s %d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
    }
    fclose(f_name);
}
```

#### execution checklist
1. add `tokenTypes`, `tokenValOrIndex`, `tokenLine`, and `tokenCol` next to `tokens[maxTokens][maxTokenLen]`.
2. set `line = 1` and `column = 1` initially; guard `column++` so `\r` does not increase it.
3. set `startLine` and `startCol` on the first character of each token and copy them into `tokenLine[numTokens]` and `tokenCol[numTokens]`.
4. replace `notInName` with the index-returning search loop so repeat identifiers receive their existing index.
5. paste the file writing blocks after the printing sections.

### What I did with it
I implemented parallel arrays for index storage and also updated the name table printing.

---

## Round 3

### Prompt
im struggling with this im not sure where to start can we start from the beginning and go through what i should be putting

### AI Output
take a breath—we will break this down step-by-step from the very top, keeping your simple array style intact so you never have to guess what goes where.

#### the full scanner blueprint
a pl/0 scanner does five clean jobs in order:
1. **validate arguments & open file:** check that `argc == 2`, print the exact usage message and exit with 1 if not.
2. **read & echo source program:** read the whole source file into a buffer and print it under the `Source Program:` header.
3. **scan loop (tokenize):** walk through character by character, tracking line and column. skip whitespace and comments. grab numbers, words, and symbols while recording their starting position.
4. **classify & build name table:** distinguish reserved words from identifiers. assign token codes and track unique identifier indices.
5. **print tables & write files:** print the three remaining tables to standard output, then write `tokens.txt` and `nametable.txt`.

#### step 1: headers, definitions, and arrays
put these declarations right above main:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_TOKENS 5000
#define MAX_TOKEN_LEN 64
#define MAX_SOURCE_LEN 50000

// token and table storage using parallel arrays
char tokens[MAX_TOKENS][MAX_TOKEN_LEN];
int tokenTypes[MAX_TOKENS];
int tokenValOrIndex[MAX_TOKENS];
int tokenLine[MAX_TOKENS];
int tokenCol[MAX_TOKENS];
int numTokens = 0;

// name table storage
char nameTokens[MAX_TOKENS][MAX_TOKEN_LEN];
int nameLine[MAX_TOKENS];
int nameCol[MAX_TOKENS];
int nameElements = 0;

// source buffer
char source[MAX_SOURCE_LEN];
int sourceLen = 0;
```

#### step 2: argument check and reading the file
at the start of `main`, handle argument checking, read the entire file into `source`, and print the `Source Program:` section:
```c
int main(int argc, char *argv[]) {
    // 1. check arguments
    if (argc != 2) {
        printf("Usage: ./lex <input file>\n");
        return 1;
    }

    // 2. open the input file
    FILE* ptr = fopen(argv[1], "r");
    if (ptr == NULL) {
        printf("Error: unable to open input file '%s'\n", argv[1]);
        return 1;
    }

    // 3. read the entire file byte-by-byte
    int ch;
    while ((ch = fgetc(ptr)) != EOF) {
        if (sourceLen < MAX_SOURCE_LEN - 1) {
            source[sourceLen++] = (char)ch;
        }
    }
    source[sourceLen] = '\0';
    fclose(ptr);

    // 4. print source program section
    printf("Source Program:\n\n");
    for (int i = 0; i < sourceLen; i++) {
        putchar(source[i]);
    }
    // ensure it ends on a fresh line
    if (sourceLen > 0 && source[sourceLen - 1] != '\n') {
        putchar('\n');
    }
    printf("\n");
```

#### step 3: the scanning loop (tracking lines and columns)
now scan directly from the `source` array; tracking positions is simple:
* `line` starts at 1, `col` starts at 1.
* a newline `\n` sets `line++` and `col = 1`.
* a carriage return `\r` does not advance `col` or `line`.
* all other characters advance `col++`.

```c
    int i = 0;
    int line = 1;
    int col = 1;

    while (i < sourceLen) {
        char c = source[i];

        // whitespace handling
        if (c == '\r') {
            i++;
            continue; // CR advances nothing
        }
        if (c == '\n') {
            line++;
            col = 1;
            i++;
            continue;
        }
        if (c == ' ' || c == '\t') {
            col++;
            i++;
            continue;
        }

        // comment handling: /* ... */
        if (c == '/' && i + 1 < sourceLen && source[i + 1] == '*') {
            i += 2;
            col += 2;
            int closed = 0;
            while (i < sourceLen) {
                if (source[i] == '\r') {
                    i++;
                } else if (source[i] == '\n') {
                    line++;
                    col = 1;
                    i++;
                } else if (source[i] == '*' && i + 1 < sourceLen && source[i + 1] == '/') {
                    i += 2;
                    col += 2;
                    closed = 1;
                    break;
                } else {
                    col++;
                    i++;
                }
            }
            continue;
        }

        // record starting position for this new token
        int startL = line;
        int startC = col;

        // case a: words (identifiers and reserved words)
        if (isalpha(c)) {
            int len = 0;
            while (i < sourceLen && (isalnum(source[i]))) {
                tokens[numTokens][len++] = source[i];
                i++;
                col++;
            }
            tokens[numTokens][len] = '\0';
            tokenLine[numTokens] = startL;
            tokenCol[numTokens] = startC;
            numTokens++;
            continue;
        }

        // case b: numbers
        if (isdigit(c)) {
            int len = 0;
            while (i < sourceLen && isdigit(source[i])) {
                tokens[numTokens][len++] = source[i];
                i++;
                col++;
            }
            tokens[numTokens][len] = '\0';
            tokenLine[numTokens] = startL;
            tokenCol[numTokens] = startC;
            numTokens++;
            continue;
        }

        // case c: two-character operators (==, <=, >=, !=, :=)
        if (c == '=' && i + 1 < sourceLen && source[i + 1] == '=') {
            strcpy(tokens[numTokens], "==");
            tokenLine[numTokens] = startL; tokenCol[numTokens] = startC;
            numTokens++; i += 2; col += 2; continue;
        }
        if (c == '<' && i + 1 < sourceLen && source[i + 1] == '=') {
            strcpy(tokens[numTokens], "<=");
            tokenLine[numTokens] = startL; tokenCol[numTokens] = startC;
            numTokens++; i += 2; col += 2; continue;
        }
        if (c == '>' && i + 1 < sourceLen && source[i + 1] == '=') {
            strcpy(tokens[numTokens], ">=");
            tokenLine[numTokens] = startL; tokenCol[numTokens] = startC;
            numTokens++; i += 2; col += 2; continue;
        }
        if (c == '!' && i + 1 < sourceLen && source[i + 1] == '=') {
            strcpy(tokens[numTokens], "!=");
            tokenLine[numTokens] = startL; tokenCol[numTokens] = startC;
            numTokens++; i += 2; col += 2; continue;
        }
        if (c == ':' && i + 1 < sourceLen && source[i + 1] == '=') {
            strcpy(tokens[numTokens], ":=");
            tokenLine[numTokens] = startL; tokenCol[numTokens] = startC;
            numTokens++; i += 2; col += 2; continue;
        }

        // case d: single-character symbols (+, -, *, /, <, >, (, ), ,, ;, ., =)
        tokens[numTokens][0] = c;
        tokens[numTokens][1] = '\0';
        tokenLine[numTokens] = startL;
        tokenCol[numTokens] = startC;
        numTokens++;
        col++;
        i++;
    }
```

#### step 4: token classification & building the name table
loop through `tokens[0 ... numTokens - 1]` to assign each `tokenTypes[t]` and insert unique identifiers into `nameTokens`:
```c
    for (int t = 0; t < numTokens; t++) {
        char *lex = tokens[t];
        if (strcmp(lex, "+") == 0) tokenTypes[t] = 3;
        else if (strcmp(lex, "-") == 0) tokenTypes[t] = 4;
        else if (strcmp(lex, "*") == 0) tokenTypes[t] = 5;
        else if (strcmp(lex, "/") == 0) tokenTypes[t] = 6;
        else if (strcmp(lex, "==") == 0) tokenTypes[t] = 7;
        else if (strcmp(lex, "!=") == 0) tokenTypes[t] = 8;
        else if (strcmp(lex, "<") == 0) tokenTypes[t] = 9;
        else if (strcmp(lex, "<=") == 0) tokenTypes[t] = 10;
        else if (strcmp(lex, ">") == 0) tokenTypes[t] = 11;
        else if (strcmp(lex, ">=") == 0) tokenTypes[t] = 12;
        else if (strcmp(lex, "(") == 0) tokenTypes[t] = 13;
        else if (strcmp(lex, ")") == 0) tokenTypes[t] = 14;
        else if (strcmp(lex, ",") == 0) tokenTypes[t] = 15;
        else if (strcmp(lex, ";") == 0) tokenTypes[t] = 16;
        else if (strcmp(lex, ".") == 0) tokenTypes[t] = 17;
        else if (strcmp(lex, "=") == 0) tokenTypes[t] = 18;
        else if (strcmp(lex, ":=") == 0) tokenTypes[t] = 19;
        else if (strcmp(lex, "begin") == 0) tokenTypes[t] = 20;
        else if (strcmp(lex, "end") == 0) tokenTypes[t] = 21;
        else if (strcmp(lex, "if") == 0) tokenTypes[t] = 22;
        else if (strcmp(lex, "fi") == 0) tokenTypes[t] = 23;
        else if (strcmp(lex, "then") == 0) tokenTypes[t] = 24;
        else if (strcmp(lex, "while") == 0) tokenTypes[t] = 25;
        else if (strcmp(lex, "elihw") == 0) tokenTypes[t] = 26;
        else if (strcmp(lex, "do") == 0) tokenTypes[t] = 27;
        else if (strcmp(lex, "od") == 0) tokenTypes[t] = 28;
        else if (strcmp(lex, "odd") == 0) tokenTypes[t] = 29;
        else if (strcmp(lex, "call") == 0) tokenTypes[t] = 30;
        else if (strcmp(lex, "const") == 0) tokenTypes[t] = 31;
        else if (strcmp(lex, "var") == 0) tokenTypes[t] = 32;
        else if (strcmp(lex, "procedure") == 0) tokenTypes[t] = 33;
        else if (strcmp(lex, "write") == 0) tokenTypes[t] = 34;
        else if (strcmp(lex, "read") == 0) tokenTypes[t] = 35;
        else if (strcmp(lex, "else") == 0) tokenTypes[t] = 36;
        else if (isdigit(lex[0])) {
            tokenTypes[t] = 2; // numbersym
        } else {
            // identsym (code 1)
            tokenTypes[t] = 1;
            int found = -1;
            for (int k = 0; k < nameElements; k++) {
                if (strcmp(lex, nameTokens[k]) == 0) {
                    found = k;
                    break;
                }
            }
            if (found == -1) {
                found = nameElements;
                strcpy(nameTokens[nameElements], lex);
                nameLine[nameElements] = tokenLine[t];
                nameCol[nameElements] = tokenCol[t];
                nameElements++;
            }
            tokenValOrIndex[t] = found;
        }
    }
```

#### step 5: printing tables and writing output files
put this block right before the return statement of main:
```c
    // 1. print lexeme table
    printf("Lexeme Table:\n\nlexeme\t\ttoken\n");
    for (int t = 0; t < numTokens; t++) {
        printf("%s\t\t%d\n", tokens[t], tokenTypes[t]);
    }
    printf("\n");

    // 2. print name table
    printf("Name Table:\n\nindex\tname\t\tline\tcolumn\n");
    for (int k = 0; k < nameElements; k++) {
        printf("%d\t%s\t\t%d\t%d\n", k, nameTokens[k], nameLine[k], nameCol[k]);
    }
    printf("\n");

    // 3. print token list
    printf("Token List:\n\n");
    for (int t = 0; t < numTokens; t++) {
        if (tokenTypes[t] == 1) {
            printf("%d %d ", tokenTypes[t], tokenValOrIndex[t]);
        } else if (tokenTypes[t] == 2) {
            printf("%d %s ", tokenTypes[t], tokens[t]);
        } else {
            printf("%d ", tokenTypes[t]);
        }
    }
    printf("\n");

    // 4. write tokens.txt
    FILE *f_tok = fopen("tokens.txt", "w");
    if (f_tok != NULL) {
        for (int t = 0; t < numTokens; t++) {
            if (tokenTypes[t] == 1) {
                fprintf(f_tok, "%d %d\n", tokenTypes[t], tokenValOrIndex[t]);
            } else if (tokenTypes[t] == 2) {
                fprintf(f_tok, "%d %s\n", tokenTypes[t], tokens[t]);
            } else {
                fprintf(f_tok, "%d\n", tokenTypes[t]);
            }
        }
        fclose(f_tok);
    }

    // 5. write nametable.txt
    FILE *f_name = fopen("nametable.txt", "w");
    if (f_name != NULL) {
        for (int k = 0; k < nameElements; k++) {
            fprintf(f_name, "%d %s %d %d\n", k, nameTokens[k], nameLine[k], nameCol[k]);
        }
        fclose(f_name);
    }

    return 0;
}
```

#### verification plan
1. put these sections together into your `lex.c`.
2. compile on eustis: `gcc -Wall -Wextra -std=c11 -O2 lex.c -o lex`.
3. run against minimal input: `./lex test_cases/inputs/s01_minimal.txt`.
4. inspect `tokens.txt` and `nametable.txt` to verify they exist and match the printed output.

### What I did with it
I reviewed the file reading, token storage, and table formatting connect before adapting my own loop.

---

## Round 4

### Prompt
okay that makes sense tysm. i can see the implementation but i want to adjust my current code so that its not just rewrite and copy since i want to do it in my solution too can we look at how to modify mine?

### AI Output
here is how we adapt your existing file step-by-step without rewriting it from scratch.

#### 1. buffer sizes and line/column initialization
in your variable declarations:
```c
// change 10 to 64 so names and words do not overflow
int maxTokenLen = 64;
int maxTokens = 5000;

// start line and column at 1, not 0
int line = 1;
int column = 1;
int startLine = 1;
int startCol = 1;

// parallel arrays to store where each token starts
int tokenLine[maxTokens];
int tokenCol[maxTokens];

// arrays to store line and column for unique names in the name table
int nameLine[maxTokens];
int nameCol[maxTokens];
```

#### 2. updating line/column tracking inside the read loop
in your `while (fscanf(ptr, "%c", &readChar) != EOF)` loop:
replace the top position block with:
```c
printf("%c", readChar);

// carriage return advances nothing
if (readChar == '\r') {
    continue;
}

// newline increments line and resets column to 1
if (readChar == '\n') {
    line++;
    column = 1;
    // if a word was being built, end it here
    if (index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }
    continue;
}
```

whenever a character starts a new alphanumeric word (`index == 0`), record its coordinates:
```c
// check if char is an alphanumeric char
if (((readChar >= 'a') && (readChar <= 'z')) ||
    ((readChar >= 'A') && (readChar <= 'Z')) ||
    ((readChar >= '0') && (readChar <= '9'))) {
    if (index == 0) {
        startLine = line;
        startCol = column;
    }
    temp[index] = readChar;
    index++;
    column++;
}
```

when handling whitespace (`readChar == ' ' || readChar == '\t'`):
```c
else if (readChar == ' ' || readChar == '\t') {
    if (index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }
    column++;
}
```

when handling operators in `isSingleKeyWord`:
* store the word currently in `temp` (if any), saving its `startLine` and `startCol`.
* capture `opLine = line` and `opCol = column` before lookahead peeking.
* save `opLine` and `opCol` into `tokenLine[numTokens]` and `tokenCol[numTokens]` for the operator token.
* advance `column++` (or `column += 2` if two characters were read).

when pushing the final trailing word after the while loop finishes:
```c
if (index > 0) {
    temp[index] = '\0';
    strcpy(tokens[numTokens], temp);
    tokenLine[numTokens] = startLine;
    tokenCol[numTokens] = startCol;
    numTokens++;
    index = 0;
}
```

#### 3. fixing the classification and name table loop
replace your lookup loop with this version that preserves your if-else matching while storing identifier indices properly:
```c
int tokenTypes[maxTokens];
int tokenValOrIndex[maxTokens];

for (int i = 0; i < numTokens; i++) {
    strcpy(lexT, tokens[i]);
    if(strcmp(lexT, "+") == 0) code = 3;
    else if(strcmp(lexT, "-") == 0) code = 4;
    else if(strcmp(lexT, "*") == 0) code = 5;
    else if(strcmp(lexT, "/") == 0) code = 6;
    else if(strcmp(lexT, "==") == 0) code = 7;
    else if(strcmp(lexT, "!=") == 0) code = 8;
    else if(strcmp(lexT, "<") == 0) code = 9;
    else if(strcmp(lexT, "<=") == 0) code = 10;
    else if(strcmp(lexT, ">") == 0) code = 11;
    else if(strcmp(lexT, ">=") == 0) code = 12;
    else if(strcmp(lexT, "(") == 0) code = 13;
    else if(strcmp(lexT, ")") == 0) code = 14;
    else if(strcmp(lexT, ",") == 0) code = 15;
    else if(strcmp(lexT, ";") == 0) code = 16;
    else if(strcmp(lexT, ".") == 0) code = 17;
    else if(strcmp(lexT, "=") == 0) code = 18;
    else if(strcmp(lexT, ":=") == 0) code = 19;
    else if(strcmp(lexT, "begin") == 0) code = 20;
    else if(strcmp(lexT, "end") == 0) code = 21;
    else if(strcmp(lexT, "if") == 0) code = 22;
    else if(strcmp(lexT, "fi") == 0) code = 23;
    else if(strcmp(lexT, "then") == 0) code = 24;
    else if(strcmp(lexT, "while") == 0) code = 25;
    else if(strcmp(lexT, "elihw") == 0) code = 26;
    else if(strcmp(lexT, "do") == 0) code = 27;
    else if(strcmp(lexT, "od") == 0) code = 28;
    else if(strcmp(lexT, "odd") == 0) code = 29;
    else if(strcmp(lexT, "call") == 0) code = 30;
    else if(strcmp(lexT, "const") == 0) code = 31;
    else if(strcmp(lexT, "var") == 0) code = 32;
    else if(strcmp(lexT, "procedure") == 0) code = 33;
    else if(strcmp(lexT, "write") == 0) code = 34;
    else if(strcmp(lexT, "read") == 0) code = 35;
    else if(strcmp(lexT, "else") == 0) code = 36;
    else if(isOnlyDigits(lexT)) {
        code = 2;
    }
    else {
        code = 1;
        int found = -1;
        for (int k = 0; k < nameElements; k++) {
            if (strcmp(lexT, nameTokens[k]) == 0) {
                found = k;
                break;
            }
        }
        if (found == -1) {
            found = nameElements;
            strcpy(nameTokens[nameElements], lexT);
            nameLine[nameElements] = tokenLine[i];
            nameCol[nameElements] = tokenCol[i];
            nameElements++;
        }
        tokenValOrIndex[i] = found;
    }
    tokenTypes[i] = code;
}
```

#### 4. printing tables and token list to stdout
replace your printing loops with this:
```c
printf("\nLexeme Table:\n\nlexeme\t\ttoken\n");
for (int i = 0; i < numTokens; i++) {
    printf("%s\t\t%d\n", tokens[i], tokenTypes[i]);
}

printf("\nName Table:\n\nindex\tname\t\tline\tcolumn\n");
for (int i = 0; i < nameElements; i++) {
    printf("%d\t%s\t\t%d\t%d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
}

printf("\nToken List:\n\n");
for (int i = 0; i < numTokens; i++) {
    if (tokenTypes[i] == 1) {
        printf("%d %d ", tokenTypes[i], tokenValOrIndex[i]);
    } else if (tokenTypes[i] == 2) {
        printf("%d %s ", tokenTypes[i], tokens[i]);
    } else {
        printf("%d ", tokenTypes[i]);
    }
}
printf("\n");
```

#### 5. writing the two output files
at the end of `main`, write `tokens.txt` and `nametable.txt`:
```c
FILE* f_tok = fopen("tokens.txt", "w");
if (f_tok != NULL) {
    for (int i = 0; i < numTokens; i++) {
        if (tokenTypes[i] == 1) {
            fprintf(f_tok, "%d %d\n", tokenTypes[i], tokenValOrIndex[i]);
        } else if (tokenTypes[i] == 2) {
            fprintf(f_tok, "%d %s\n", tokenTypes[i], tokens[i]);
        } else {
            fprintf(f_tok, "%d\n", tokenTypes[i]);
        }
    }
    fclose(f_tok);
}

FILE* f_name = fopen("nametable.txt", "w");
if (f_name != NULL) {
    for (int i = 0; i < nameElements; i++) {
        fprintf(f_name, "%d %s %d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
    }
    fclose(f_name);
}

return 0;
```

#### execution checklist
1. replace line 47 (`int line = 0; int column = 0;`) with `line = 1; column = 1;` and bump array capacities.
2. insert `tokenLine` and `tokenCol` tracking inside each branch of your `fscanf` loop where `strcpy(tokens[numTokens], ...)` is called.
3. update the lookup loop to populate `tokenValOrIndex` for identifiers.
4. update the printing section and append the file writers for `tokens.txt` and `nametable.txt`.

### What I did with it
I changed my buffer sizes to 64, set the initial line/column counters to 1, and updated into my existing `fscanf` loop.

---

## Round 5

### Prompt
okay i see it makes sense tysm. i was able to implement it into my own code going off of your blueprint ty again. for the tokenType[] array tho, is that the same as my tokenCodes[] array bc i used that one instead to put and update the codes

### AI Output
yes, `tokenTypes[]` serves the exact same role as `tokenCode[]`, with one important distinction in how values are inserted into it:
* in your original code, you used `tokenCode[]` as an output stream where you pushed two numbers for an identifier: you pushed the index into `tokenCode[numCodes]`, and then pushed the code 1 into `tokenCode[numCodes]` right after it.
* in the updated design, `tokenTypes[i]` holds only the token category code (e.g. 1 for identifiers, 2 for numbers, 3 to 36 for symbols and reserved words) for the token at `tokens[i]`.
* the identifier's name-table index is tracked alongside it in `tokenValOrIndex[i]`, so the indices align 1:1 with `tokens[i]`.

if you prefer to keep the name `tokenCode[]`, you can use it instead of `tokenTypes[]` as long as each `tokenCode[i]` holds only the single category code for `tokens[i]`.

### What I did with it
I kept the name `tokenCode[]` for category codes while using a parallel array `tokenIndex[]` to store the indices so elements mapped with `tokens[i]`.

---

## Round 6

### Prompt
for my code here:
```c
#include<stdio.h>
#include<string.h>

//checks and returns if word is a single reserved word
int isSingleKeyWord(char character, char single[], int numSingle) {
    for(int i = 0; i < numSingle; i++) {
        if(character == single[i]) {
            return 1;
        }
    }
    return 0;
}

//checks if a string is only digits
int isOnlyDigits(char lexT[]) {
    //if string is empty, not only digits
    if(lexT[0] == '\0') return 0;
    int k = 0;
    while(lexT[k] != '\0') {
        if(!((lexT[k] >= '0') && (lexT[k] <= '9'))) return 0;
        k++;
    }
    return 1;
}

int main(int argc, char *argv[]) {
    //check if argc is valid
    if(argc != 2) {
        printf("Usage: ./lex <input file>\n");
        return 1;
    }
    //open the input file
    FILE *ptr = fopen(argv[1], "r");
    //check if input file is valid
    if(ptr == NULL) {
        printf("Error: unable to open input file '%s'\n", argv[1]);
        return 1;
    }
    //variables to track line, column, and text
    int line = 1;
    int column = 1;
    int startLine = 1;
    int startCol = 1;
    int maxTokenLen = 64;
    int maxTokens = 5000;
    //array to store name tokens
    char nameTokens[maxTokens][maxTokenLen];
    int nameElements = 0;
    int nameLine[maxTokens];
    int nameCol[maxTokens];
    //token array for symbols (excludes letter and digit case: codes 1, 2)
    char singleKeys[] = {'+', '-', '*', '/', '(', ')', '=', ',', '.', '<', '>', ';', ':', '!'};
    int numSingle = 14;
    //2D array to store input text and parse
    char readChar;
    char rTemp;
    //token arrays to store general tokens
    char tokens[maxTokens][maxTokenLen];
    int numTokens = 0;
    //track the indices and type of tokens
    int tokenCode[maxTokens];
    char tokenIndex[maxTokens];
    //track the token line and columns
    char tokenLine[maxTokens];
    char tokenCol[maxTokens];
    int index = 0;
    char temp[maxTokenLen];
    //output
    printf("\nSource Program:\n\n");
    //scan through and print the input file
    while(fscanf(ptr, "%c", &readChar) != EOF) {
        printf("%c", readChar);
        //check carriage return and new line chars
        if(readChar == '\r') continue;
        if(readChar == '\n') {
            line++;
            column = 1;
            //end word if it falls at a new line
            if(index > 0) {
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            continue;
        }
        //check if char is an alphanumeric char
        if(((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')) || ((readChar >= '0') && (readChar <= '9'))) {
            //set line and column tracker for new word
            if(index == 0) {
                startLine = line;
                startCol = column;
            }
            temp[index] = readChar;
            index++;
            column++;
        }
        //check if char is a white space and add if new word is being scanned
        else if(((readChar == ' ') || (readChar == '\n') || (readChar == '\t') || (readChar == '\r')) && (index > 0)) {
            if(index > 0) {
                //end building the current word
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                //add the line and column track values for the current token
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            column++; //update column
        }
        //each if char is a reserved single token
        else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
            //add current word to token array
            if(index > 0) {
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            //check if next char makes a longer operator
            if((readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
                fscanf(ptr, "%c", &rTemp);
                //check if next operator is '='
                if(rTemp == '=') {
                    printf("%c", rTemp);
                    column++;
                    //add long operator to token array
                    temp[0] = readChar;
                    temp[1] = '=';
                    temp[2] = '\0';
                    strcpy(tokens[numTokens], temp);
                    numTokens++;
                }
                //store reserved char into array as well
                else {
                    //check if rTemp was not a '='
                    if(rTemp != '=') {
                        //put scanned char back into input stream
                        ungetc(rTemp, ptr);
                    }
                    //store a single char
                    temp[0] = readChar;
                    temp[1] = '\0';
                    strcpy(tokens[numTokens], temp);
                    numTokens++;
                }
            }
            //store a single char
            else {
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                numTokens++;
            }
        }
    }
    //add last word to token array once scanning finishes
    if(index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        numTokens++;
        index = 0;
    }
    printf("\n");
    char lexT[maxTokenLen];
    int code = 0;
    //loop through token array and print token and code
    for(int i = 0; i < numTokens; i++) {
        strcpy(lexT, tokens[i]);
        if(strcmp(lexT, "+") == 0) code = 3;
        else if(strcmp(lexT, "-") == 0) code = 4;
        else if(strcmp(lexT, "*") == 0) code = 5;
        else if(strcmp(lexT, "/") == 0) code = 6;
        else if(strcmp(lexT, "==") == 0) code = 7;
        else if(strcmp(lexT, "!=") == 0) code = 8;
        else if(strcmp(lexT, "<") == 0) code = 9;
        else if(strcmp(lexT, "<=") == 0) code = 10;
        else if(strcmp(lexT, ">") == 0) code = 11;
        else if(strcmp(lexT, ">=") == 0) code = 12;
        else if(strcmp(lexT, "(") == 0) code = 13;
        else if(strcmp(lexT, ")") == 0) code = 14;
        else if(strcmp(lexT, ",") == 0) code = 15;
        else if(strcmp(lexT, ";") == 0) code = 16;
        else if(strcmp(lexT, ".") == 0) code = 17;
        else if(strcmp(lexT, "=") == 0) code = 18;
        else if(strcmp(lexT, ":=") == 0) code = 19;
        else if(strcmp(lexT, "begin") == 0) code = 20;
        else if(strcmp(lexT, "end") == 0) code = 21;
        else if(strcmp(lexT, "if") == 0) code = 22;
        else if(strcmp(lexT, "fi") == 0) code = 23;
        else if(strcmp(lexT, "then") == 0) code = 24;
        else if(strcmp(lexT, "while") == 0) code = 25;
        else if(strcmp(lexT, "elihw") == 0) code = 26;
        else if(strcmp(lexT, "do") == 0) code = 27;
        else if(strcmp(lexT, "od") == 0) code = 28;
        else if(strcmp(lexT, "odd") == 0) code = 29;
        else if(strcmp(lexT, "call") == 0) code = 30;
        else if(strcmp(lexT, "const") == 0) code = 31;
        else if(strcmp(lexT, "var") == 0) code = 32;
        else if(strcmp(lexT, "procedure") == 0) code = 33;
        else if(strcmp(lexT, "write") == 0) code = 34;
        else if(strcmp(lexT, "read") == 0) code = 35;
        else if(strcmp(lexT, "else") == 0) code = 36;
        else if(isOnlyDigits(lexT)) code = 2;
        else {
            code = 1;
            int foundAtIndex = -1;
            //check if lexT is in name token array already
            for(int k = 0; k < nameElements; k++) {
                if(strcmp(lexT, nameTokens[k]) == 0) {
                    //get the index the word was found inside the name array
                    foundAtIndex = k;
                    break;
                }
            }
            //if token not inside name array, store and update values
            if(foundAtIndex == -1) {
                foundAtIndex = nameElements;
                strcpy(nameTokens[nameElements], lexT);
                //update line and col name array from the values in token line and col
                nameLine[nameElements] = tokenLine[i];
                nameCol[nameElements] = tokenCol[i];
                nameElements++;
            }
            tokenIndex[i] = foundAtIndex;
        }
        tokenCode[i] = code;
    }
    printf("\nLexeme Table:\n\n");
    printf("lexeme\t\ttoken\n");
    //print out the token and
    for(int i = 0; i < numTokens; i++) {
        printf("%s\t\t%d\n", tokens[i], tokenCode[i]);
    }
    printf("\n");
    printf("\nName Table:\n\n");
    printf("index name\t\tline column\n");
    for(int i = 0; i < nameElements; i++) {
        printf("%d %s\t\t\t%d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
    }
    printf("\n");
    printf("\nToken List:\n\n");
    //print out the token codes in tokenCode[]
    for(int i = 0; i < numTokens; i++) {
        if(tokenCode[i] == 1) {
            //print back to back integers stored for token
            printf("%d %d ", tokenCode[i], tokenIndex[i]);
        }
        else if(tokenCode[i] == 2) {
            //print code and word
            printf("%d %s ", tokenCode[i], tokens[i]);
        } else {
            //print only the token word
            printf("%d ", tokenCode[i]);
        }
    }
    printf("\n");
    //write to file tokens.txt
    FILE *tokenFile = fopen("tokens.txt", "w");
    if(tokenFile != NULL) {
        //check token code and write to file like from printing
        for(int i = 0; i < numTokens; i++) {
            if(tokenCode[i] == 1) {
                //print back to back integers stored for token
                fprintf(tokenFile, "%d %d\n", tokenCode[i], tokenIndex[i]);
            } else if(tokenCode[i] == 2) {
                //print code and word
                fprintf(tokenFile, "%d %s\n", tokenCode[i], tokens[i]);
            } else {
                //print only the token word
                fprintf(tokenFile, "%d\n", tokenCode[i]);
            }
        }
        fclose(tokenFile);
    }
    //write to file nametable.txt
    FILE *nameFile = fopen("nametable.txt", "w");
    if(nameFile != NULL) {
        //loop and print name token array contents with line and columns
        for(int i = 0; i < nameElements; i++) {
            fprintf(nameFile, "%d %s %d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
        }
        fclose(nameFile);
    }
    return 0;
}
```
i do fine for the following test case: `const limit := 10; var n, total; begin n := 1; total := 0; while n < limit then total := total + n; n := n + 1 elihw; write total end.` except the last column value printed is 6 instead of 8, which is throwing off my other cases as well

### AI Output
the column being 6 instead of 8 for `total` is caused by where `column++` happens when operators are scanned.

#### where the column count slips
on line 2:
```pascal
var n, total;
```
`total` starts at column 8. your scanner reported column 6 because of this branch in your operator logic:
```c
else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
    // 1. push previous word (n)
    if (index > 0) { ... }

    // 2. check lookahead for two-char operator
    if ((readChar == '>') || (readChar == '<') || ...) {
    }
    // 3. store single char
    else {
        temp[0] = readChar;
        temp[1] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = line;
        tokenCol[numTokens] = column;
        numTokens++;
    }
}
```
notice what is missing: `column++` is never executed for single-character symbols like `,`, `;`, `+`, or `(`.
* when `,` is read, `column` remains unchanged.
* when the space after `,` is read, your whitespace branch does `column++`.
* because `,` did not advance `column`, every column position after `,` on that line is too small by 1.

in addition, looking at `tokenLine` and `tokenCol`, you have:
```c
char tokenLine[maxTokens];
char tokenCol[maxTokens];
char tokenIndex[maxTokens];
```
`char` can overflow if a line or column exceeds 127. changing those to `int` prevents silent wrapping on longer files.

also check your lookahead branch: when `rTemp != '='`, you `ungetc(rTemp, ptr)`, but neither `readChar`'s line/col is saved to `tokenLine`/`tokenCol` nor is `column++` counted for the single character.

#### the fix
capture the operator's position, push it, and always advance column by how many characters the operator actually consumed:
```c
// each if char is a reserved single token
else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
    // add current word to token array
    if (index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }

    // capture the starting position of this operator
    int opLine = line;
    int opCol = column;

    // check if next char makes a longer operator
    if((readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
        if (fscanf(ptr, "%c", &rTemp) == 1) {
            if(rTemp == '=') {
                printf("%c", rTemp);
                temp[0] = readChar;
                temp[1] = '=';
                temp[2] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = opLine;
                tokenCol[numTokens] = opCol;
                numTokens++;
                column += 2; // two characters consumed
            }
            else {
                ungetc(rTemp, ptr);
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = opLine;
                tokenCol[numTokens] = opCol;
                numTokens++;
                column++; // only readChar consumed
            }
        }
        else {
            // EOF right after lookahead character
            temp[0] = readChar;
            temp[1] = '\0';
            strcpy(tokens[numTokens], temp);
            tokenLine[numTokens] = opLine;
            tokenCol[numTokens] = opCol;
            numTokens++;
            column++;
        }
    }
    // single-character operator (+, -, *, /, ,, ;, ., etc.)
    else {
        temp[0] = readChar;
        temp[1] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = opLine;
        tokenCol[numTokens] = opCol;
        numTokens++;
        column++; // advance for the single symbol
    }
}
```

#### verification
1. change `tokenLine`, `tokenCol`, and `tokenIndex` from `char` arrays to `int` arrays.
2. replace the `isSingleKeyWord` block with the version above so every single symbol adds 1 to column and saves its `opLine` and `opCol`.
3. recompile and rerun on your test input: `total` on line 2 will now register at line 2, column 8.

### What I did with it
I changed `tokenLine`, `tokenCol`, and `tokenIndex` from `char` to `int` to prevent overflow, and added `column++` to adjust column changes.

---

## Round 7

### Prompt
i went ahead and updated those parts but it only still prints 7 instead:
```c
#include<stdio.h>
#include<string.h>

//checks and returns if word is a single reserved word
int isSingleKeyWord(char character, char single[], int numSingle) {
    for(int i = 0; i < numSingle; i++) {
        if(character == single[i]) {
            return 1;
        }
    }
    return 0;
}

//checks if a string is only digits
int isOnlyDigits(char lexT[]) {
    //if string is empty, not only digits
    if(lexT[0] == '\0') return 0;
    int k = 0;
    while(lexT[k] != '\0') {
        if(!((lexT[k] >= '0') && (lexT[k] <= '9'))) return 0;
        k++;
    }
    return 1;
}

int main(int argc, char *argv[]) {
    //check if argc is valid
    if(argc != 2) {
        printf("Usage: ./lex <input file>\n");
        return 1;
    }
    //open the input file
    FILE *ptr = fopen(argv[1], "r");
    //check if input file is valid
    if(ptr == NULL) {
        printf("Error: unable to open input file '%s'\n", argv[1]);
        return 1;
    }
    //variables to track line, column, and text
    int line = 1;
    int column = 1;
    int startLine = 1;
    int startCol = 1;
    int maxTokenLen = 64;
    int maxTokens = 5000;
    //array to store name tokens
    char nameTokens[maxTokens][maxTokenLen];
    int nameElements = 0;
    int nameLine[maxTokens];
    int nameCol[maxTokens];
    //token array for symbols (excludes letter and digit case: codes 1, 2)
    char singleKeys[] = {'+', '-', '*', '/', '(', ')', '=', ',', '.', '<', '>', ';', ':', '!'};
    int numSingle = 14;
    //2D array to store input text and parse
    char readChar;
    char rTemp;
    //token arrays to store general tokens
    char tokens[maxTokens][maxTokenLen];
    int numTokens = 0;
    //track the indices and type of tokens
    int tokenCode[maxTokens];
    int tokenIndex[maxTokens];
    //track the token line and columns
    int tokenLine[maxTokens];
    int tokenCol[maxTokens];
    int index = 0;
    char temp[maxTokenLen];
    //output
    printf("\nSource Program:\n\n");
    //scan through and print the input file
    while(fscanf(ptr, "%c", &readChar) != EOF) {
        printf("%c", readChar);
        //check carriage return and new line chars
        if(readChar == '\r') continue;
        if(readChar == '\n') {
            line++;
            column = 1;
            //end word if it falls at a new line
            if(index > 0) {
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            continue;
        }
        //check if char is an alphanumeric char
        if(((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')) || ((readChar >= '0') && (readChar <= '9'))) {
            //set line and column tracker for new word
            if(index == 0) {
                startLine = line;
                startCol = column;
            }
            temp[index] = readChar;
            index++;
            column++;
        }
        //check if char is a white space and add if new word is being scanned
        else if(((readChar == ' ') || (readChar == '\n') || (readChar == '\t') || (readChar == '\r')) && (index > 0)) {
            if(index > 0) {
                //end building the current word
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                //add the line and column track values for the current token
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            column++; //update column
        }
        //each if char is a reserved single token
        else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
            //add current word to token array
            if(index > 0) {
                temp[index] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            //track the line and column for long operators
            int lineOp = line;
            int colOp = column;
            //check if next char makes a longer operator
            if((readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
                if(fscanf(ptr, "%c", &rTemp) == 1) {
                    //check if next operator is '='
                    if(rTemp == '=') {
                        printf("%c", rTemp);
                        column++;
                        //add long operator to token array
                        temp[0] = readChar;
                        temp[1] = '=';
                        temp[2] = '\0';
                        strcpy(tokens[numTokens], temp);
                        numTokens++;
                        column += 2; //long operators take up 2 chars
                    }
                    //store reserved char into array as well if next char isnt '='
                    else {
                        //put scanned char back into input stream
                        ungetc(rTemp, ptr);
                        //store a single char
                        temp[0] = readChar;
                        temp[1] = '\0';
                        strcpy(tokens[numTokens], temp);
                        tokenLine[numTokens] = lineOp;
                        tokenLine[numTokens] = colOp;
                        numTokens++;
                        column++;
                    }
                }
                //store at EOF
                else {
                    temp[0] = readChar;
                    temp[1] = '\0';
                    strcpy(tokens[numTokens], temp);
                    tokenLine[numTokens] = lineOp;
                    tokenLine[numTokens] = colOp;
                    numTokens++;
                    column++;
                }
            }
            //store a single char
            else {
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = lineOp;
                tokenLine[numTokens] = colOp;
                numTokens++;
                column++;
            }
        }
    }
    //add last word to token array once scanning finishes
    if(index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        numTokens++;
        index = 0;
    }
    printf("\n");
    char lexT[maxTokenLen];
    int code = 0;
    //loop through token array and print token and code
    for(int i = 0; i < numTokens; i++) {
        strcpy(lexT, tokens[i]);
        if(strcmp(lexT, "+") == 0) code = 3;
        else if(strcmp(lexT, "-") == 0) code = 4;
        else if(strcmp(lexT, "*") == 0) code = 5;
        else if(strcmp(lexT, "/") == 0) code = 6;
        else if(strcmp(lexT, "==") == 0) code = 7;
        else if(strcmp(lexT, "!=") == 0) code = 8;
        else if(strcmp(lexT, "<") == 0) code = 9;
        else if(strcmp(lexT, "<=") == 0) code = 10;
        else if(strcmp(lexT, ">") == 0) code = 11;
        else if(strcmp(lexT, ">=") == 0) code = 12;
        else if(strcmp(lexT, "(") == 0) code = 13;
        else if(strcmp(lexT, ")") == 0) code = 14;
        else if(strcmp(lexT, ",") == 0) code = 15;
        else if(strcmp(lexT, ";") == 0) code = 16;
        else if(strcmp(lexT, ".") == 0) code = 17;
        else if(strcmp(lexT, "=") == 0) code = 18;
        else if(strcmp(lexT, ":=") == 0) code = 19;
        else if(strcmp(lexT, "begin") == 0) code = 20;
        else if(strcmp(lexT, "end") == 0) code = 21;
        else if(strcmp(lexT, "if") == 0) code = 22;
        else if(strcmp(lexT, "fi") == 0) code = 23;
        else if(strcmp(lexT, "then") == 0) code = 24;
        else if(strcmp(lexT, "while") == 0) code = 25;
        else if(strcmp(lexT, "elihw") == 0) code = 26;
        else if(strcmp(lexT, "do") == 0) code = 27;
        else if(strcmp(lexT, "od") == 0) code = 28;
        else if(strcmp(lexT, "odd") == 0) code = 29;
        else if(strcmp(lexT, "call") == 0) code = 30;
        else if(strcmp(lexT, "const") == 0) code = 31;
        else if(strcmp(lexT, "var") == 0) code = 32;
        else if(strcmp(lexT, "procedure") == 0) code = 33;
        else if(strcmp(lexT, "write") == 0) code = 34;
        else if(strcmp(lexT, "read") == 0) code = 35;
        else if(strcmp(lexT, "else") == 0) code = 36;
        else if(isOnlyDigits(lexT)) code = 2;
        else {
            code = 1;
            int foundAtIndex = -1;
            //check if lexT is in name token array already
            for(int k = 0; k < nameElements; k++) {
                if(strcmp(lexT, nameTokens[k]) == 0) {
                    //get the index the word was found inside the name array
                    foundAtIndex = k;
                    break;
                }
            }
            //if token not inside name array, store and update values
            if(foundAtIndex == -1) {
                foundAtIndex = nameElements;
                strcpy(nameTokens[nameElements], lexT);
                //update line and col name array from the values in token line and col
                nameLine[nameElements] = tokenLine[i];
                nameCol[nameElements] = tokenCol[i];
                nameElements++;
            }
            tokenIndex[i] = foundAtIndex;
        }
        tokenCode[i] = code;
    }
    printf("\nLexeme Table:\n\n");
    printf("lexeme\t\ttoken\n");
    //print out the token and
    for(int i = 0; i < numTokens; i++) {
        printf("%s\t\t%d\n", tokens[i], tokenCode[i]);
    }
    printf("\n");
    printf("\nName Table:\n\n");
    printf("index name\t\tline column\n");
    for(int i = 0; i < nameElements; i++) {
        printf("%d %s\t\t\t%d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
    }
    printf("\n");
    printf("\nToken List:\n\n");
    //print out the token codes in tokenCode[]
    for(int i = 0; i < numTokens; i++) {
        if(tokenCode[i] == 1) {
            //print back to back integers stored for token
            printf("%d %d ", tokenCode[i], tokenIndex[i]);
        }
        else if(tokenCode[i] == 2) {
            //print code and word
            printf("%d %s ", tokenCode[i], tokens[i]);
        } else {
            //print only the token word
            printf("%d ", tokenCode[i]);
        }
    }
    printf("\n");
    //write to file tokens.txt
    FILE *tokenFile = fopen("tokens.txt", "w");
    if(tokenFile != NULL) {
        //check token code and write to file like from printing
        for(int i = 0; i < numTokens; i++) {
            if(tokenCode[i] == 1) {
                //print back to back integers stored for token
                fprintf(tokenFile, "%d %d\n", tokenCode[i], tokenIndex[i]);
            } else if(tokenCode[i] == 2) {
                //print code and word
                fprintf(tokenFile, "%d %s\n", tokenCode[i], tokens[i]);
            } else {
                //print only the token word
                fprintf(tokenFile, "%d\n", tokenCode[i]);
            }
        }
        fclose(tokenFile);
    }
    //write to file nametable.txt
    FILE *nameFile = fopen("nametable.txt", "w");
    if(nameFile != NULL) {
        //loop and print name token array contents with line and columns
        for(int i = 0; i < nameElements; i++) {
            fprintf(nameFile, "%d %s %d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
        }
        fclose(nameFile);
    }
    return 0;
}
```

### AI Output
the reason `total` is still landing on column 7 instead of 8 is due to two bugs in your updated code:

#### bug 1: the whitespace branch advances column only if a word was being scanned
look at lines 145-158:
```c
else if(((readChar == ' ') || (readChar == '\n') || (readChar == '\t') || (readChar == '\r')) && (index > 0)) {
    if (index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }
    column++; //update column
}
```
on line 2:
```pascal
var n, total;
```
1. `var` is scanned (`index` reset to 0).
2. the space after `var` hits this branch with `index == 0`, so the `else if` condition is **false**!
3. `,` is scanned (`index = 0`).
4. the space after `,` also hits this branch with `index == 0` so it is skipped entirely and `column++` is never called for that space!

because that space is ignored by all if-else branches, `column` does not increment for it, leaving `column` 1 short (7 instead of 8).

#### bug 2: accidental double-counting and typo in the operator branch
1. in your two-character operator branch (`rTemp == '='`):
```c
if(rTemp == '=') {
    printf("%c", rTemp);
    column++;        // here (+1)
    ...
    column += 2;     // and here (+2)! total +3 instead of +2
}
```
remove the extra `column++;`.

2. when assigning `colOp`, notice you typed `tokenLine` twice in the single-character and eof branches:
```c
tokenLine[numTokens] = lineOp;
tokenLine[numTokens] = colOp; // this overwrote tokenLine instead of tokenCol!
```
it needs to be:
```c
tokenLine[numTokens] = lineOp;
tokenCol[numTokens] = colOp;
```

#### how to update both branches
replace your whitespace and single-keyword blocks with the following:
```c
// check if char is a white space
else if ((readChar == ' ') || (readChar == '\t')) {
    if(index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }
    column++; // always advance column for spaces and tabs!
}

// each if char is a reserved single token
else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
    // add current word to token array
    if(index > 0) {
        temp[index] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = startLine;
        tokenCol[numTokens] = startCol;
        numTokens++;
        index = 0;
    }

    // track the line and column for operators
    int lineOp = line;
    int colOp = column;

    // check if next char makes a longer operator
    if((readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
        if (fscanf(ptr, "%c", &rTemp) == 1) {
            // check if next operator is '='
            if(rTemp == '=') {
                printf("%c", rTemp);
                // add long operator to token array
                temp[0] = readChar;
                temp[1] = '=';
                temp[2] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = lineOp;
                tokenCol[numTokens] = colOp;
                numTokens++;
                column += 2; // long operators take 2 chars
            }
            // store reserved char into array as well if next char isnt '='
            else {
                ungetc(rTemp, ptr);
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = lineOp;
                tokenCol[numTokens] = colOp;
                numTokens++;
                column++; // single char consumed
            }
        }
        // store at EOF
        else {
            temp[0] = readChar;
            temp[1] = '\0';
            strcpy(tokens[numTokens], temp);
            tokenLine[numTokens] = lineOp;
            tokenCol[numTokens] = colOp;
            numTokens++;
            column++;
        }
    }
    // store a single char (+, -, *, /, ,, ;, etc.)
    else {
        temp[0] = readChar;
        temp[1] = '\0';
        strcpy(tokens[numTokens], temp);
        tokenLine[numTokens] = lineOp;
        tokenCol[numTokens] = colOp;
        numTokens++;
        column++;
    }
}
```

#### verification
recompile and run your test case:
* `var` starts at line 2, column 1
* `n` starts at line 2, column 5
* `,` starts at line 2, column 6
* space advances column to 7
* `total` starts at line 2, column 8

### What I did with it
I removed the `index > 0` for whitespace so spaces always advance `column` and corrected the double increment for the long operator condition.

---

## Round 8

### Prompt
that makes since i see. im working on the error cases rn. for error 6, does it count if i have a lexeme such as "ab23c"?

### AI Output
no, `ab23c` is not error 6.

#### why ab23c is valid
* an identifier begins with a letter and is followed by any combination of letters and digits up to twelve characters.
* `ab23c` begins with a letter (`a`).
* its total length is 5 characters (within the 12-character limit).
* therefore, `ab23c` is recognized as a valid identifier (`identsym`, token code 1).

#### what triggers error 6
error 6 specifically occurs when a token begins with a digit run that is immediately followed by a letter without whitespace between them (for example, `2end`, `123abc`, or `0x`).
* **message format:** `Error 6 at line <L>, column <C>: number followed by a letter '<lexeme>'`
* **reporting rule:** report the entire alphanumeric run as the offending lexeme (e.g., `'123abc'`).
* **precedence rule:** if a number is both longer than 6 digits and followed by a letter (like `1234567abc`), report error 6 instead of error 3.

#### implementation rule
* if `isalpha(first_char)`: read all following letters and digits. if length > 12, report error 2. otherwise, classify as a keyword or identifier.
* if `isdigit(first_char)`: read the entire alphanumeric run. if any character in that run is a letter, report error 6. if it contains only digits and length > 6, report error 3. otherwise, classify as a valid number.

### What I did with it
I clarified if identifiers can contain digits as long as they start with a letter, and asked about the rule that Error 6 only applies when a token starts with digits and has an ending letter.

---

## Round 9

### Prompt
does that mean 12a45 is an error?

### AI Output
yes, `12a45` is error 6.

#### why 12a45 triggers error 6
* the token begins with digits (`12`), but before any whitespace or operator separates it, an alphabetic character (`a`) appears.
* in pl/0, a number can only be followed by whitespace, punctuation, or an operator.
* as soon as a letter directly follows a digit run, the entire alphanumeric run is considered an illegal token.

#### required error reporting
when your scanner sees `12a45`, it must:
1. collect the entire continuous run of alphanumeric characters as the lexeme: `'12a45'`.
2. print all valid sections scanned up to that point (`Source Program:`, `Lexeme Table:`, `Name Table:`, `Token List:`).
3. write `tokens.txt` and `nametable.txt` containing whatever was scanned prior to this token.
4. print the exact error message at the line and column where the `1` started:
   `Error 6 at line <L>, column <C>: number followed by a letter '12a45'`
5. immediately exit with a non-zero status (such as `exit(1);` or returning 1) without continuing to scan.

### What I did with it
I asked and confirmed about sequences starting with digits that contains a letter between trigger Error 6 with the entire run reported with the lexeme.

---

## Round 10

### Prompt
for the errors, which variables do i use to report the line and column the error was reported at

### AI Output
which variables you use depends on the type of error, because the line and column must always point to where the offending token or delimiter began.

#### 1. word and number errors (errors 2, 3, and 6)
* **variables to use:** `startLine` and `startCol`.
* **why:** for errors over an entire run of characters (such as an identifier longer than 12 characters, a number longer than 6 digits, or a number followed by a letter), report the position of the first character of that run.
* **example message:** `Error 2 at line <startLine>, column <startCol>: identifier too long '<lexeme>'`

#### 2. operator lookahead errors (errors 4 and 5)
* **variables to use:** `lineOp` and `colOp` (the position captured right before checking the lookahead).
* **why:** `:` not followed by `=` (error 4) or `!` not followed by `=` (error 5) is reported at the position of the `:` or `!` character itself.
* **example message:** `Error 4 at line <lineOp>, column <colOp>: ':' must be followed by '='`

#### 3. single character and byte errors (errors 1 and 10)
* **variables to use:** `line` and `column`.
* **why:** when an invalid character (error 1) or an invalid byte (error 10) is read, report the exact position where that single byte sits in the file.
* **example message:** `Error 1 at line <line>, column <column>: invalid character '<c>'`

#### 4. comment errors (errors 7, 8, and 9)
* **error 7 (comment not closed before end of file):**
  * **variables to use:** `commentStartLine` and `commentStartCol`.
  * **why:** store line and column when the opening `/*` is first encountered; if the file reaches EOF before `*/` closes it, report the position of the opening `/`, not the EOF position.
* **error 8 ('*/' without matching '/*'):**
  * **variables to use:** `lineOp` and `colOp` (the position of the `*` character).
* **error 9 ('/*' inside a comment):**
  * **variables to use:** `line` and `column` of the nested `/` delimiter.

#### 5. empty source error (error 11)
* **variables to use:** literal `1` and `1`.
* **why:** when the file has no tokens at all, the specification defines its position strictly as line 1, column 1.
* **message:** `Error 11 at line 1, column 1: no tokens in the source program`

### What I did with it
I mapped each of the 11 error types to its exact line and column tracking variables (`startLine`/`startCol`, `lineOp`/`colOp`, `commentStartLine`/`commentStartCol`, or literals).

---

## Round 11

### Prompt
when an error occurs do i keep printing tokens bc in one of the test cases it has it keep going even tho the instructions say to stop at the first error

### AI Output
no, you never continue scanning or printing tokens that appear after the error. your scanner must stop immediately at the very first lexical error it encounters.

#### what the specification requires on an error
when an error occurs, your output includes:
1. **the entire source program:** printed in full under `Source Program:`.
2. **the partial tables:** `Lexeme Table:`, `Name Table:`, and `Token List:` containing only the tokens scanned before the error was reached.
3. **the single error line:** printed directly below the `Token List:` section in the exact format required.
4. **the written files:** `tokens.txt` and `nametable.txt` containing only the pre-error tokens and names.
5. **exit:** terminate immediately with a non-zero exit status (e.g., `return 1;` or `exit(1);`).

nothing from the rest of the source file after the error is scanned or printed in the tables.

#### why a test case output might look like it kept going
if an expected output file seems to have many tokens after what you thought was an error, one of two things is happening:
* **the text before the error was long:** case `e15_valid_prefix_then_error` has four lines of valid code before reaching `@` on line 5. all eighteen valid tokens from lines 1 through 5 appear in the lexeme table and token list, followed by `Error 1 at line 5, column 17: invalid character '@'`, while everything after `@` is omitted.
* **a character sequence was valid syntax rather than an error:**
  * `<>` is not an error; it scans as two valid tokens: `<` (code 9) followed by `>` (code 11).
  * `a === b` is not an error; it scans as `a`, `==` (code 7), `=` (code 18), and `b`.
  * a slash or star not forming a comment is valid division (`/`, code 6) or multiplication (`*`, code 5).
  * `odd` is a valid reserved word (code 29), not the identifier `od` followed by a stray `d`.

#### concrete rule to verify
as soon as you detect any of the 11 errors:
1. stop the character scanning loop immediately.
2. run your printing and file-writing routines on the arrays populated up to that moment.
3. print the error line.
4. exit with status 1.

### What I did with it
I understood that scanning must halt at the first lexical error and preserve previous tokens for the name and token tables.

---

## Round 12

### Prompt
after an error do i continue from my while loop or keep going

### AI Output
you do not continue from your while loop at all. you break out immediately, print what you collected up to that point, write the two files, print the error line, and exit the entire program with `return 1;`.

#### what happens when an error triggers
1. **break the scanning loop:** do not read another byte from the file.
2. **classify & build the tables:** run your lookup and name table loops on the tokens collected before the error occurred.
3. **print standard output:**
   * `Lexeme Table:`
   * `Name Table:`
   * `Token List:`
4. **write the two files:** write `tokens.txt` and `nametable.txt` containing only those pre-error items.
5. **print the error message:** output `Error <n> at line <L>, column <C>: <message>` below the token list.
6. **terminate:** `return 1;` (or `exit(1);`).

#### how to structure it in code
set an error flag or error code variable so your program knows it stopped on an error:
```c
int errorCode = 0;
int errLine = 0;
int errCol = 0;
char errLexeme[64] = "";

// example inside scanning loop when error 6 is detected:
if (foundError6) {
    errorCode = 6;
    errLine = startLine;
    errCol = startCol;
    strcpy(errLexeme, temp);
    break; // exit while loop immediately
}
```

after the while loop, proceed through your table generation, printing, and file writing as normal. right after printing `Token List:\n\n`:
```c
if (errorCode != 0) {
    // print the specific error line based on errorCode
    if (errorCode == 6) {
        printf("Error 6 at line %d, column %d: number followed by a letter '%s'\n", errLine, errCol, errLexeme);
    }
    // handle other error codes ...
    return 1; // exit with non-zero status
}

return 0; // normal clean exit
```

#### verification steps
1. add `break;` to every error branch in your while loop so no subsequent characters are read.
2. verify that `tokens.txt` and `nametable.txt` are written before `return 1;` is called.
3. test with `e15_valid_prefix_then_error.txt` and run `echo $?` in your terminal to confirm the status is 1.

### What I did with it
I rearranged my error handling so it branches out of the while loop and later prints and returns 1 after writing to the output files.

---

## Round 13

### Prompt
im really struggling idk what to do. my code is like i tried to set up the thing with tracking the error code and then printing the error at the end and stuff, i didnt implement error 10 bc idk what it wants me to do. and also my code is very long and difficult to follow its just too big for me to test anything i miss almost all the points for lexical errors and also the stuff with comments im just stuck now

### AI Output
it is completely normal to feel overwhelmed right here. trying to scan characters from an active file stream using `fscanf` with `ungetc`, tracking coordinates on the fly, skipping comments, and catching eleven different edge-case errors all inside one loop gets tangled very fast.

you do not have to struggle through that messy file-stream state. there is a clean, standard way to simplify this entire program so that every single error and comment rule becomes straightforward to test.

#### why it feels messy right now
when using `fscanf` and `ungetc`:
* looking ahead to see if `/` is a comment (`/*`) or a division operator requires peeking and ungetting characters.
* checking if `*` is followed by `/` (comment end or error 8) requires more peeking.
* unwinding column after an `ungetc` is difficult because the stream does not remember previous coordinates.

#### the clean way: buffer the whole file first
read the entire file into a single character array at the very start:
```c
char src[100000];
int srcLen = 0;
```
1. print `Source Program:\n\n` and echo the whole buffer directly.
2. loop through `src[i]` with a simple index `i` from 0 to `srcLen`.
   * peeking ahead is just `src[i + 1]` without needing `ungetc`.
   * looking at two characters is just `src[i] == '/' && src[i + 1] == '*'`.

#### understanding error 10
error 10 is very simple once you know what it is asking for:
* an ordinary character in a PL/0 file is either a letter, digit, valid symbol (`+`, `-`, etc.), or whitespace (` `, `\t`, `\n`, `\r`).
* if a printable ASCII character like `@`, `$`, `#`, `?`, `~`, or `` ` `` appears, that is error 1 (`invalid character '<c>'`).
* if a byte is outside the printable range `0x20` to `0x7E` and is not one of the 4 whitespace characters (`\n`, `\r`, `\t`), that is error 10!
  * this includes things like a null byte (`0x00`), control characters, or non-ASCII bytes like smart quotes pasted from a word processor.
* you do not print the byte itself (which would corrupt the terminal); you format its hex value:
  ```c
  printf("Error 10 at line %d, column %d: byte 0x%02X is not part of this language\n", errLine, errCol, (unsigned char)b);
  ```

#### handling comments without getting lost
when you hit `src[i] == '/' && src[i + 1] == '*'`:
1. record where it opened: `commentStartL = line; commentStartC = col;`.
2. advance past `/*`: `i += 2; col += 2;`.
3. run an inner loop until you find `*/`:
   * if `src[i] == '\n'`: `line++; col = 1; i++;`
   * if `src[i] == '\r'`: `i++;` (advances nothing)
   * if `src[i] == '/' && src[i + 1] == '*'`: that is a nested comment opener $\rightarrow$ error 9 (`'/*' inside a comment`)!
   * if `src[i] == '*' && src[i + 1] == '/'`: comment is closed! advance `i += 2; col += 2;` and `break`.
   * if `i >= srcLen`: reached EOF without closing $\rightarrow$ error 7 (`comment is not closed before end of file` at `commentStartL`, `commentStartC`)!
   * otherwise: `col++; i++;`

and outside a comment:
* if `src[i] == '*' && src[i + 1] == '/'`: that is a stray closing comment $\rightarrow$ error 8 (`'*/' without a matching '/*'` at line, col)!

#### handling words and numbers (errors 2, 3, 6)
* when `isalpha(src[i])`:
  * collect the whole run of letters and digits while `isalnum(src[i])`.
  * if length > 12: trigger error 2 (`identifier too long '<lexeme>'`).
  * otherwise: store the word.
* when `isdigit(src[i])`:
  * collect the whole run of letters and digits while `isalnum(src[i])`.
  * check if any character in the collected run is a letter:
    * if yes: trigger error 6 (`number followed by a letter '<lexeme>'`)!
    * if no: it is purely digits. if length > 6, trigger error 3 (`number too long '<lexeme>'`).
    * otherwise: store the number.

#### handling operators (errors 4, 5)
* if `src[i] == ':'`:
  * if `src[i + 1] == '='`: valid `:=`, advance `i += 2; col += 2;`
  * otherwise: trigger error 4 (`':' must be followed by '='` at current line, col)!
* if `src[i] == '!'`:
  * `src[i + 1] == '='`: valid `!=`, advance `i += 2; col += 2;`
  * otherwise: trigger error 5 (`'!' must be followed by '='` at current line, col)!

#### empty source check (error 11)
after your scanning loop finishes, before printing the tables:
* if `numTokens == 0` and no other error was triggered:
  * that is error 11 (`no tokens in the source program` at line 1, column 1)!

#### immediate next steps
1. do not try to patch five things at once; take your current code and move the input file into an in-memory buffer `src` first.
2. write a helper function `void reportError(int code, int l, int c, const char *msg)` that sets the error variables and breaks out.
3. run `./test_lex.sh lex.c --quiet` to see your score jump on the comment and lexical error test suites.

### What I did with it
I understood how buffering the source avoids retracing the input, asked about the definition for Error 10, and looked at the conditions for errors 4, 5, 7, 8, 9, and 11.

---

## Round 14

### Prompt
can you take a look at my code and see what errors there are with the lexical error checking bc i seem to be missing smth that makes the test cases fail:

```c
#include<stdio.h>
#include<string.h>

//checks and returns if word is a single reserved word
int isSingleKeyWord(char character, char single[], int numSingle) {
    for(int i = 0; i < numSingle; i++) {
        if(character == single[i]) {
            return 1;
        }
    }
    return 0;
}

//checks if a string is only digits
int isOnlyDigits(char lexT[]) {
    //if string is empty, not only digits
    if(lexT[0] == '\0') return 0;
    int k = 0;
    while(lexT[k] != '\0') {
        if(!((lexT[k] >= '0') && (lexT[k] <= '9'))) return 0;
        k++;
    }
    return 1;
}

int main(int argc, char *argv[]) {
    //check if argc is valid
    if(argc != 2) {
        printf("Usage: ./lex <input file>\n");
        return 1;
    }
    //open the input file
    FILE *ptr = fopen(argv[1], "r");
    //check if input file is valid
    if(ptr == NULL) {
        printf("Error: unable to open input file '%s'\n", argv[1]);
        return 1;
    }
    //variables to track line, column, and text
    int line = 1;
    int column = 1;
    int startLine = 1;
    int startCol = 1;
    int lineOp = 1;
    int colOp = 1;
    int maxTokenLen = 64;
    int maxTokens = 5000;
    //array to store name tokens
    char nameTokens[maxTokens][maxTokenLen];
    int nameElements = 0;
    int nameLine[maxTokens];
    int nameCol[maxTokens];
    //token array for symbols (excludes letter and digit case: codes 1, 2)
    char singleKeys[] = {'+', '-', '*', '/', '(', ')', '=', ',', '.', '<', '>', ';', ':', '!'};
    int numSingle = 14;
    //2D array to store input text and parse
    char readChar;
    char rTemp;
    //token arrays to store general tokens
    char tokens[maxTokens][maxTokenLen];
    int numTokens = 0;
    //track the indices and type of tokens
    int tokenCode[maxTokens];
    int tokenIndex[maxTokens];
    //track the token line and columns
    int tokenLine[maxTokens];
    int tokenCol[maxTokens];
    int index = 0;
    char temp[maxTokenLen];
    int commentLine = 1;
    int commentCol = 1;
    int inDigit = 0;
    int inComment = 0;
    int error = 0;
    //output
    printf("\nSource Program:\n\n");
    //scan through and print the input file
    while(fscanf(ptr, "%c", &readChar) != EOF) {
        printf("%c", readChar);
        if(error != 0) continue;
        //check carriage return and new line chars
        if(readChar == '\r') continue;
        if(readChar == '\n') {
            line++;
            column = 1;
            //end word if it falls at a new line
            if(index > 0) {
                temp[index] = '\0';
                //error 2: check if lexeme is longer than 12 chars
                if(index > 12) {
                    error = 2;
                    break;
                }
                //check if a letter is in a word that starts with digits
                if(inDigit && (((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')))) {
                    error = 6;
                    break;
                }
                inDigit = 0;
                //error 3: check if number is longer than 6 digits
                if(inDigit && index > 6) {
                    error = 3;
                    break;
                }
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            continue;
        }
        //check if char is an alphanumeric char
        if(((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')) || ((readChar >= '0') && (readChar <= '9'))) {
            //set line and column tracker for new word
            if(index == 0) {
                if((readChar >= '0') && (readChar <= '9')) {
                    inDigit = 1;
                }
                startLine = line;
                startCol = column;
            }
            temp[index] = readChar;
            index++;
            column++;
        }
        //check if char is a white space and add if new word is being scanned
        else if(((readChar == ' ') || (readChar == '\t'))) {
            if(index > 0) {
                //end building the current word
                temp[index] = '\0';
                //check if a letter is in a word that starts with digits
                if(inDigit && (((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')))) {
                    error = 6;
                    break;
                }
                inDigit = 0;
                //error 3: check if number is longer than 6 digits
                if(inDigit && index > 6) {
                    error = 3;
                    break;
                }
                strcpy(tokens[numTokens], temp);
                //add the line and column track values for the current token
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            column++; //update column
        }
        //each if char is a reserved single token
        else if(isSingleKeyWord(readChar, singleKeys, numSingle)) {
            //add current word to token array
            if(index > 0) {
                temp[index] = '\0';
                //check if a letter is in a word that starts with digits
                if(inDigit && (((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')))) {
                    error = 6;
                    break;
                }
                inDigit = 0;
                //error 3: check if number is longer than 6 digits
                if(inDigit && index > 6) {
                    error = 3;
                    break;
                }
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = startLine;
                tokenCol[numTokens] = startCol;
                numTokens++;
                index = 0;
            }
            //track the line and column for long operators
            lineOp = line;
            colOp = column;
            //check if next char makes a longer operator
            if((readChar == '*') || (readChar == '/') || (readChar == '>') || (readChar == '<') || (readChar == '=') || (readChar == '!') || (readChar == ':')) {
                if(fscanf(ptr, "%c", &rTemp) == 1) {
                    //check if chars are comments
                    if((readChar == '*') && (rTemp == '/')) {
                        if(inComment) continue;
                        if(!inComment) { //error 8: */ outside a comment
                            error = 8;
                            break;
                        }
                        inComment = 0; //end comment
                        continue;
                    }
                    else if((readChar == '/') && (rTemp == '*')) {
                        //check if in comment
                        if(inComment) { //error 9: /* inside a comment
                            error = 9;
                            break;
                        }
                        //start comment
                        commentLine = line;
                        commentCol = column;
                        inComment = 1;
                        continue;
                    }
                    //check if : is paired with = (error 4)
                    else if(readChar == ':' && rTemp != '=') {
                        error = 4;
                        break;
                    }
                    //check if ! is paired with = (error 5)
                    else if(readChar == '!' && rTemp != '=') {
                        error = 5;
                        break;
                    }
                    //check if next operator is '='
                    else if(rTemp == '=') {
                        printf("%c", rTemp);
                        //add long operator to token array
                        temp[0] = readChar;
                        temp[1] = '=';
                        temp[2] = '\0';
                        strcpy(tokens[numTokens], temp);
                        tokenLine[numTokens] = lineOp;
                        tokenCol[numTokens] = colOp;
                        numTokens++;
                        column += 2; //long operators take up 2 chars
                    }
                    //store reserved char into array as well if next char isnt '='
                    else {
                        //put scanned char back into input stream
                        ungetc(rTemp, ptr);
                        //store a single char
                        temp[0] = readChar;
                        temp[1] = '\0';
                        strcpy(tokens[numTokens], temp);
                        tokenLine[numTokens] = lineOp;
                        tokenCol[numTokens] = colOp;
                        numTokens++;
                        column++;
                    }
                }
                //store at EOF
                else {
                    temp[0] = readChar;
                    temp[1] = '\0';
                    strcpy(tokens[numTokens], temp);
                    tokenLine[numTokens] = lineOp;
                    tokenCol[numTokens] = colOp;
                    numTokens++;
                    column++;
                }
            }
            //store a single char
            else {
                temp[0] = readChar;
                temp[1] = '\0';
                strcpy(tokens[numTokens], temp);
                tokenLine[numTokens] = lineOp;
                tokenCol[numTokens] = colOp;
                numTokens++;
                column++;
            }
        }
        //error 1: otherwise, char is invalid
        else {
            error = 1;
            break;
        }
    }
    //error 11: check if file has no tokens
    if(numTokens == 0) error = 11;
    //check if file ended without closing an opened comment
    if(inComment) error = 7;
    //add last word to token array once scanning finishes
    if(index > 0) {
        temp[index] = '\0';
        //check if a letter is in a word that starts with digits
        if(inDigit && (((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')))) {
            error = 6;
        }
        inDigit = 0;
        //error 3: check if number is longer than 6 digits
        if(inDigit && index > 6) {
            error = 3;
        }
        strcpy(tokens[numTokens], temp);
        numTokens++;
        index = 0;
    }
    printf("\n");
    char lexT[maxTokenLen];
    int code = 0;
    //loop through token array and print token and code
    for(int i = 0; i < numTokens; i++) {
        strcpy(lexT, tokens[i]);
        if(strcmp(lexT, "+") == 0) code = 3;
        else if(strcmp(lexT, "-") == 0) code = 4;
        else if(strcmp(lexT, "*") == 0) code = 5;
        else if(strcmp(lexT, "/") == 0) code = 6;
        else if(strcmp(lexT, "==") == 0) code = 7;
        else if(strcmp(lexT, "!=") == 0) code = 8;
        else if(strcmp(lexT, "<") == 0) code = 9;
        else if(strcmp(lexT, "<=") == 0) code = 10;
        else if(strcmp(lexT, ">") == 0) code = 11;
        else if(strcmp(lexT, ">=") == 0) code = 12;
        else if(strcmp(lexT, "(") == 0) code = 13;
        else if(strcmp(lexT, ")") == 0) code = 14;
        else if(strcmp(lexT, ",") == 0) code = 15;
        else if(strcmp(lexT, ";") == 0) code = 16;
        else if(strcmp(lexT, ".") == 0) code = 17;
        else if(strcmp(lexT, "=") == 0) code = 18;
        else if(strcmp(lexT, ":=") == 0) code = 19;
        else if(strcmp(lexT, "begin") == 0) code = 20;
        else if(strcmp(lexT, "end") == 0) code = 21;
        else if(strcmp(lexT, "if") == 0) code = 22;
        else if(strcmp(lexT, "fi") == 0) code = 23;
        else if(strcmp(lexT, "then") == 0) code = 24;
        else if(strcmp(lexT, "while") == 0) code = 25;
        else if(strcmp(lexT, "elihw") == 0) code = 26;
        else if(strcmp(lexT, "do") == 0) code = 27;
        else if(strcmp(lexT, "od") == 0) code = 28;
        else if(strcmp(lexT, "odd") == 0) code = 29;
        else if(strcmp(lexT, "call") == 0) code = 30;
        else if(strcmp(lexT, "const") == 0) code = 31;
        else if(strcmp(lexT, "var") == 0) code = 32;
        else if(strcmp(lexT, "procedure") == 0) code = 33;
        else if(strcmp(lexT, "write") == 0) code = 34;
        else if(strcmp(lexT, "read") == 0) code = 35;
        else if(strcmp(lexT, "else") == 0) code = 36;
        else if(isOnlyDigits(lexT)) code = 2;
        else {
            code = 1;
            int foundAtIndex = -1;
            //check if lexT is in name token array already
            for(int k = 0; k < nameElements; k++) {
                if(strcmp(lexT, nameTokens[k]) == 0) {
                    //get the index the word was found inside the name array
                    foundAtIndex = k;
                    break;
                }
            }
            //if token not inside name array, store and update values
            if(foundAtIndex == -1) {
                foundAtIndex = nameElements;
                strcpy(nameTokens[nameElements], lexT);
                //update line and col name array from the values in token line and col
                nameLine[nameElements] = tokenLine[i];
                nameCol[nameElements] = tokenCol[i];
                nameElements++;
            }
            tokenIndex[i] = foundAtIndex;
        }
        tokenCode[i] = code;
    }
    printf("\nLexeme Table:\n\n");
    printf("lexeme\t\ttoken\n");
    //print out the token and
    for(int i = 0; i < numTokens; i++) {
        printf("%s\t\t%d\n", tokens[i], tokenCode[i]);
    }
    printf("\n");
    printf("\nName Table:\n\n");
    printf("index name\t\tline column\n");
    for(int i = 0; i < nameElements; i++) {
        printf("%d %s\t\t\t%d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
    }
    printf("\n");
    printf("\nToken List:\n\n");
    //print out the token codes in tokenCode[]
    for(int i = 0; i < numTokens; i++) {
        if(tokenCode[i] == 1) {
            //print back to back integers stored for token
            printf("%d %d ", tokenCode[i], tokenIndex[i]);
        } else if(tokenCode[i] == 2) {
            //print code and word
            printf("%d %s ", tokenCode[i], tokens[i]);
        } else {
            //print only the token word
            printf("%d ", tokenCode[i]);
        }
    }
    printf("\n");
    //write to file tokens.txt
    FILE *tokenFile = fopen("tokens.txt", "w");
    if(tokenFile != NULL) {
        //check token code and write to file like from printing
        for(int i = 0; i < numTokens; i++) {
            if(tokenCode[i] == 1) {
                //print back to back integers stored for token
                fprintf(tokenFile, "%d %d\n", tokenCode[i], tokenIndex[i]);
            } else if(tokenCode[i] == 2) {
                //print code and word
                fprintf(tokenFile, "%d %s\n", tokenCode[i], tokens[i]);
            } else {
                //print only the token word
                fprintf(tokenFile, "%d\n", tokenCode[i]);
            }
        }
        fclose(tokenFile);
    }
    //write to file nametable.txt
    FILE *nameFile = fopen("nametable.txt", "w");
    if(nameFile != NULL) {
        //loop and print name token array contents with line and columns
        for(int i = 0; i < nameElements; i++) {
            fprintf(nameFile, "%d %s %d %d\n", i, nameTokens[i], nameLine[i], nameCol[i]);
        }
        fclose(nameFile);
    }
    //check and report errors
    if(error != 0) {
        switch(error) {
            case 1:
                printf("Error 1 at line %d, column %d: invalid character '%c\n", line, column, readChar);
                break;
            case 2:
                printf("Error 2 at line %d, column %d: identifer too long '%s'\n", startLine, startCol, tokens[numTokens]);
                break;
            case 6:
                printf("Error 6 at line %d, column %d: number followed by a letter '%s'\n", startLine, startCol, tokens[numTokens]);
                break;
            case 3:
                printf("Error 3 at line %d, column %d: number too long '%s'\n", startLine, startCol, tokens[numTokens]);
                break;
            case 4:
                printf("Error 4 at line %d, column %d: ':' must be followed by'='\n", lineOp, colOp);
                break;
            case 5:
                printf("Error 5 at line %d, column %d: '!' must be followed by'='\n", lineOp, colOp);
                break;
            case 7:
                printf("Error 7 at line %d, column %d: comment is not closed before end of file\n", commentLine, commentCol);
                break;
            case 8:
                printf("Error 8 at line %d, column %d: '*/' without a matching '/*'\n", lineOp, colOp);
                break;
            case 9:
                printf("Error 9 at line %d, column %d: '/*' inside a comment\n", line, column);
                break;
            case 10:
                break;
            case 11:
                printf("Error 11 at line 1, column 1: no tokens in the source program\n");
                break;
        }
        return 1;
    }
    return 0;
}
```

### AI Output
here are the specific issues breaking the lexical error test cases in your code:

#### 1. typos in the error strings
the autograder performs character-for-character string matching on error lines. several of your format strings differ from the specification:
* **error 1:** missing closing single quote.
  * yours: `"Error 1 at line %d, column %d: invalid character '%c\n"`
  * required: `"Error 1 at line %d, column %d: invalid character '%c'\n"`
* **error 2:** typo in identifier.
  * yours: `"identifer too long '%s'\n"`
  * required: `"identifier too long '%s'\n"`
* **errors 4 and 5:** missing space between `by` and `'='`.
  * yours: `"... must be followed by'='"`
  * required: `"... must be followed by '='"`

#### 2. printing tokens[numTokens] instead of temp
for errors 2, 3, and 6:
```c
printf("Error 2 at line %d, column %d: identifier too long '%s'\n", startLine, startCol, tokens[numTokens]);
```
when an error triggers, you execute `break;` before copying `temp` into `tokens[numTokens]`. `tokens[numTokens]` is empty or uninitialized memory!
* **fix:** print `temp` (or a dedicated `errLexeme` string where `temp` was copied).

#### 3. the error check conditions for words and numbers
in your newline and whitespace branches:
```c
// check if a letter is in a word that starts with digits
if(inDigit && (((readChar >= 'a') && (readChar <= 'z')) || ((readChar >= 'A') && (readChar <= 'Z')))) {
    error = 6;
    break;
}
inDigit = 0;
// error 3: check if number is longer than 6 digits
if(inDigit && index > 6) { ... }
```
two bugs here:
1. `readChar` in this branch is whitespace (like `' '`, `'\t'`, or `'\n'`). it will never be `'a'..'z'`! the letters were read into `temp` earlier, so checking `readChar` here never catches `123abc`.
2. you set `inDigit = 0;` on the line immediately before checking `if (inDigit && index > 6)`. because `inDigit` was just cleared to 0, error 3 can never trigger!

**how to fix:**
when an alphanumeric run ends (when whitespace, a newline, or an operator is hit), examine the word stored in `temp`:
```c
temp[index] = '\0';
if (inDigit) {
    int hasLetter = 0;
    for (int k = 0; k < index; k++) {
        if (((temp[k] >= 'a') && (temp[k] <= 'z')) || ((temp[k] >= 'A') && (temp[k] <= 'Z'))) {
            hasLetter = 1;
            break;
        }
    }
    if (hasLetter) {
        error = 6;
        break;
    }
    if (index > 6) {
        error = 3;
        break;
    }
} else {
    if (index > 12) {
        error = 2;
        break;
    }
}
inDigit = 0;
```

#### 4. comment handling and printf("%c", readChar)
1. **source echo on error:**
   ```c
   while(fscanf(ptr, "%c", &readChar) != EOF) {
       printf("%c", readChar);
       if(error != 0) continue;
   ```
   when `error != 0`, your loop does `break;` in most branches, but in others it does not echo the remaining source program. the specification requires printing the complete source program under `Source Program:` regardless of errors.
2. **characters inside comments:** inside a comment, every character—including operators, whitespace, and letters—is ignored until `*/` or EOF is reached. your scanner currently routes comment body characters into `isSingleKeyWord` and alphanumeric, creating tokens while `inComment == 1`!
   * inside a comment, do not generate tokens.
   * look only for `*/` (to close) and `/*` (to trigger error 9).

#### 5. implement error 10
error 10 is required whenever an invalid byte is not printable ASCII and not whitespace:
```c
unsigned char b = (unsigned char)readChar;
if (b < 0x20 || b > 0x7E) {
    if (b != '\n' && b != '\r' && b != '\t') {
        error = 10;
        errByte = b;
        break;
    }
}
```
in your error switch:
```c
case 10:
    printf("Error 10 at line %d, column %d: byte 0x%02X is not part of this language\n", line, column, errByte);
    break;
```

#### 6. error 11 false triggers
at the bottom:
```c
// error 11: check if file has no tokens
if(numTokens == 0) error = 11;
```
if another error occurred earlier (for example, error 1 on the very first character), `numTokens` is 0, which overwrites your real error with error 11!
* **fix:** `if (numTokens == 0 && error == 0) error = 11;`

### What I did with it
I fixed the spelling mistakes in the error strings, switched the error printouts from `tokens[numTokens]` to `temp`, corrected the loop with `temp` for digits and letters, added byte checks for Error 10, and added `error == 0` for Error 11's check.

---

## Round 15

### Prompt
when i see a comment how am i supposed to not make any tokens and why doesnt my continue part work

### AI Output
your `continue` does not work because of where it sits inside your control flow.

#### why your continue fails
in your code:
```c
if ((readChar == '*') || (readChar == '/') || ...) {
    if(fscanf(ptr, "%c", &rTemp) == 1) {
        // 1. this check happens BEFORE finding the comment opener!
        if(inComment) continue;

        if((readChar == '/') && (rTemp == '*')) {
            inComment = 1;
            continue; // 2. continues the outer while loop
        }
    }
}
```
here is what happens step-by-step:
1. you read `/` and `*`. you set `inComment = 1` and run `continue`.
2. the outer while loop reads the next character inside the comment (say `v` from `/* var x */`).
3. at the top of the while loop, `v` is an alphanumeric character, so it hits `if(((readChar >= 'a') && ...))`!
4. your code starts building a token out of the characters inside the comment because none of your other branches check whether `inComment == 1`!
5. later, when a `+` or `/` appears inside the comment, it enters `isSingleKeyWord`, reaches `if (inComment) continue;`, but `rTemp` was consumed from `ptr` without updating `column` or looking for `*/` properly.

#### how comments must actually be handled
a comment is not an operator, and once it starts, you must discard everything until it closes.
you can handle comments cleanly using one of two methods:

#### method a: an inner loop right when /* is discovered
as soon as you detect `readChar == '/'` and `rTemp == '*'`, do not return to the main tokenizing loop. stay inside an inner loop until the comment finishes:
```c
else if (readChar == '/' && rTemp == '*') {
    if (inComment) {
        // error 9: nested comment opener
        error = 9;
        errLine = line;
        errCol = column;
        break;
    }
    int commentStartLine = line;
    int commentStartCol = column;
    column += 2; // for /*
    int closed = 0;
    char c1;
    while (fscanf(ptr, "%c", &c1) == 1) {
        // echo source
        printf("%c", c1);
        if (c1 == '\r') {
            continue;
        }
        if (c1 == '\n') {
            line++;
            column = 1;
            continue;
        }
        // check for nested opener '/*' -> error 9
        if (c1 == '/') {
            char c2;
            if (fscanf(ptr, "%c", &c2) == 1) {
                printf("%c", c2);
                if (c2 == '*') {
                    error = 9;
                    errLine = line;
                    errCol = column;
                    break;
                } else {
                    ungetc(c2, ptr);
                }
            }
            column++;
            continue;
        }
        // check for closer '*/'
        if (c1 == '*') {
            char c2;
            if (fscanf(ptr, "%c", &c2) == 1) {
                printf("%c", c2);
                if (c2 == '/') {
                    column += 2;
                    closed = 1;
                    break; // comment is closed!
                } else {
                    ungetc(c2, ptr);
                    column++;
                }
            } else {
                column++;
                continue;
            }
        }
        column++;
    }
    if (error != 0) {
        break;
    }
    if (!closed) {
        // reached EOF without closing -> error 7
        error = 7;
        errLine = commentStartLine;
        errCol = commentStartCol;
        break;
    }
    continue; // now safely continue the main loop outside the comment
}
```

#### method b: check inComment at the very top of the outer loop
if you keep `inComment` as a flag, you must place the check at the very top of the while loop so no other branches (letters, numbers, operators) run while inside a comment:
```c
while (fscanf(ptr, "%c", &readChar) != EOF) {
    printf("%c", readChar);

    if (inComment) {
        if (readChar == '\r') continue;
        if (readChar == '\n') {
            line++;
            column = 1;
            continue;
        }
        // look for closer */
        if (readChar == '*') {
            if (fscanf(ptr, "%c", &rTemp) == 1) {
                printf("%c", rTemp);
                if (rTemp == '/') {
                    column += 2;
                    inComment = 0; // comment closed
                    continue;
                } else {
                    ungetc(rTemp, ptr);
                }
            }
        }
        // look for nested /* -> error 9
        if (readChar == '/') {
            if (fscanf(ptr, "%c", &rTemp) == 1) {
                printf("%c", rTemp);
                if (rTemp == '*') {
                    error = 9;
                    errLine = line;
                    errCol = column;
                    break;
                } else {
                    ungetc(rTemp, ptr);
                }
            }
        }
        column++;
        continue; // ignore everything else inside comments
    }

    // regular tokenizing logic runs only when inComment == 0
```

#### checking for stray */ outside comments (error 8)
when `inComment == 0`, if `readChar == '*'` and the next char is `'/'`, that is an unattached closer:
```c
if (readChar == '*' && rTemp == '/') {
    error = 8;
    errLine = lineOp;
    errCol = colOp;
    break;
}
```

#### action steps
1. choose either method a (eat the whole comment inside its own loop) or method b (guard the entire body of your while loop with `if (inComment)`).
2. ensure characters read inside comments never reach the alphanumeric or operator arrays.
3. test on a comment case like `c01_one_comment` to verify the comment disappears completely from your lexeme table and token list.

### What I did with it
I updated the top of my reading loop so characters inside comments are ignored without making tokens, while properly detecting `*/` closers.
