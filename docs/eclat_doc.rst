eCLAT documentation
===========================

eCLAT instructions 
------------------------------

Definition of a variable
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    u64 : res = 0

Logical conditions
^^^^^^^^^^^^^^^^^^^^^

eCLAT supports the logical conditions like Python:

.. code-block:: text

    Equals: a == b
    Not Equals: a != b
    Less than: a < b
    Less than or equal to: a <= b
    Greater than: a > b
    Greater than or equal to: a >= b

If statement
^^^^^^^^^^^^^^^^^

The if statement is the same as in Python:

.. code-block:: python

    if var == 1:
        # my comment
        res = 1
    elif var == 1:
        res = 2
    else:
        res = 0

While statement
^^^^^^^^^^^^^^^^^

The while statement is the same as in Python:

.. code-block:: python
    
    var = 1
    while var < 5:
        var = var + 1


eCLAT Data Types
-------------------------------

=============== ==================
eCLAT data type eBPF (C) data type
=============== ==================
   u64            __u64 
   u32            __u32 
   u16            __u16
   u8             __u8
   s64            __s64 
   s32            __s32 
   s16            __s16
   s8             __s8
=============== ==================

eCLAT operators
-------------------------------

Unary Operators 
^^^^^^^^^^^^^^^^^

============ ============================= =========== 
 Operator     Name                          Example    
============ ============================= =========== 
 \-           Negation                      \- x
 not          Logical NOT                   not x      
 ~            Bitwise NOT                   ~ x        
============ ============================= =========== 

Binary Operators 
^^^^^^^^^^^^^^^^^

============ ============================= =========== 
 Operator     Name                          Example    
============ ============================= =========== 
 \+            Addition                      x + y      
 \-            Subtraction                   x - y      
 \*            Multiplication                x * y      
 /            Division                      x / y      
 %            Modulus                       x % y      
 <<           Left bit shift (zero fill)    x << y     
 >>           Right bit shift (zero fill)   x >> y     
 >=           Greater than or equal         x >= y     
 <=           Less then or equal            x <= y     
 >            Greater than                  x > y      
 <            Less than                     x < y      
 ==           Equal                         x == y     
 !=           Not equal                     x != y     
 and          Logical AND                   x and y    
 or           Logical OR                    x or y     
 &            Bitwise AND                   x & y      
 \|            Bitwise OR                    x | y      
 ^            Bitwise XOR                   x ^ y      
============ ============================= =========== 

Formal specs of eCLAT language 
---------------------------------

.. code-block:: text

    program : statement_full | statement_full program

    statement_full : statement NEWLINE | statement

    statement : chain_statement | import_statement | map_statement

    import_statement : FROM NAME DOT NAME IMPORT module_list

    module_list : NAME COMMA module_list | NAME

    map_statement : NAME LSPAR NAME RSPAR ASSIGN kv_mapping

    kv_mapping : LCPAR key_value_pairs RCPAR  | LCPAR NEWLINE INDENT key_value_pairs DEDENT NEWLINE RCPAR

    key_value_pairs : key_value_pair COMMA key_value_pairs
                    | key_value_pair COMMA NEWLINE key_value_pairs
                    | key_value_pair NEWLINE
                    | key_value_pair | EMPTYLINE

    key_value_pair : LPAR exprlist RPAR COLON LPAR exprlist RPAR

    chain_statement : DEF NAME LPAR arglist RPAR COLON NEWLINE block

    block : INDENT block_statements DEDENT

    block_statements: statement_full | statement_full block_statements

    statement : PASS
              | expression
              | IF expression COLON NEWLINE block NEWLINE elif_statement
              | IF expression COLON NEWLINE block NEWLINE else_statement
              | IF expression COLON NEWLINE block NEWLINE
              | WHILE expression COLON NEWLINE block
              | RETURN expression
              | RETURN
              | NAME ASSIGN const
              | NAME ASSIGN expression
              | type COLON NAME ASSIGN const
              | type COLON NAME ASSIGN expression

    else_statement: ELSE COLON NEWLINE block

    elif_statement : ELIF expression COLON NEWLINE block NEWLINE elif_statement
                   | ELIF expression COLON NEWLINE block NEWLINE else_statement
                   | ELIF expression COLON NEWLINE block NEWLINE

    expression : NAME LPAR exprlist RPAR
               | NAME DOT NAME LPAR exprlist RPAR
               | expression PLUS expression
               | expression MINUS expression
               | expression MULT expression
               | expression DIV expression
               | expression MOD expression
               | expression RSHIFT expression
               | expression LSHIFT expression
               | expression GTE expression
               | expression LTE expression
               | expression GT expression
               | expression LT expression
               | expression EQ expression
               | expression NEQ expression
               | expression AND expression
               | expression OR expression
               | expression AMP expression
               | expression PIPE expression
               | expression HAT expression
               | NOT expression
               | MINUS expression
               | TILDE expression
               | LPAR expression RPAR
               | const
               | NAME


    exprlist: expression COMMA exprlist | expression

    arglist : argument COMMA arglist | argument

    argument :  type COLON NAME

    type :  U8 | U16 | U32  | U64  | S8 | S16 | S32 | S64

    const : HEX  | FLOAT  | INTEGER  | STRING | BOOLEAN
