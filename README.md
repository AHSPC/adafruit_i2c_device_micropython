# adafruit_i2c_device_micropython

### Modifications:

##### I2CDevice().readinto()
- In that method you will find the following code:
```python
x = buf[start:end]
self.i2c.readfrom_into(self.device_address, x)
buf[start:end] = x
```
- It might look strange
- But
- CircuitPython has their own C code that writes into the buffer in place, while slicing the buffer in python makes a copy
- When we simply passed `buf[start:end]` into readfrom_into, not realizing it made a copy, the adafruit library we were using (TODO add link to code) was hanging in a loop 
- It seems like that was because the CircuitPython C code was writing into the buffer asynchronously while the main python code waited for the write to happen to `buf`, which never did
- So we slice the buf and store the new buf in a variable, then we pass that variable into readfrom_into (now the newly written data is in `x`), and then we write that into the original `buf` variable
- Also, using the other MicroPython I2C functions were giving us a not implemented for I2C device error (didn't investigate why further)
- **Essentially** `self.i2c.readfrom_into(self.device_address, buf[start:end])` doesn't work because `buf[start:end]` makes a copy of `buf`, but we want to write to `buf` in place
