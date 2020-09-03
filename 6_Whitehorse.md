## Whitehorse - 50 Points

### Notes:
- Overwriting stack pushed value to `0x7F` instead of `0x7E` unlocks the door
- Stack overflow unlikely - area of stack written is very far from area of mem where program is
- Appears that the return address for login function can be overwritten
- Can set return address to jump to `INT` at `0x4532` -> `0x3245` (endian fix)
    - Need to also set stack so `0x7F` is on there at appropriate place
    - First step - how many leading bytes to get to ret addr:
        - Ret Addr at `0x3ee2`
        - Stores starting at `0x3ed2`
        - So, need 16 - prefix: `0123456789012345`
        - Add ret addr of 3245: `01234567890123452E`
        - And set sp+2 to 0x7F (change to hex input): `00112233445566778899001122334455324500007F`

### Solution Summary:
- `main` calls `login`, storing the return address on the stack
- `login` reads user input without proper limiting, and it can overwrite the return address
- Overwriting the return address allows you to jump into the `INT` routine
- `INT` checks for an opcode on the stack - `0x7F` is for unlocking the door
- Overwriting the return address and storing the opcode on the stack results in unlocking the door

### Solution (hex):
```
00112233445566778899001122334455324500007F
                                ^^^^    ^^
                    ret addr for INT    INT code to unlock door
```
