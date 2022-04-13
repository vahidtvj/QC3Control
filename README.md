# QC3Control

Set the voltage of a Quick Charge 3.0 source via Arduino.

This is a fork of Vincent Deconinck's QC3Control, customized to work with ESP32 and use less power.

All credits go to :

- [Hugatry's HackVlog](https://www.youtube.com/channel/UCHgeChD442K0ah-KxEg0PHw) because he came up with the idea and first code to control QC2.
- Timo Engelgeer (Septillion) who made a nice wrapper for Quick Charge 2.0: [QC2Control](https://github.com/septillion-git/QC2Control). The QC3Control project is just a fork of QC2Control adapted for Quick Charge3 while retaining maximum compatibility so it can be used as a drop-in replacement.
- [vdeconinck](https://github.com/vdeconinck), the author of the original [QC3Control](https://github.com/vdeconinck/QC3Control)

## What does it do?

QC3Control makes it possible to set the voltage (even on the fly) of a Quick Charge 3.0 source like a mains charger or power bank by simulating the behavior of a QC3 portable device.

Of course, to take advantage of this library, the source needs to support the [Quick Charge 3.0](https://www.qualcomm.com/products/features/quick-charge) technology form [Qualcomm](https://www.qualcomm.com/).
However, the library can also be used with a QC2.0 compatible charger if using only set5V(), set9V(), set12V(), or setVoltage(5), setVoltage(9) or setVoltage(12).

## Class A vs Class B

QC3.0 class A is the most common, is mainly used for phone chargers and outputs up to 12V. It is supported by QC3Control and fully tested. Possible voltages are 5V (USB default), 9V and 12V, plus any value between 3.6V and 12V obtained by 200mV steps from 5V (or from the previous voltage reached using setMilliVoltage()).

QC3.0 class B is more targeted at laptops and can output up to 20V. **WARNING:** this means that the Arduino will be **destroyed** if powered directly from the controlled output voltage, and the use of a separate USB output, power supply or voltage regulator is mandatory. That being said, class B is now supported by QC3Control although testing was limited, and it is enabled by calling begin(true) instead of begin(). Possible voltages are then 5V (USB default), 9V, 12V and 20V, plus any value between 3.6V and 20V obtained by 200mV steps from 5V (or from the previous voltage reached using setMilliVoltage()).

### How to connect?

This fork of the library is designed to work with 3.3v boards. If you want to use it with an Arduino with 5v, use [vdeconinck's design](https://github.com/vdeconinck/QC3Control).

This circuit is very suitable for battery operated and low power designs as no power is wasted when QC is not being used.

![QC3Control circuit](extras/circuit.png)

**_NOTE:_** The new circuit makes use of ESP32's internal pullup resistor ( which is **45KΩ** ). You might need to change resistor values for a different board

### Powering the Arduino

Arduino's Vin recommended voltage is 7-12V (absolute maximum rating 6-20V) while the voltage range supported by QC3Control (particularly in Class B) extends to 3.6-20V (compared to QC2 class A's range of 5-12V).

Depending on your required output voltage range, it may be possible to power the Arduino from the output voltage it is controlling as proposed in the QC2Control project (note though that feeding 5V to Vin is already outside of the Arduino specification). A clever use of diodes or other discrete components may work in your particular case.

Alternately, you can add a buck-boost converter which will be able to power the Arduino based on the full QC3 range, but you will probably end up with a solution generally more expensive and more complex than designing your own variable power supply in the first place.

The recommended alternative is to select a multi-port QC3 charger, to use one QC3 port as your main output and to power the Arduino's 5V pin from another (possibly non-QC) port. In that case, don't forget to connect the GND of both ports together (but **NOT** their VCCs of course).

## Usage

```C++
#include <QC3Control.h>

//Pin 4 for Data+
//Pin 5 for Data-
//See How to connect in the documentation for more details.
QC3Control quickCharge(16, 17);

void setup() {
  //Optional
  //quickCharge.begin();

  //set voltage to 12V
  quickCharge.set12V();

  delay(1000);
}

void loop() {
  //And you can change it on the fly
  delay(1000);
  quickCharge.set9V();
  delay(1000);
  quickCharge.set5V();
  delay(1000);
  quickCharge.setMilliVoltage(6000);
  delay(1000);
  for (int i = 0; i < 10; i++) quickCharge.decrementVoltage();
  delay(1000);
  quickCharge.set12V();
}
```

**Please note**, delay() here is just used to demonstrate. Better not to stop the complete program with delay()'s.

If you can, place the call to begin() (or setMilliVoltage()) at the end of the setup(). The handshake needs a fixed time but that already starts when the QC 3.0 source (and thus the Arduino) is turned on. So by doing begin() last you can do stuff while waiting.

## Quick start

### Constructors

#### QC3Control(byte DpPin, byte DmPin)

[QC2 or QC3 power source] This will create a QC3Control object to control the voltage of the Quick Charge source. DpPin is the pin number for the Data+ side, DmPin is the pin number for the Data- side. See [**How to connect?**](#how-to-connect).

### Methods

#### void .begin()

[QC2 or QC3 power source] Just does the handshake with a Quick Charge 3.0 (class A) source so it will accept commands for different voltages. It's not mandatory to call begin(), if it's not called before setting a voltage the library will call begin() at that moment.

#### void .set5V(), .set9V() and .set12V()

[QC2 or QC3 power source] Sets the desired voltage of the QC source using discrete (QC2) mode.

#### void setMilliVoltage(unsigned int milliVolt)

[QC3 power source only] Sets the desired voltage of the QC3.0 source using continuous (QC3) mode. Setting an unreachable voltage will result in the closest supported voltage.

#### unsigned int getMilliVoltage()

[QC2 or QC3 power source] Return the last voltage that was requested, or the closest one in the range 3.6 - 12V.

**Important:** Note that this method returns the value the library requested, not the actual voltage which may be different if the power source did not behave as expected.

#### void incrementVoltage(), decrementVoltage()

[QC3 power source only] Requests an increment or decrement of the voltage by 0.2V
