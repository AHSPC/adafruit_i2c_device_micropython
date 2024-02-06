# adafruit_i2c_device_micropython_port

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
- CircuitPython has their own C code that writes into the buffer in place, while slicing the buf in python makes a copy
- The code for adafruit's RGB color sensor (TODO add which sensor and link to code) was hanging at a while loop when we simply passed `buf[start:end]` into readfrom_into, not knowing it just made a copy
- It seems like that was because the C code was writing into the buffer asynchronously while the main python code waited for the write to happen forever
- So we slice the buf and store the new buf in a variable, then we pass that variable into readfrom_into (now the newly written data is in `x`), and then we write that into the original `buf` variable
