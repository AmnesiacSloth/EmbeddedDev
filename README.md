# Personal Notes and Tracking My Embedded Learning Journey

# Tools used (As of 5/30/25)

## Picoscope 2204A Oscilloscope 

[Software Download Page](https://www.picotech.com/downloads) -  Be sure to select the correct model and download the stable release

**Note:** The model I ordered from Newark did not come with probes, so I ordered 
[RioRand 2-Pack PP150 100 MHz Oscilloscope Clip Probes](https://www.amazon.com/dp/B0CYZ4RQR1?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)
off Amazon

## Cypress CY8CKIT-059 PSoC Microcontroller

Allows for **Analog** and **Digital Circuitry** in the same chip! Referred to as a ***true mixed signal dev kit***


***Chip vs Development Kit Distinction***
- The *development kit* is the entire black electronic board, referred to as the **CY8CKIT-059**
- The *chip* is ON the board and is the large black square near the cypress logo directly under the chip name, which is **CY8C5888LTI-LP097**

The chip is not a *fixed design* meaning you get to wire the analog and digital components as you see fit
- Analog, op amps, comparators, and sample & hold etc
- Digital, timers and counters etc.
> The ***CY8C5888LTI-LP097*** is like an FPGA but with analog circuitry support too 

### PSoC Creator, Development Software
[Software Download Page](https://www.infineon.com/cms/en/design-support/tools/sdk/psoc-software/psoc-creator/) - 
Software can be used without creating an account :)

***Workspace Vs Project***
- Workspace refers to a small file that just points to different projects
- Project contains all the files that are necessary to build your design


#### Using the Software 

LHS gives you a menu of various things such as 
- Schmatics for your project (of which there could be various) 
- Design Wide Resources <!-- TODO: Add More Information Here -->
- Source files (main.c etc etc)
- Header files 

RHS give you a menu of components that you can use in your schematic design, both *on-chip* and *off-chip*
- Analog Components
- Digital Components 
- Filters
- Ports & Pins
- etc etc.

#### Pin Assignment 

Green wire == Digital Pin\
Yellow Orange wire == Analog Pin

**Inputs should always be connected to a signal, never leave them undefined regardless of MCU family**

##### Example of Connecting a Schematic Pin to Physical Pin 

Simple Ex: 
1. Drag and drop a Digital output pin from the *on-chip* component catalog
> Note : The default name is *arbitrary*, it does not necessarily refer to the physical pin on the ***chip***
2. Rename to something that makes sense and use a naming convention 
    - Ex: If I drag and drop an output pin that I intend to connect to `Port 0 bit 3`, I will name is `P0_3`
    - Names cannot use spaces or certain special characters
3. Now that the name reflects what the pin should represent, go to pins tab under *Design Wide Resources* on the 
    LHS and use the pull-down menu to select the corresponding (GPIO) pin
    - Chip Pin shown on the chip image is different from the Port and Bit <!-- TODO: ADD MORE HERE -->

#### Automatically Generated Code

Library of routines that you can call at will from your program

PSoC Creator provides routines such as the CyDelay() function for all projects. <!--TODO: What is this?-->
Other routines are provided "as needed", depending on your schematic contents

##### Key Auto-Generated Counter Functions 

`Start()`: Analogous to a "power-on" function

`writeCounter(uint8 counter)`: Sets counter to the specified unsigned 8-bit integer
- The counter can also be configured under the schematic tab right clicking
    the component and configuring

`ReadCounter()`: Wonder what this does?? 

`WritePeriod(uint8)`: Period refers to the max value/start value when the counter begins,
    - This overwrites the default preconfigured values set in the schematic component


##### Notable Counter Info

- Counter is default configured to count **down**

*There are two types of counters*
- **Fixed Function** counters can only count down
- **UDB** counters offer more flexibility

##### Example with Counter 

1. Add Counter to Schematic
2. Connect Logic Low to Reset
3. Connect a Clock (default is 12MHz but can be changed) to the counter clock pin
4. Upper left of Workspace explorer, click on "generate application" which will examine the schematic
    and create APIs for the placed components 
5. In Workspace Explorer, you wil now see `Generated_Source` folder that holds the C files related to the 
    schematic components
6. `Generated_Source/cy_boot` will contain system wide function calls that are useful but not necessarily related to a 
    specific component on your schematic
    - The delay function will likely be the most useful

**ALL** components (with exception of pins) require that you call their `start()` function. If it is not called, 
your component will literally do nothing.

```c
#include "project.h"

uint8 count1;

int main(void)
{
    CyGlobalIntEnable; /* Enable global interrupts. */

    Counter_1_Start(); /* Initialize counter */
    for(;;)
    {
        count1 = Counter_1_ReadCounter();
    }
}
```

*Why do we initialize count1 outside of main?* 
- This makes count1 a **static** variable which comes with many desirable traits 
    - Static variables are assigned a permanent place in RAM
    - These variables are always visible to the debugger


# Techniques 

## Debugging Tools 

### Built-In Debugger

Allows developer to start and stop code execution

- When stopped, we can *read* values of the local and global variables
- When stopped, you can *alter* the values of local and global variables

**NOTE:** Does not allow for inspection/alterations while the code is executing

#### Stopping and Analyzing is not always feasible 

- Motion control systems cannot be stopped, otherwise the machine will run away

### Oscilloscope 

We tackle real-time inspection of embedded systems using an Oscilloscope

1. Typically done by writing a `0` or `1` to an output pin
2. Connecting that pin to an oscilloscope 
3. Compare that to when other inputs change giving vital timing readings to further debug

Or

1. Write a pulse to an output pin
    - The width of the pulse indicates where you are in the code
```c
Test_Point_P3_2_Write(1); // Timing marker
CyDelayUs(1); // 1 Microsecond delay
Test_Point_P3_2_Write(0);
```

Another cool technique is using a *Digital-to-Analog Converter* on the chip for 
debugging purposes

1. Set up a DAC to output to any available pin
2. Feed in the data you want to inspect in real time into the DAC
    > DAC input *must* be a 8bit unsigned number (like char or uint8)
3. If data you want to read is something like a uint32 or int16
    - Normalize the data point before feeding it into the DAC
```c
// Example 
int32 noise_level; // This is the value we want to read on our oscope
/* If we know that we do not expect values to be as large as 2^31 
and we know that the largest value,
when divided by 16 is <= 255
we can downsample, cast to uint8 for the DAC, and feed it in
*/
DAC_SetValue( (uint8) ( noise_level/ 16 ) );
// Every time the above line is hit, it will be read on the oscilloscope
```
4. Connect that pin to your oscilloscope to read the outputs in ***real-time***

*Why not just use an LCD?*
- There may be some data we want to measure in real-time, and 
    perhaps an LCD will not be adequate at updating as fast as we require

### LCD Display

Works best for slow changing variables, or displaying text messages
> Usually pretty hard for a regular person to read an LCD display at a rate faster than 2x/sec

### Digital Logic Analyzer
