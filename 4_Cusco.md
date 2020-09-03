## Cusco - 25 Points

### Solution Summary:
- User input is pushed to stack without proper bounds checking
- Able to overwrite return address, so when `login` returns it jumps to `unlock_door`

### Solution (hex):
```
001122334455667788990011223344554644
                                ^^^^
                           return address 0x4446
```
