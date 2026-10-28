# Mood Lamp - Bluetooth & Potentiometer - Controlled RGB Lighting

A compact RGB mood lamp built around an **Arduino Nano**, featuring two
control modes
1) manual potentiometer control and wireless Bluetooth control
2) plus an optional pseudo-random flickering effect.

## Overview

The goal of this project was to build an interactive lighting device
that can generate a wide range of colours and ambience settings. The
lamp uses a 3 W high-power SMD RGB LED. Its red, green, and blue
channels are controlled independently, allowing the user to mix colours
and adjust brightness.

A USB Type-C receptacle provides a regulated 5 V DC input. The LED is
mounted on a dedicated zero-PCB and backed by an aluminium heat sink. A
half table-tennis ball acts as a diffuser to soften the light.

## Features

-   **Manual RGB control:** Three panel-mounted potentiometers adjust
    the red, green, and blue channel intensities independently.
-   **Bluetooth control:** An HC-05 module receives RGB values from a
    smartphone using the Serial Bluetooth Terminal app.
-   **Flickering mode:** A software-based 64-bit Linear Feedback Shift
    Register (LFSR) produces a pseudo-random on/off flicker effect.
-   **PWM brightness control:** The Arduino Nano generates 8-bit PWM
    outputs for the three colour channels.
-   **Smoother manual adjustment:** An eight-sample rolling average and
    a small PWM dead zone help reduce input fluctuations and faint
    unwanted illumination.
-   **LED driver stage:** A ULQ2003A Darlington transistor array drives
    the LED channels.
-   **Thermal management:** An aluminium heat sink and thermal paste
    help remove heat from the high-power LED.
-   **Compact enclosure:** The circuit is housed in an IP65-rated
    plastic enclosure, with external controls and a USB Type-C power
    input.

## Hardware Used

  -----------------------------------------------------------------------
  Component                           Purpose
  ----------------------------------- -----------------------------------
  Arduino Nano                        Reads inputs and generates PWM
                                      control signals

  3 W SMD RGB LED                     Main light source; red, green, and
                                      blue channels are controlled
                                      independently

  ULQ2003A driver IC                  Provides current-driving capability
                                      between the controller and LED
                                      channels

  HC-05 Bluetooth module              Wireless serial communication with
                                      a smartphone

  3 × 50 kΩ potentiometers            Manual control of the RGB channel
                                      intensities

  2 × SPDT toggle switches            Select manual/Bluetooth mode and
                                      enable/disable flickering

  USB Type-C receptacle               5 V DC power input

  Aluminium heat sink and thermal     LED thermal management
  paste                               

  Zero-PCBs, resistors, capacitors,   Circuit assembly and
  connectors, and wires               interconnections

  IP65 plastic enclosure              Mechanical protection and housing

  Half table-tennis ball              Light diffuser
  -----------------------------------------------------------------------

## System Architecture

The Arduino Nano is the central controller.

1.  In **manual mode**, the Arduino reads the three potentiometers
    through its analog inputs.
2.  The 10-bit ADC readings (0--1023) are converted to 8-bit PWM values
    (0--255).
3.  An eight-sample rolling average smooths the readings, and a
    dead-zone threshold suppresses very low PWM values.
4.  In **Bluetooth mode**, the HC-05 module receives RGB values over
    serial communication at 9600 baud.
5.  The Arduino applies the selected RGB values to the PWM outputs, and
    the ULQ2003A driver stage controls the LED channels.
6.  When flickering is enabled, the LFSR's pseudo-random output
    determines whether the LED is illuminated or switched off at each
    flicker interval.

## Control Modes

### 1. Manual Mode

Each potentiometer corresponds to one RGB channel:

-   **Red potentiometer:** Adjusts red intensity.
-   **Green potentiometer:** Adjusts green intensity.
-   **Blue potentiometer:** Adjusts blue intensity.

Combining these three channels creates different colours. With 8-bit
values, there are **256 possible input levels per channel**, giving
(256\^3 = 16,777,216) theoretical RGB combinations. This is the number
of possible digital combinations, not the number of colours that are
necessarily distinguishable by eye or achievable at the lamp's physical
output.

### 2. Bluetooth Mode

The lamp can be controlled from a smartphone using the **Serial
Bluetooth Terminal** app and the HC-05 module.

Send the three channel values in this format: R,G,B

Each value should be in the range **0--255**. Configure the terminal app
to append **CR+LF (carriage return and line feed)** to each transmission
so the Arduino firmware can parse the values correctly. The firmware
uses serial communication through the `SoftwareSerial` library at **9600
baud**.

Example RGB command: 255,0,0

This requests maximum red and zero green and blue at the digital control
level. Actual output intensity is limited by the LED driver and the
project's current-limiting resistor choices.

### 3. Flickering Effect

A second toggle switch enables or disables flickering. The firmware
implements a 64-bit LFSR using XOR feedback taps at bit positions 0, 1,
3, and 4. Its least significant bit determines whether the selected RGB
output is on or off, with an approximately 20 ms delay between flicker
states.

The initial LFSR seed is assembled from readings of an unused analog
input. A non-zero fallback seed is used if the generated seed is zero.

## PWM and Thermal Considerations

The manual-mode potentiometer readings are filtered with an eight-sample
rolling average. The software also applies a dead zone to reduce faint
illumination when a control is near its minimum position.

The high-power RGB LED can generate significant heat. The design uses an
aluminium heat sink and thermal paste, and the resistor values were
selected to limit the LED's effective maximum intensity to approximately
**150/255** rather than running it at full PWM intensity. The report
records an overall current requirement of approximately **550 mA** at
full combined RGB intensity under the selected operating condition.

Thermal management remains important when operating high-power LEDs; the
heat sink and current-limiting design should be retained in any
reproduction.

## Mechanical Assembly

The main electronics including the Arduino Nano, HC-05 module,
ULQ2003A driver IC, and resistors are assembled on a main zero-PCB.
JST connectors make external components easier to connect and
disconnect. The RGB LED is mounted on a separate zero-PCB with the
aluminium heat sink secured behind it.

The assembly is installed in an IP65-rated plastic enclosure. A circular
opening lets the light pass through the diffuser, while the
potentiometers, toggle switches, and USB Type-C receptacle are mounted
on the enclosure.

## Firmware

The firmware was developed for the Arduino Nano using the Arduino
development environment. It uses:

-   SoftwareSerial for HC-05 serial communication
-   analogRead() to read potentiometer inputs
-   analogWrite() for PWM-based RGB control
-   An eight-sample rolling average for manual input smoothing
-   A PWM dead zone to suppress very low values
-   A 64-bit LFSR for the pseudo-random flickering effect

The complete firmware can be added to this repository as an .ino file.

## Demonstration

A project demonstration can show:

1.  The completed lamp and its enclosure.
2.  Manual RGB mixing using the three potentiometers.
3.  Switching to Bluetooth mode and sending RGB values from a
    smartphone.
4.  Enabling and disabling the flickering effect.
5.  The USB Type-C power input, LED heat sink, and internal circuit
    assembly.

## Possible Future Improvements

-   Add a dedicated mobile interface with colour selection and
    brightness controls.
-   Improve thermal monitoring and add temperature-based brightness
    limiting.
-   Add more lighting effects, such as smooth fading and colour cycling.
-   Publish a complete wiring diagram, firmware, and assembly
    photographs alongside the README.

## Project Summary

This project combines **embedded control, PWM, RGB colour mixing, UART
serial communication, Bluetooth, and pseudo-random firmware logic** in a
practical electronics build. It provided hands-on experience with
microcontroller interfacing, high-power LED driving, input filtering,
thermal management, and enclosure design.

------------------------------------------------------------------------

**Project:** Mood Lamp\
**Controller:** Arduino Nano\
**Wireless module:** HC-05\
**LED driver:** ULQ2003A\
**Power input:** 5 V DC via USB Type-C
