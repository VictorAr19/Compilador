# Compilador

# Development

## Lexical Analyzer (Lexer)

The lexical analyzer constitutes the first phase of the compilation process and serves the fundamental purpose of scanning the source code to transform it into an ordered sequence of tokens. Each token represents a basic lexical unit of the language and contains information about its type, the corresponding lexeme, and its exact location in the source code through line and column coordinates.

The lexer implementation was carried out in the `user_lexer.py` module, using compiled regular expressions and deterministic finite automata for efficient pattern recognition. The system defines six main token categories: reserved words (keywords), identifiers, punctuation symbols, operators, numeric constants, and string literals. Each category is associated with a regular expression pattern that establishes the valid formation rules for its lexemes.

During execution, the lexer processes the code character by character, applying a matching algorithm that prioritizes the recognition of the longest possible token at each position. The system maintains a continuous record of the current line and column, which allows generating precise error messages when invalid characters or malformed tokens are detected.

A fundamental aspect of the design is the intelligent handling of non-significant elements: whitespace, tabs, and line breaks are processed but not converted into tokens, while single-line comments (delimited by `//`) and multi-line comments (delimited by `/* */`) are identified and completely omitted from the analysis. This special treatment is crucial because these elements, although present in the source code, do not influence the syntactic structure of the program.

The lexer implements a robust error detection system that not only identifies the exact position of the error but also provides context about the problematic symbol and suggestions about the possible nature of the error. For example, if an unexpected character is detected that could be part of an unclosed string literal, the system explicitly indicates this.

The final result of the lexical analysis is an ordered list of tuples, where each tuple contains the token type, the lexeme, the line, and the column where it appears. This structured information is complemented by a counter dictionary that records the number of tokens detected per category, facilitating both system debugging and statistical analysis of the processed code.

## Token Adapter

The adapter, implemented in the `adapter_lexer.py` module, represents an essential conversion layer in the compiler architecture. Its existence responds to a fundamental principle of software engineering: decoupling between components. The direct output of the lexical analyzer is designed exclusively for its internal representation, but the parser requires working with a normalized and consistent format.

The adapter acts as a bidirectional translator that encapsulates each token in a well-defined data structure through the `Token` class. This class contains four key attributes: the abstract type of the token, the original lexeme, the literal value (when applicable), and the position in the source code. This structural normalization ensures that the parser always works with uniform objects, regardless of variations in the underlying lexer implementation.

Beyond structural normalization, the adapter performs a critical semantic mapping process. Punctuation symbols and operators, which the lexer identifies by their literal textual representation, are transformed into canonical token names. For example, the `;` symbol becomes the `SEMI` token, the `=` operator becomes `ASSIGN`, and the opening brace `{` becomes `LBRACE`. This level of abstraction allows the parser to work exclusively with semantic token categories rather than language-specific symbols.

The adapter also handles specialized processing of literals. Numeric values are converted to their appropriate types (integers or floats), while text strings are processed by removing the delimiting quotes and preserving only the content. This preprocessing significantly simplifies the parser's work in later stages.

An additional responsibility of the adapter is to ensure the formal completeness of the token stream. To this end, it automatically adds an `EOF` (End Of File) token at the end of the sequence, unequivocally signaling the end of the input. This practice prevents premature parsing errors and facilitates the implementation of predictive syntactic analysis algorithms.

The adapter architecture reflects solid software design principles: it promotes low coupling between the lexer and parser, maintains high cohesion in each component, and facilitates independent evolution of both phases. This abstraction layer allows completely replacing the lexer implementation or modifying the tokenization strategy without needing to alter the parser or subsequent compiler stages.

## Parser and Syntax-Directed Translation

The parser constitutes the core of the syntactic-semantic analysis phase of the compiler. Its dual responsibility consists of verifying that the token sequence conforms to the grammatical structure of the language and constructing a hierarchical representation of the program in the form of an Abstract Syntax Tree (AST). Simultaneously, it incorporates a Syntax-Directed Translation (SDT) layer that executes semantic actions in real-time during the analysis process.

### Recursive Descent LL(1) Parser Design

The implementation was based on a recursive descent parser that follows the LL(1) parsing strategy. This design decision was founded on multiple theoretical and practical advantages. First, the recursive descent approach establishes a direct correspondence between grammar production rules and code functions, creating a transparent and highly readable implementation structure.

The fundamental characteristic of LL(1) analysis lies in its predictive nature: the parser examines only the next token (hence the "1" in LL(1)) to unequivocally determine which production rule to apply. This determinism eliminates the need for backtracking and considerably simplifies both the implementation and reasoning about the parser's behavior.

The choice of a top-down approach offers significant advantages for integrating semantic actions. By processing code in a descending manner from the start symbol, the parser can execute semantic validations synchronously with syntactic analysis. This contrasts with bottom-up approaches, where semantic actions must be deferred until reductions are completed.

### Grammar Refinement

To guarantee the grammar's compatibility with LL(1) predictive analysis, systematic transformations were applied. Left recursion, which is incompatible with descent analysis, was completely eliminated and replaced with right recursion. Ambiguous productions were restructured to eliminate any decision conflicts.

A particularly important case was the treatment of arithmetic and logical expressions. These were decomposed into multiple hierarchical precedence levels: term (multiplication, division, modulo), additive expression (addition, subtraction), relational expression (comparisons), equality expression, logical AND, and logical OR. This stratification allows the parser to make deterministic decisions about which rule to apply in each context, while also respecting standard operator precedences.

### Syntax-Directed Translation (SDT)

The SDT system integrated into the parser transcends mere syntactic validation to perform deep semantic verifications during analysis. This integration is implemented through actions embedded directly in the recursive parsing functions, executing at the precise moment when each language construct is recognized.

The central component of semantic analysis is a type checking system that rigorously validates each operation, assignment, and function return. The parser maintains a local symbol table for each function, recording declared variables along with their types. This context isolation prevents variables defined within a function from being accidentally referenced from outside, eliminating confusions and erroneous redefinitions.

Complementarily, a global function table is maintained that records all function declarations in the program. For each function, its return type and the complete list of parameters with their respective types are stored. This information is fundamental for validating function calls, ensuring that the correct arguments are provided both in number and type.

One of the most critical validations performed by the parser is return consistency verification. The system tracks each function's expected return type and validates that all `return` statements provide a value of the correct type. This verification prevents subtle errors where a function might return an unexpected type or not return a value at all when required.

### AST and Parser Extensions

To support these advanced semantic capabilities, the AST architecture was significantly expanded. Six new node types were introduced: `FuncDecl` for function declarations, `Block` for code blocks delimited by braces, `Return` for return statements, `FuncCall` for function invocations, `IfStmt` for conditionals, `WhileStmt` for while loops, and `ForStmt` for for loops. Additionally, the `Program` node was strengthened to represent complete programs with multiple functions and nested structures.

The parser implementation incorporates specialized methods to process each of these constructs. The `func_decl()` method handles function declaration including its parameters and body, `block()` processes code blocks maintaining the local variable context, and the `if_stmt()`, `while_stmt()`, and `for_stmt()` methods manage control structures with validation of their conditions and bodies.

A particularly sophisticated aspect is the handling of function calls. The parser implements a first pass over the code to collect all function declarations before beginning the main analysis. This allows validating calls to functions that may be declared later in the source code, thus supporting the common pattern of defining auxiliary functions after their use.

### Advantages of the Implemented Approach

The combination of recursive descent parsing with embedded SDT actions achieves an elegant integration of syntactic and semantic analysis. The resulting system is deterministic, which guarantees predictable and linear analysis times. It is modular, allowing grammar extension simply by adding new recursive functions. And it is extensible, facilitating the incorporation of new language constructs without architectural restructuring.

The conceptual clarity of the design makes it particularly valuable in academic contexts, where implementation transparency facilitates understanding of theoretical compilation principles. At the same time, the robustness of the semantic verification system makes it practical for real applications, detecting early a wide range of programming errors.

## Intermediate Code Generation

The intermediate code generation phase, implemented in the `ir_generator.py` module, transforms the AST into an intermediate representation based on three-address code (TAC). This intermediate representation acts as a crucial bridge between high-level analysis and machine code generation.

Three-address code imposes a fundamental restriction: each instruction can contain at most three operands. This limitation greatly simplifies subsequent optimization and code generation phases by standardizing all complex operations into a sequence of simple atomic operations. For example, an expression like `x = a + b * c` is decomposed into separate instructions: first `t1 = b * c` is calculated, then `x = a + t1`.

The IR generator implements a visitor system that recursively traverses the AST, translating each node to one or more IR instructions. To manage intermediate values, the system maintains a temporary counter that generates unique identifiers (`t0`, `t1`, `t2`, etc.) to store partial calculation results. Similarly, a label counter generates unique names (`L0`, `L1`, `L2`, etc.) for jump points in control structures.

Arithmetic and logical expressions are translated to sequences of `IRBinOp` instructions representing elementary binary operations. Each binary operation from the AST generates an IR instruction that stores its result in a new temporary, which can be referenced by subsequent instructions. Unary operations are handled analogously through `IRUnaryOp` instructions.

Control structures require special treatment. `if-else` conditionals are translated to a sequence of condition evaluation, conditional jump to the appropriate label, execution of the then block, unconditional jump to the end (if else exists), else label, and finally end label. `while` loops are implemented with a start label where the condition is evaluated, a conditional jump to the end if the condition is false, execution of the body, and an unconditional jump back to the start.

`for` loops are decomposed into their equivalent structure: first the initialization is executed, then a start label is established where the condition is evaluated, the body is executed, the increment step is performed, and it jumps back to the start. This decomposition eliminates the need to support specialized `for` instructions in later stages.

Function calls are translated to a sequence of `IRParam` instructions that specify each argument, followed by an `IRCall` instruction that invokes the function and optionally stores the return value in a temporary. Function declarations are delimited with `IRFuncBegin` and `IRFuncEnd` instructions that specify the function name and its parameters.

An important additional component is the handling of string literals. When the generator encounters a string literal in the code, it registers it in a global dictionary that maps the string content to a unique identifier (`str0`, `str1`, etc.). These identifiers are used in IR instructions and subsequently translated to data labels in assembly code.

The IR generator design follows the visitor pattern, where each AST node type has a corresponding visit method. This pattern facilitates extensibility: adding support for new language constructs simply requires implementing a new visit method without modifying the existing infrastructure.

## Assembly Code Generation

The final code generation phase, implemented in the `asm_generator.py` module, translates intermediate code instructions to x86-64 assembly code for Windows architecture. This transformation must respect the operating system's calling conventions, correctly manage processor registers, and generate efficient and correct code.

The assembly code generator sequentially processes IR instructions, emitting the equivalent ASM code for each one. The target architecture is 64-bit x86-64 using NASM (Netwide Assembler) Intel syntax, which allows reasonable portability while maintaining optimal performance.

### Variable and Memory Management

A fundamental aspect is space allocation for variables. The generator maintains a mapping between variable names (including temporaries) and their locations on the stack. Each variable is assigned an offset relative to the stack base register `RBP`. As new variables are encountered, the generator increments the offset and records the association, allowing subsequent generation of correct references like `QWORD [rbp-8]`, `QWORD [rbp-16]`, etc.

### Calling Conventions

The generated code strictly respects the Windows x64 calling convention, which specifies that the first four function arguments are passed in registers `RCX`, `RDX`, `R8`, and `R9`. Additional arguments would be passed on the stack, although the current compiler focuses on functions with up to four parameters. Additionally, the convention requires reserving 32 bytes of "shadow space" on the stack before each call, space that the called function can use to save parameter registers if needed.

### Generated Code Structure

The assembly code is organized into three main sections. The data section (`.data`) contains the string literals identified during IR generation, each with its unique label. The text section (`.text`) contains the executable code, including external function declarations (such as `printf`, `scanf`, and `exit`) and the program's function definitions.

Each function begins with its standard prologue: save the previous `RBP` value on the stack, set `RBP` to the current `RSP` value, and reserve space on the stack for local variables via `sub rsp, N`. At the end of each function, the epilogue restores the stack state: resets `RSP` to its original value by copying it from `RBP`, restores the previous `RBP` value, and returns to the caller with `ret`.

### Instruction Translation

Simple assignment instructions (`IRAssign`) are translated to sequences of `mov` instructions that copy values between registers and memory. Numeric literals are loaded directly, while string references are loaded via `lea` (Load Effective Address) which obtains the address of the corresponding label.

Binary operations (`IRBinOp`) are translated by first loading both operands into registers (`RAX` and `RBX`), executing the appropriate operation (`add`, `sub`, `imul`, `idiv`, etc.), and finally storing the result in the destination location. Comparison operations use the `cmp` instruction followed by `setcc` (set on condition) instructions that set a byte to 0 or 1 according to the comparison result, then extending this value to 64 bits with `movzx`.

Conditional jumps (`IRIfGoto`, `IRIfFalseGoto`) are implemented via `test` instructions to check if a value is zero or non-zero, followed by conditional jumps (`jnz` to jump if not zero, `jz` to jump if zero). Unconditional jumps (`IRGoto`) are directly translated to `jmp` instructions.

Function calls require special treatment. First the shadow space is reserved, then arguments are loaded into appropriate registers (respecting whether they are immediate values, variable references, or string addresses), the `call` instruction is executed, the shadow space is cleaned up by incrementing `RSP`, and finally the return value from `RAX` is saved to the destination if necessary.

### Optimizations and Considerations

The generator implements several basic optimizations. For example, it recognizes numeric literals and loads them directly without needing memory references. It also reuses the `RAX` register for multiple sequential operations when possible, reducing memory traffic.

Register management is deliberately conservative: the generated code primarily uses `RAX`, `RBX`, `RCX`, `RDX`, `R8`, and `R9`, preserving additional registers for possible future extensions. This strategy guarantees correctness although it sacrifices some efficiency in favor of simplicity and maintainability.

## Integration and Execution

The `main.py` module coordinates the complete compiler execution, orchestrating all phases from lexical analysis to final executable generation. This component implements a complete command-line interface that allows controlling the compilation process through various parameters.

The execution flow begins by reading the specified source code file. Subsequently, it sequentially invokes each compiler phase: first lexical analysis that generates tokens, then syntactic-semantic analysis that constructs and validates the AST, followed by intermediate code generation, translation to assembly, and finally assembly and linking to produce the executable.

Each phase reports its execution status with clear messages indicating success or failure. If a phase detects errors, the process immediately stops and displays a descriptive message of the problem, including contextual information when available. This fail-fast strategy allows developers to identify and correct errors in the earliest possible stages.

The system includes options to inspect intermediate representations. The `--show-ir` option displays the generated intermediate code, allowing verification of the AST to IR instruction translation. The `--show-asm` option displays the produced assembly code, useful for understanding the final translation or debugging low-level problems. The `--asm-only` option generates only the assembly file without proceeding to assembly, allowing manual inspection or custom processing.

For the assembly phase, the system automatically invokes NASM (Netwide Assembler) which translates assembly code to object code. If NASM is not installed or not found in the system PATH, the compiler provides clear instructions on how to obtain and install it. Similarly, GCC is used for linking, which combines the object code with system libraries to produce the final executable.

Error handling is comprehensive and contemplates multiple scenarios: non-existent source files, lexical errors from invalid characters, syntactic errors from incorrect grammatical structures, semantic errors from type violations or use of undeclared variables, errors in IR or assembly generation, and failures in the assembly or linking process. In each case, specific information is provided that facilitates problem identification and correction.

The modular architecture of the integration system facilitates future extension. Adding new compilation phases (such as intermediate code optimization or code generation for other architectures) simply requires inserting the new phase into the execution sequence, without needing to restructure the existing control flow.
