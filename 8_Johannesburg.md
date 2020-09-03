## Johannesburg - 20 Points

### Notes:
- `main` just calls `login`
- There is an unconditional `unlock_door` function
- Looks like "bounds checking" is just checking if the 17th byte is 0x28
- So the same usual return address overwrite approach is still applicable

### Solution Summary:
- Overwrite the return address to jump from `login` to `unlock_door`

### Solution (hex):
```
1122334455667788991122334455667728284644
                                ^^  ^^^^
                  For bounds check  Address of unlock_door  
```
