# Interface an LCD with a Microcontroller
[NewHaven 2x16 LCD Display](./datasheets/NHD_0216BZ_RN_YBW_Datasheet.pdf), 
Part Number: *NHD-0216BZ-RN-YBW-ND*



Datasheet pinouts are meant to be written in a way to skim quickly and 
gather necessary information. However, due to a couple different reasons,
**the PHYSICAL pinout is not necessarily in the same order as the 
datasheet pinout**

Facing the display, this is the actual ordering of the pinouts

`14|13|12|11|10|9|8|7|6|5|4|3|2|1| 15|16`

|Pin| Symbol | Description                                           | 
|---|--------|-------------------------------------------------------|
| 14| DB7    | MPU, 4 high order D bus lines                         |
| 13| DB6    | MPU, 4 high order D bus lines                         |
| 12| DB5    | MPU, 4 high order D bus lines                         |
| 11| DB4    | MPU, 4 high order D bus lines                         |
| 10| DB3    | MPU, 4 low order D bus lines, *not used in 4-bit mode*|
|  9| DB2    | MPU, 4 low order D bus lines, *not used in 4-bit mode*|
|  8| DB1    | MPU, 4 low order D bus lines, *not used in 4-bit mode*|
|  7| DB0    | MPU, 4 low order D bus lines, *not used in 4-bit mode*|
|  6| E      | MPU, Operation En. Signal, *Falling edge triggered*   |
|  5| R/W    | MPU, Read/Write Select,`1: Read 0: Write`             |
|  4| RS     | MPU, Register Select, `1: Data 0: Command`            |
|  3| V0     | Adj Power Supply, for contrast (~0.6V)                | 
|  2| Vdd    | Power Supply, Voltage (+5V)                           |
|  1| Vss    | Power Supply, Ground                                  |
| 15| Vss    | Do Not Connect                                        |
| 16| Vss    | Do Not Connect                                        |

> MPU just refers to what you connect this pin to, in this case our PSoC microcontroller.\
> V0 is a resistor divider or potentiometer that controls the contrast of the actual display.


## Using PSoC Board to Interface

V0 can be controlled using a voltage divider circuit using vdd from the MPU such as a 10k & 100 ohm resistor setup

**Always use a bypass capacitor between the LCD power pin and Vssd digital ground on the devkit**\
Why? Smooths the input signal in the event of fluctuations and prevents weird behavior/bugs

### 4-Bit Mode

We will be designing our circuit to use 4-bit mode as opposed to 8-bit mode to save output pins
on our MPU/MCU & for simplicity  

- We only need DB7-DB4, which are physical pins 11-14
- R/W (pin 5) can be controlled, or wired to GND (logic 0) 
    - We only really need to write (R/W 0) to the lcd, not read (R/W 1)
- RS?

## Custom Firmware

IDK if the Psoc Creator LCD Component in the components catalog is compatible with 
my display, and I might as well learn, so I will be writing my own drivers, firmware
whatever you want to call it, for the LCD

### Initalize the Display 


- RW -> `0`
- TS- 


