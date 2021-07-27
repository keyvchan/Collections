# Work With Ghidra Script API

Ghidra is a tools made by NSA, which specifically used for reverse engineering. It has a lot of benefits and nearly equal functionally compare to other tools such as IDA Pro and Binary Ninja.
It use a special medium level intermedium language(MLIL) called `P-code`. P-code can be treated as a more detailed assembly, it contains all information to be used in a analysis.

P-code has some core concepts hard to understand and it is not well documented. This post can be used as a reference of these core concepts.
To demonstrated it, here is a example:

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    int main(int argc, char **argv) {
    	int data = 0;
    	int result = 0;

    	if (argc > 2) {
    		data = 10;
    	}

    	result = 100 / data;

    	printf("Result: %d\n", result);
    }
    ```

The code above is fairly simple, it takes `argc` into account print result when `argc > 2` fault instead. And here is the corresponding P-code:

    ```
    (ram, 0x100003f36, 24, 0)
    	(stack, 0xffffffffffffffe4, 4) COPY (const, 0x0, 4)
    (ram, 0x100003f48, 42, 1)
    	(unique, 0xce80, 1) INT_SLESS (register, 0x38, 4) , (const, 0x3, 4)
    (ram, 0x100003f48, 43, 2)
    	 ---  CBRANCH (ram, 0x100003f55, 1) , (unique, 0xce80, 1)
    (ram, 0x100003f4e, 46, 0)
    	(stack, 0xffffffffffffffe4, 4) COPY (const, 0xa, 4)
    (ram, 0x100003f55, 148, 0)
    	(stack, 0xffffffffffffffe4, 4) MULTIEQUAL (stack, 0xffffffffffffffe4, 4) , (stack, 0xffffffffffffffe4, 4)
    (ram, 0x100003f5b, 53, 1)
    	(unique, 0x34080, 8) INT_SEXT (stack, 0xffffffffffffffe4, 4)
    (ram, 0x100003f5b, 58, 2)
    	(unique, 0x34400, 8) INT_DIV (const, 0x64, 8) , (unique, 0x34080, 8)
    (ram, 0x100003f5b, 61, 3)
    	(unique, 0x34580, 8) INT_REM (const, 0x64, 8) , (unique, 0x34080, 8)
    (am, 0x100003f6d, 75, 4)
    	 ---  CALL (ram, 0x100003f7c, 8) , (const, 0x100003f9e, 8) , (unique, 0x34400, 8) , (unique, 0x34580, 8)
    (ram, 0x100003f72, 154, 5)
    	(register, 0x0, 4) COPY (const, 0x0, 4)
    (ram, 0x100003f7a, 93, 6)
    	 ---  RETURN (const, 0x0, 8) , (register, 0x0, 4)r
    ```

Every address in memory following its operations. It just a quick preview of what is P-code, I will explain it in a bit.

## Varnode

P-code use `Varnode` as minimum unit of operation. Take P-code above as a example. The format as follow are called a `Varnode`:

    ```
    # (Address space, offset, Size)
    (ram, 0x100003f7c, 8)
    (stack, 0xffffffffffffffe4, 4)
    (unique, 0x34080, 8)
    (const, 0x2, 4)
    (register, 0x38, 4)

    ```

As shown above, a `Varnode` use a format as `(Address Space, Offset, Size)`. The `Address Space` usually have five types, respectively `ram`, `stack`, `unique`, `const` and `register`.

For `ram`, there are predefined functions and variables placed in it. Also if code use control flow such as if, it may probably use this type of `Varnode` as their `goto` destination.
A imported function may also use this type of `Varnode` to be called by other statements.

For `stack`, it usually a variable defined in code.

For `unique`, it use as a temprarely storage.

For `const`, it represented a value which can be used to add or sub.

For `register`, it reference to value stored in registers.

## Operation

Format as following:

    ```
    output OPERATE input0, input1, input2
    ```
