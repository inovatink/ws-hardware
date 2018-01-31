# Bootloader and Initial Setup

## Necessary Tools

- ARM Cortex JTAG (ATMEL-ICE, SAM-ICE or equivalent )

## Procedure

Below procedure explains how to program the bootloader into the wearable sensor board and how to change variant.cpp, variant.h and boards.txt files in order to have wearable sensor working with Arduino enviroment.

> If you already have board with Arduino bootloader you can skip step 1.

> This repo  includes the changed boards.txt, variant.cpp and varian.h files. This files were changed for Arduino version 1.8.5 (Windows) and Arduino SAMD Boards package version 1.6.17. If you have different version of Arduino and SAMD board package it is recommended that you go through procedure given below. Otherwise just download these files and replace the original ones in the corresponding folders and you will be all set.

1. Flash Board with bootloader circuitplay_m0_samd21g18_sam_ba.bin from https://github.com/arduino/ArduinoCore-samd/tree/master/bootloaders/circuitplay (also found in this repo)
2. Install latest Arduino IDE 
3. Install latest SAMD board package (Boards Manager)
4. In AppData\Local\Arduino15\packages\arduino\hardware\samd\1.6.xx folder in boards.txt file under the section named # Arduino/Genuino Zero (Native USB Port) insert -DCRYSTALLESS flag in; 	
```cpp 
arduino_zero_native.build.extra_flags=-D__SAMD21G18A__ {build.usb_flags}
```
This operation shoud result in line like shown below;

```cpp
arduino_zero_native.build.extra_flags= -DCRYSTALLESS -D__SAMD21G18A__ {build.usb_flags}
```

5. In variant.cpp from \packages\arduino\hardware\samd\1.6.15\variants\arduino_zero change pin description in PinDescription g_APinDescription

- Comment out:
```cpp
{ PORTA,  9, PIO_TIMER, (PIN_ATTR_DIGITAL|PIN_ATTR_PWM|PIN_ATTR_TIMER), No_ADC_Channel, PWM0_CH1, TCC0_CH1, EXTERNAL_INT_9 },]
{ PORTA,  8, PIO_TIMER, (PIN_ATTR_DIGITAL|PIN_ATTR_PWM|PIN_ATTR_TIMER), No_ADC_Channel, PWM0_CH0, TCC0_CH0, EXTERNAL_INT_NMI },
```

- Replace with
```cpp
{ PORTA,  9, PIO_SERCOM_ALT, (PIN_ATTR_DIGITAL), No_ADC_Channel,NOT_ON_PWM, NOT_ON_TIMER, EXTERNAL_INT_9 },
{ PORTA,  8, PIO_SERCOM_ALT, (PIN_ATTR_DIGITAL), No_ADC_Channel, NOT_ON_PWM, NOT_ON_TIMER, EXTERNAL_INT_NMI },
```
- Add following code at the end of variant.cpp
```cpp
Uart Serial2( &sercom2, PIN_SERIAL2_RX, PIN_SERIAL2_TX, PAD_SERIAL2_RX, PAD_SERIAL2_TX);		

void SERCOM2_Handler(){
	Serial2.IrqHandler();
	}
```

6. In variant.h under the same folder add following code in the SERCOM DEFINITION
```cpp
extern Uart Serial2;    // Serial2 definition
```

- Also add serial2 pin definition as shown below

```cpp	
#define PIN_SERIAL2_RX       (3ul)                 // Pin description number for PIO_SERCOM on D3
#define PIN_SERIAL2_TX       (4ul)                 // Pin description number for PIO_SERCOM on D4
#define PAD_SERIAL2_TX       (UART_TX_PAD_0)       // SERCOM pad 0
#define PAD_SERIAL2_RX       (SERCOM_RX_PAD_1)     // SERCOM pad 1`
```
