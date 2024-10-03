# internshala-project
internshala project
Here's a basic outline of how you can structure the SimpleLang compiler. This implementation focuses on the lexer, parser, and code generation steps.
1. Lexer (Tokenizer)
This lexer breaks the input into tokens:
#include <stdio.h>
#include <ctype.h>
#include <string.h>

#define MAX_TOKEN_LEN 100

typedef enum {
    TOKEN_INT, TOKEN_IDENTIFIER, TOKEN_NUMBER, TOKEN_ASSIGN,
    TOKEN_PLUS, TOKEN_MINUS, TOKEN_IF, TOKEN_EQUAL, TOKEN_LBRACE, TOKEN_RBRACE,
    TOKEN_SEMICOLON, TOKEN_EOF, TOKEN_UNKNOWN
} TokenType;

typedef struct {
    TokenType type;
    char text[MAX_TOKEN_LEN];
} Token;

void getNextToken(FILE *file, Token *token) {
    int c;
    while ((c = fgetc(file)) != EOF) {
        if (isspace(c)) continue;

        // Handle identifiers and keywords
        if (isalpha(c)) {
            int len = 0;
            token->text[len++] = c;
            while (isalnum(c = fgetc(file))) {
                if (len < MAX_TOKEN_LEN - 1)
                    token->text[len++] = c;
            }
            ungetc(c, file);
            token->text[len] = '\0';

            if (strcmp(token->text, "int") == 0) token->type = TOKEN_INT;
            else if (strcmp(token->text, "if") == 0) token->type = TOKEN_IF;
            else token->type = TOKEN_IDENTIFIER;
            return;
        }

        // Handle numbers
        if (isdigit(c)) {
            int len = 0;
            token->text[len++] = c;
            while (isdigit(c = fgetc(file))) {
                if (len < MAX_TOKEN_LEN - 1)
                    token->text[len++] = c;
            }
            ungetc(c, file);
            token->text[len] = '\0';
            token->type = TOKEN_NUMBER;
            return;
        }

        // Handle symbols
        switch (c) {
            case '=': token->type = TOKEN_ASSIGN; token->text[0] = '='; token->text[1] = '\0'; return;
            case '+': token->type = TOKEN_PLUS; token->text[0] = '+'; token->text[1] = '\0'; return;
            case '-': token->type = TOKEN_MINUS; token->text[0] = '-'; token->text[1] = '\0'; return;
            case '{': token->type = TOKEN_LBRACE; token->text[0] = '{'; token->text[1] = '\0'; return;
            case '}': token->type = TOKEN_RBRACE; token->text[0] = '}'; token->text[1] = '\0'; return;
            case ';': token->type = TOKEN_SEMICOLON; token->text[0] = ';'; token->text[1] = '\0'; return;
            case EOF: token->type = TOKEN_EOF; token->text[0] = '\0'; return;
            default: token->type = TOKEN_UNKNOWN; token->text[0] = c; token->text[1] = '\0'; return;
        }
    }
    token->type = TOKEN_EOF;
    token->text[0] = '\0';
}

int main() {
    FILE *file = fopen("input.txt", "r");
    if (!file) {
        perror("Failed to open file");
        return 1;
    }
    Token token;
    do {
        getNextToken(file, &token);
        printf("Token: %d, Text: %s\n", token.type, token.text);
    } while (token.type != TOKEN_EOF);
    fclose(file);
    return 0;
}

2. Parser
This parser processes tokens and builds an Abstract Syntax Tree (AST). The AST is a structured representation of your code.

Here's a simple structure for the AST:

typedef enum { NODE_VAR_DECL, NODE_ASSIGN, NODE_IF, NODE_EXPR } NodeType;

typedef struct Node {
    NodeType type;
    // You can add more fields depending on what data you want each node to store.
    char identifier[MAX_TOKEN_LEN];
    int value;
    struct Node *left;
    struct Node *right;
} Node;

Node* parseVarDecl(Token token) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->type = NODE_VAR_DECL;
    strcpy(node->identifier, token.text);  // store identifier
    return node;
}

Node* parseAssignment(Token token) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->type = NODE_ASSIGN;
    strcpy(node->identifier, token.text);  // store identifier
    return node;
}

3. Code Generation
The following code traverses the AST and generates assembly code. It will map each operation (like addition, assignments, etc.) to the corresponding assembly instructions for the 8-bit CPU.
void generateAssembly(Node* node) {
    switch (node->type) {
        case NODE_VAR_DECL:
            printf("Declare variable %s\n", node->identifier);
            break;
        case NODE_ASSIGN:
            printf("MOV %s, %d\n", node->identifier, node->value); // assuming value is stored in the AST node
            break;
        case NODE_EXPR:
            printf("ADD %s, %s\n", node->left->identifier, node->right->identifier); // assuming it's an addition expression
            break;
        case NODE_IF:
            printf("CMP %s, %d\n", node->identifier, node->value); // compare condition
            printf("JNE else_label\n"); // jump if condition is false
            // body of if
            printf("else_label:\n"); // else label
            break;
        default:
            break;
    }
}

4. Integration
You will need to combine the lexer, parser, and code generator into a single workflow:

Lexer: Reads the SimpleLang code and breaks it into tokens.
Parser: Reads the tokens and builds the AST.
Code Generator: Traverses the AST and produces 8-bit assembly code.
5. Testing
Once integrated, you can test with an input like this:

input.txt:

int a;
a = 10;
int b;
b = a + 20;
if (a == 10) {
    a = a + 1;
}

