cc500
=====

CC500: a tiny self-hosting C compiler

http://homepage.ntlworld.com/edmund.grimley-evans/cc500/


CC500: a tiny self-hosting C compiler

Introduction

I wrote this tiny compiler, which translates a subset of C into x86 machine code, for fun. It has no use, unless it counts as educational. I called it CC500 because I initally guessed it would take about 500 lines. It turned out to be about 600 lines even without the comments and blank lines. With the comments and blank lines it has about 750 lines. It could be made shorter, but I wanted the code to be clear and simple rather than obfuscated.

Download

cc500.c - just the code
Compilation:

gcc cc500.c -o cc500_1
Self-compilation:

./cc500_1 < cc500.c > cc500_2
chmod +x cc500_2
How it works

The compiler reads the whole program in C from stdin and writes an ELF binary for Linux/x86 to stdout. The ELF binary is somewhat non-standard as it consists of a single section that is both writable and executable. It seems to work for me on a real Linux box, but it might cause problems on an emulator.

No libraries are used. Tiny, machine-code implementations of exit(), getchar(), malloc() and putchar() are included in the binary that is generated. There is no free() as malloc() is just sbrk(), really, implemented with two calls to brk().

There is almost no error-checking. With some invalid C programs the compiler calls exit(1) instead of generating a binary, but in most cases it generates a binary that crashes when you run it.

There is no preprocessor. The lexer looks for tokens that match one of the following Perl regular expressions: /[a-z0-9_]+/, /[<=>|&!]+/, /'[^']*'/, /"[^"]*"/, /./. Traditional C-style comments are handled, but not C++-style comments.

The symbol table is implemented with a single array of char. The type of each symbol is recorded as 'D' (defined global), 'U' (undefined global), 'L' (local variable) or 'A' (argument of a function) - see sym_get_value(). There is also an address recorded for each symbol, but no type, as there is no type-checking and all arrays are assumed to be arrays of char. The address of an 'L' or 'A' is its position on the stack. The address of a 'D' is the position of that function or global variable in the binary. The address of a 'U' is also a pointer into the binary, but a pointer to where the symbol was last used and the head of a linked list of all the forward references to that symbol.

Scoping rules for local variables are implemented correctly as it is easy to do so: at the end of a compound statement the pointer to the end of the symbol table is restored to what it was at the start of the compound statement - see the assignment to table_pos in statement().

I started off writing CC500 without local variables, but it felt very unnatural. Nested scopes are an essential part of C and it is not easy to write a recursive-descent parser without automatic variables.

It is assumed that all variables, arguments and return values are char * or int, though the compiler does not even parse the types, let alone record them in the symbol table. The functions that implement the recursive-descent parser return 1, 2 or 3, corresponding to a "char lval", "int lval" or "other" - see promote().

Each parsing function is preceded by a comment that documents the syntactic structures recognised, in an obvious notation. In some cases the code accepts additional, incorrect structures in addition to what is documented in the comment. Only a tiny subset of C's operators are accepted: just the ones I needed while writing the compiler. There are no unary operators: instead of the indirection operator * you must use x[y], where it is assumed that x is char * and y is int. The only statements recognised are compound statements, expressions, if, while and return. Local variable declarations, with initialisers, are treated as a kind of statement. (If you write something like if (x) int y; the compiler will not complain, but the binary will crash.)

There are no arbitary limits on the length of symbols, number of symbols are the like: all three buffers (token, table and code) are supposed to be automatically extended as required by calls to my_realloc().

Extending it

Some simple and obvious enhancements would be to print an appropriate message message when an error is detected, check for undefined globals, and allow main() to be anywhere in the file.

It would be easy to extend the compiler to handle the remaining operators and statement types of C. To implement break and continue you would probably add additional arguments to the parsing functions that record the address and stack height of the current loop. The easiest way to implement switch statements might be to add an argument that records the location of the previous case label in the current switch statement; a case label would then be translated with a comparison and a branch forwards to the next case label. In general, C is quite amenable to direct translation into machine code without going through an IR (intermediate representation).

Extending the type system would be a bit more challenging. You could correctly implement scalars and pointers, with type checking, by adding a single integer field to the symbol table to record the type. However, if you wanted to implement structs, or function pointers with proper type checking, you would need a recursive data structure to represent the types. It is interesting, but not really surprising, that you start to start to need structs in the implementation language at roughly the same point that you start to implement them. However, C's types get interesting even without structs, unions and typedefs. For example, void (*(*f())(void (*)(void)))(void); declares a function that returns a pointer to a function that takes a pointer to a function as an argument and returns another pointer to a function. I must admit to finding that part of C's syntax rather difficult to parse mentally. I'm glad we have computers to do it for us.

Links

bcompiler - Bootstrapping a simple compiler from nothing
OTCC - The smallest self compiling pseudo C compiler
Tiny ELF programs by Brian Raiter
Edmund Grimley Evans
