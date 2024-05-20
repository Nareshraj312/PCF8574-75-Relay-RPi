# PCF8574-75-Relay-RPi
 This repository contains a Python script for interfacing PCF8574 and PCF8575 IO expanders (for relay) with a Raspberry Pi 3 using the SMBus library.

## Requirements

* Raspberry Pi 3
* PCF8574 and PCF8575 IO expanders
* Python 3
* Relay module

# Step 1 - Enabling I2C

```
sudo raspi-config
```
This will launch the raspi-config utility

Select "Interfacing Options"  -> Highlight the “I2C” option and activate “Select”  
Select "Yes" when prompted to enable the I2C interface  -> Select "Finish"  
Select "Yes" when prompted to reboot.

<img src="https://github.com/Nareshraj312/PCF8574-75-Relay-RPi/assets/94052859/0709b2ea-7f6b-4d69-9092-bc704f929e8d" width="500" height="300">  <img src="https://github.com/Nareshraj312/PCF8574-75-Relay-RPi/assets/94052859/61751710-8f25-4222-b91a-2af164de4b4a" width="500" height="300"> 



# Step 2 - Installing smbus library
  ```
  sudo apt-get install python3-smbus
  ```
# Step 3 - Wiring

| RPI 3 B+ | PCF8574 | PCF8575 |
| ------------- | ------------- | ------------- |
| 5V  |  VCC  | VCC  |
| GND  | GND  | GND  |
| SDA(GPIO2)  | SDA  | SDA  |
| SCL(GPIO3)  | SCL  | SCL  |


![image](https://github.com/Nareshraj312/PCF8574-75-Relay-RPi/assets/94052859/85150e37-1eaf-4ea1-9ace-29a51778fcd2)


# Step 4 - Testing I2C Communication
If you’ve got a Model A, B Rev 2 or B+ Pi then type the following command :
```
i2cdetect -y 1
```

If you’ve got an original Model B Rev 1 Pi then type the following command :
```
i2cdetect -y 0
```

The Output should look like:

```
pi@raspberrypi ~ $ i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- 21 22 -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

(0X21) - Address of PCF8574

(0X22) - Address of PCF8575


# Step 5 - Code

This code will turn on and off 24 Relays (8+6) one by one.

```
bus = smbus.SMBus(1)
i2c_bus = 1
pcf8574_address = 0x21
pcf8575_address = 0x22 
def set_pin_state_pcf8574(pin_number, state):
    current_state = bus.read_byte(pcf8574_address)
    pin_mask = 1 << pin_number
    if state == 1:
        new_state = current_state | pin_mask
    else:
        new_state = current_state & ~pin_mask
    bus.write_byte(pcf8574_address, new_state)
    
def set_pin_state(pin_number, state):
    current_state = bus.read_word_data(pcf8575_address, 0)
    pin_mask = 1 << pin_number
    if state == 1:
        new_state = current_state | pin_mask
    else:
        new_state = current_state & ~pin_mask
    bus.write_word_data(pcf8575_address, new_state & 0xFF, new_state >> 8)

for i in range(8):
    set_pin_state_pcf8574(i,0)
    time.sleep(0.5)
for i in range(16):
    set_pin_state(i,0)
    time.sleep(0.5)
for i in range(8):
    set_pin_state_pcf8574(i,1)
    time.sleep(0.5)
for i in range(16):
    set_pin_state(i,1)
    time.sleep(0.5)
```

You can use this script to change the state of particular pin - PCF8574.
```
set_pin_state_pcf8574(i,0)  // i is the pin number (0-7), state 0 - HIGH, 1 - LOW
```

You can use this script to change the state of particular pin - PCF8575.
```
set_pin_state(i,0)  // i is the pin number (0-15), state 0 - HIGH, 1 - LOW
```
