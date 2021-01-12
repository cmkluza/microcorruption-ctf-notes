## Jakarta - 40 Points

### Notes:
- There is an `unlock_door` function... ROP
- `login` is main function:
    - Prints some stuff
    - Username:
        - User input bounded by 0xFF
        - Does manual strlen, uncoditionally copies onto the stack, and then checks length < 0x21 (if not, program exits)
    - Password:
        - User input bounded by 0x1FF (longer than username, but not long enough to hit anything interesting?)
        - User input is at same address username was entered before being copied to the stack
        - Does manual strlen, unconditionally copies to the stack, then checks length < 0x21 
- There's a string at 0x4638 that isn't placed in the .strings section
    - This address 0x4638 is encoded at 0x2400
    - String input starts at 0x2402, immediately after this address..
- Test username and password valid:
    - Function starts at 0x4458 (definitely not reachable by username + password length)
    - Just does the password test interrupt w/ username + password concatenated on the stack
- If length checks fail, cannot do ROP because it jumps to another function to end the program
    - But if we get past those and the password is incorrect, it'll return from the function... can do ROP there?
    - Stack return address is at: 0x4016, user input starts at: 0x3FF2, that's 0x24 bytes apart
        - Can reach this with username + password length?
- I misread the password input assembly... 0x1FF is ANDed with the password
    - So the password input is limited to `(0x1F - strlen(username)) & 0x1FF`
    - But `strlen(username) == 0x20` is valid... so you can underflow this subtraction and end up with 0x1FF as the password limit
    - The password is still `strcpy`'d after the username so we can get 0x20 + 0x20 = 0x40 bytes onto the stack, enough to reach the return address at 0x4016
    - With a 32 byte username, the 5th/6th bytes of password should be the return address
    - But the second length check uses sum of username length and password length... inputting anything for the password exceeds this
    - Can we trick the second length check?
        - Length check sequence:
            - strlen(username) in r11
            - strlen(password) in r15
            - Lengths summed: r15 = r15 + r14
            - Is overflow not possible?
                - Does `cmp.b` actually just compare a single byte? If the 16-bit register has 0xXX00 will it just use the 0x00 in `cmp.b`?
                - IT DOES! So we can overflow the length with a long password and get past the length check to the function return

### Solution and Summary:
#### Username (hex):
```
757365726E616D65757365726E616D65757365726E616D65757365726E616D65
(any 32-byte long string; this is "usernameusername..." in ASCII hex)
```

#### Password (hex):
```
Relevant snippet:
303030304C44
        ^^^^
        return address
Full:
303030304C443030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030
(After the "relevant snippet" therer just needs to be enough bytes that the string length addition overflows; this is the minimal amount of bytes to achieve this)
```
