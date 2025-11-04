# Calculator-using-only-C
This project contains source code of a very basic calculator that we use in day to day life
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>
#include <math.h>

#define MAX 256

// ======= STACKS =======
double numStack[MAX];
char opStack[MAX];
int numTop = -1, opTop = -1;

void pushNum(double num) { numStack[++numTop] = num; }
double popNum() { return numStack[numTop--]; }

void pushOp(char op) { opStack[++opTop] = op; }
char popOp() { return opStack[opTop--]; }
char peekOp() { return opStack[opTop]; }
int isEmptyOp() { return opTop == -1; }

// ======= PRECEDENCE =======
int precedence(char op) {
    if (op == '+' || op == '-') return 1;
    if (op == '*' || op == '/' || op == '%') return 2;
    return 0;
}

// ======= APPLY OPERATION =======
void applyOp(char op) {
    double b = popNum();
    double a = popNum();
    double res = 0;

    switch (op) {
        case '+': res = a + b; break;
        case '-': res = a - b; break;
        case '*': res = a * b; break;
        case '/':
            if (b == 0) {
                printf("Error: Division by zero!\n");
                exit(1);
            }
            res = a / b;
            break;
        case '%':
            res = fmod(a, b);
            break;
    }
    pushNum(res);
}

// ======= MAIN FUNCTION =======
int main() {
    char expr[MAX];
    printf("----- Ultimate Flexible Calculator -----\n");
    printf("Enter an expression (e.g. 2+2-4/6*7 or (5+3)*2-4/2):\n");

    fgets(expr, sizeof(expr), stdin);

    for (int i = 0; i < strlen(expr); i++) {
        char ch = expr[i];

        // Skip spaces
        if (isspace(ch))
            continue;

        // Read number (including decimals)
        if (isdigit(ch) || ch == '.') {
            char numStr[32];
            int j = 0;

            while (isdigit(expr[i]) || expr[i] == '.') {
                numStr[j++] = expr[i++];
            }
            numStr[j] = '\0';
            pushNum(atof(numStr));
            i--; // Adjust for loop increment
        }

        // Handle '('
        else if (ch == '(') {
            pushOp(ch);
        }

        // Handle ')'
        else if (ch == ')') {
            while (!isEmptyOp() && peekOp() != '(')
                applyOp(popOp());
            if (!isEmptyOp() && peekOp() == '(')
                popOp();
        }

        // Handle operators
        else if (ch == '+' || ch == '-' || ch == '*' || ch == '/' || ch == '%') {
            while (!isEmptyOp() && precedence(peekOp()) >= precedence(ch))
                applyOp(popOp());
            pushOp(ch);
        }

        else {
            printf("Invalid character: %c\n", ch);
            return 1;
        }
    }

    // Apply remaining operators
    while (!isEmptyOp())
        applyOp(popOp());

    printf("Result = %.6lf\n", popNum());
    return 0;
}
