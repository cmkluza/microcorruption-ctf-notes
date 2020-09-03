## Montevideo - 50 Points

### Notes:
- `main` just calls `login`
- `login`:
    - Stores user input at `0x2400`
    - Copies it to stack
    - `memset`s 0x64 bytes @ `0x2400` to 0
    - Calls `conditional_unlock_door` w/ `sp` as argument
- `conditional_unlock_door`:
    - Calls the HSM-2 interrupt w/ stack password
- The `strcpy` overwrites the return address - this is the same as Whitehorse
    - One catch: can't use 0x00 in input since it uses `strcpy`

### Solution Summary:
- Same process as Whitehorse:
    - Create a string that overwrites the return address to jump to INT
    - And have the string place 0x7F appropriately on the stack

### Solution (hex):
```
112233445566778899112233445566774C4511117F
                                ^^^^    ^^
                    ret addr for INT    INT code to unlock door
```
