## Round 1

### Prompt
hi i was wondering if you could help me approach my program with the file attached. i have the lexeme and token matching working but im not sure what to write for the output file and also my token list is having trouble
'''
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

int main(int argc, char* argv[]) {

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
	char singleKeys[] = {'+', '-', '*', '/', '<', '>', '(', ')', ',', ';', '.', '=', '!', ':'};

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
	int numCol = 0; //ISSUE: figure out how to
			//store line and col while reading file
			//in while loop below

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
			//printf("%c", readChar);
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
	printf("index  name\t\tline  column\n");

	for(int i = 0; i < nameElements; i++) {
		printf("%d  %s\t\t\t%d     %d\n", i, nameTokens[i], line, column);
	}

	printf("\n");

	printf("\nToken List:\n\n");

	//print out the token codes in tokenCode[]
	for(int i = 0; i < numCodes; i++) {
		printf("%d ", tokenCode[i]);
	}

	printf("\n");
}
'''
### AI Output
the issues in the token list and file writing come from how the scanner structures token data and how positions are tracked.

---

### 1. why the token list is having trouble

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
}
tokenCode[numCodes] = code;
numCodes++;
