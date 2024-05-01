### Overview
Interpreted language, but support more general data types hence able to tackle a larger domain of problems than Shell/batch/Awk/Perl.

### Python Interpreter
- Argument Passing
    ```
    import sys

    sys.argv[0]
    ```

- Source code encoding: default as UTF-8, can be customized via 
    ```
    # -*- coding: encoding -*-
    ```

### Key Concepts
<https://docs.python.org/3/tutorial/introduction.html>
- string, ' and " are interchangable, except for a string quoted with ", raw " needs to be escaped by \. Multi-line strings can use triple """ or '''
- list - muteble, shallow copy
- Flow control tools
    - if ... elif... else
    - for 
    - [range()](https://docs.python.org/3/library/stdtypes.html#range)
    - break
    - continue
    - pass - can be used as place holder
    - match
    - define functions
    - [function arguments, lamda expressions](https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions)
- Data Structure
    - list (a = [1,2,3] two ways to delete an item .pop(), del x)
    - tuple (immutable)
    - set {'apple', 'orange'}, emptyset = set()
    - dictionary 
    - loop
    ```
     for k, v in somedict.items():
     for i, v in enumerate(['a','b','c']):
    ```
    - conditions: not, and, or 
    - sequence comparison (lexicographical ordering)

- Modules
    - The file name is the module name with the suffix .py appended.
    - Within a module, the module’s name (as a string) is available as the value of the global variable __name__.
    - Execute module as script 
    ```
    if __name__ == "__main__":
    ```
    - [packages and imports](https://docs.python.org/3/tutorial/modules.html#packages)
    
- Input and Output
    - string formatting
    - read and write files