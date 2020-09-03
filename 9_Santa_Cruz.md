## Santa Cruz - 50 Points

### Notes:
- There's an `unlock_door` function; probably overwrite a return value on the stack somewhere
- Username and password stored away from stack pointer
- Username and password then `strcpy`'d onto stack, allowing buffer overflow?
- After `strcpy`, r14 stores pointer to end of password
- Hard stops the program if pwd length isn't in [8, 16]
- Doesn't seem to check username; can break it with username
    - Does check username?
    - There's a check where a value is loaded from memory that was overwritten with username
    - Value loaded from `0x43b3`; hard-coded?
- Password is copied after the username - it can overwrite it on the stack
    - Password is length checked, but username is not?
    - There are certain bytes at hard-coded memory addresses checked at various
      parts of the program; if checks fail, program jumps to terminate
    - With the right sequence of checks, can reach the end of the program and hit a `ret`
    - One byte in the middle of username needs to be 00 for a check; but username can't
      include 00 since it's `strcyp`'d
    - So write the username w/o 00s and have it overwrite the return address on the stack
    - Then have the password overwrite the appropriate byte, set it 00 so it passes the check
      and the program reaches the "ret" and uses the injected return address

### Solution and Summary:
#### Username (hex):
```
0111223344556677889901112233445566057788990111223344556677889933445566770199334455664A44
                                  ^^^^                                  ^^          ^^^^
                                   0 1                                   2            3
    - 0: Overwrites the password min length (stored on the stack). Just needs to be lower than password length
    - 1: Overwrites the password max length (stored on the stack), allowing longer password to pass
    - 2: Gets replaced by 00 in password, so a check passes and the program jumps to the ret section
    - 3: The new return address (0x444A, open door)
```

#### Password (hex):
```
303132333435363738393031323435363700
                                  ^^
        Overwrites 01 in username so it passes a final check
```
