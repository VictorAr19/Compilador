# Compilador

he user_lexer.py mo
compiler/
├── main.py                 # Main compiler driver
├── lexer/
│   ├── __init__.py
│   ├── user_lexer.py       # Core lexical analyzer
│   └── adapter_lexer.py    # Token adaptation layer
├── parser.py               # Parser and semantic analyzer
├── ir_generator.py         # IR code generation
├── asm_generator.py        # Assembly code generation
├── README.md               # This file
└── .gitignore
