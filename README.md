# SmartGC
A smart garbage collector for C.

SmartGC is not exactly a garbage collector, the one main difference is that it does not 
run during the runtime of the code.

SmartGC pre-emptively(statically) decides when and where to clear up the memory in a c program.

There are 2 main advantages that SmartGC has over a dedicated GC Engine

1. It does not cause any overhead, so runtime speeds are not affected at all
2. It does not require the user to change any existing code, so it can work on any native C program

## How does Smart GC work?

1. First using a python parser, we parse the ast to determine all the references to any identifier,
   determine if it was allocated dynamically, and determines the last reference to it in the program
   and returns the line number. It parses through the entire program as such and returns references, 
   and its line number in the form of a json.
2. Then another python script parses the json, determines where to free the memory and injects a free();
   statement into the existing user code and returns a compilable c program that is now a memory safe 
   code.

## What are future prospects regarding this?

1. Determination of a dynamically allocated pointer is a bit flawed and can cause issues, this 
   will be resolved in future releases.
2. As of the current version the references are stored as the variable name itself, which means that 
   there can be issues regarding the same variable name being used in different scopes. This can be fixed 
   by hashing the references to avoid clashes.
3. Storing the line no might not be safe as multiple lines of code can be written in a single line, hence we 
   we need to determine at which exact location the memory was freed and then clear it.
4. Edge cases have not yet been tested, references used inside loops have been handled, but there are other 
   cases, such as multiple file structures
   that share memory have to be handled in the future.

## How to use SmartGC? 
(assuming that the git repo has been pulled)
	
First libclang dll needs to be installed

(FOR LINUX BASED SYSTEMS ONLY)

1. ` python3 -m venv venv && source venv/bin/activate `
    
    This will create a virtual environment.


2. ` sudo apt install libclang-14-dev `

    (As of 15/10/2024 this should install version 14 of the libclang dll required to run it)

3. ` pip install clang==14 `

    This will install clang version 14 for python.


Now that the setup is ready, you can run the following command to use SmartGC

` python3 ParseInject.py UserCode.c OutCode.c references.json `

This will (if it does not already exist), create a OutCode.c that has been injected with memory free statements
and can be compiled and used at will. 

## A few things to take care of:

In your c program there are a few things that you need to take care of:

Make sure the allocation statements are in this exact format:
  `int *x = (int *)malloc(sizeof(int));`

Ofcourse the types and the variable names can be different but make sure the whitespaces match 
the above format.

Another very important thing to make sure is that when you allocate memory, the allocated space 
should be used, as in if you have 

```
int *x = (int *)malloc(sizeof(int));
*x = 10;
```

make sure you have used the variable somewhere else in the code as well, (if you donot, the parser
will not accept the allocation and reject it since its not been used and hence it is free by default).

If in case you close your workspace (terminal/vscode/etc) you will need to run

`source venv/bin/activate`

before you run the python commands.

