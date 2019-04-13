---
date: "2019-04-13T08:33:40+02:00"
lastmod: "2019-04-13T17:05:54+02:00"
title: "Porting to a new keyboard"
type: post
tags: [guide, programming]
---

The goal of this guide is to show you how to add support for a new keyboard to
[Kaleidoscope][k]. It doesn't go further than that: this guide doesn't help you
maintain an hardware plugin in an external repository, nor does it document
everything that makes for a good pull request for a keyboard. It goes far enough
to have it working, in a quick and dirty way, anything further than that will be
the topic of another guide.

 [k]: https://github.com/keyboardio/Kaleidoscope

To make things simple, this guide is focused on a narrower set of keyboards:
those powered by an `ATMega32U4` controller, which do not use I2C or anything
similar.

## Architectural overview

## The board we'll build a plugin for

TODO (4x5 numpad)

## Creating the hardware plugin

### `src/kaleidoscope/hardware/example/NumPad.h`

#### Binding the header to the hardware

{{< code numbered="true" >}}
[[[#pragma once]]]

#ifdef [[[ARDUINO_AVR_NUMPAD]]]
{{< /code >}}

1. TODO
2. TODO

#### Setting the stage

{{< code numbered="true" >}}
#include <Arduino.h>
#define [[[HARDWARE_IMPLEMENTATION]]]           \
  [[[kaleidoscope::hardware::example::NumPad]]]
#include "Kaleidoscope-HIDAdaptor-KeyboardioHID.h"

#include "kaleidoscope/macro_helpers.h"
#include [[["kaleidoscope/hardware/avr/pins_and_ports.h"]]]

#include [[["kaleidoscope/hardware/ATMegaKeyboard.h"]]]
#include [[["kaleidoscope/driver/bootloader/avr/Caterina.h"]]]
{{< /code >}}

 1. Some parts of the firmware need to know the class that implements the hardware it is being compiled for. To make this simple, we set the `HARDWARE_IMPLEMENTATION` macro to the class name of our hardware plugin.
 2. As explained before, hardware plugins live in a vendor namespace (`example` in our case) under `kaleidoscope::hardware`, and the class is named after the board (`NumPad`).
 3. The `avr/pins_and_ports.h` header contains definitions for the PIN and Port macros common to all AVR devices. We'll be using these (the pins) to set up the key matrix for our keyboard.
 4. The `ATMegaKeyboard.h` header implements a helper base-class, which abstracts away most of the boring parts of hardware support, at least for keyboards based on the `ATMega32U4` MCU.
 5. We also include a header for the bootloader used on the MCU. For this example, we'll be using Caterina, but `HalfKay` (Teensy) and `FLIP` are also supported. The bootloader is used to allow restarting the keyboard in programmable mode, from within the firmware.

#### The hardware class itself

{{< code numbered="true" >}}
namespace kaleidoscope {
namespace hardware {
namespace example {
class NumPad: public [[[ATMegaKeyboard<NumPad>]]] {
  friend class ATMegaKeyboard<NumPad>;
 public:
  NumPad(void) {
    mcu_.disableJTAG();
  }

  [[[ATMEGA_KEYBOARD_CONFIG]]](
    [[[ROW_PIN_LIST]]]({ PIN_D0, PIN_D1, PIN_D2, PIN_D3 }),
    [[[COL_PIN_LIST]]]({ PIN_F0, PIN_F1, PIN_F4, PIN_F5, PIN_F6 })
  );

  static constexpr int8_t [[[led_count = 0]]];

 protected:
  driver::bootloader::avr::Caterina [[[bootloader_]]];
};

}
}
}
{{< /code >}}

1. TODO
2. TODO
3. TODO
4. TODO
5. TODO
6. TODO

#### Keymaps and other helpers

{{< code numbered="true" >}}
#define KEYMAP(                                                                    \
      r0c0 ,r0c1 ,r0c2 ,r0c3 ,r0c4 ,r0c5   ,r0c6 ,r0c7 ,r0c8 ,r0c9 ,r0c10 ,r0c11   \
     ,r1c0 ,r1c1 ,r1c2 ,r1c3 ,r1c4 ,r1c5   ,r1c6 ,r1c7 ,r1c8 ,r1c9 ,r1c10 ,r1c11   \
     ,r2c0 ,r2c1 ,r2c2 ,r2c3 ,r2c4 ,r2c5   ,r2c6 ,r2c7 ,r2c8 ,r2c9 ,r2c10 ,r2c11   \
                             ,r3c4 ,r3c5   ,r3c6 ,r3c7                             \
  )                                                                                \
  {                                                                                \
    { r0c0 ,r0c1 ,r0c2 ,r0c3 ,r0c4 ,r0c5 , r0c6 ,r0c7 ,r0c8 ,r0c9 ,r0c10 ,r0c11 }, \
    { r1c0 ,r1c1 ,r1c2 ,r1c3 ,r1c4 ,r1c5 , r1c6 ,r1c7 ,r1c8 ,r1c9 ,r1c10 ,r1c11 }, \
    { r2c0 ,r2c1 ,r2c2 ,r2c3 ,r2c4 ,r2c5 , r2c6 ,r2c7 ,r2c8 ,r2c9 ,r2c10 ,r2c11 }, \
    { XXX  ,XXX  ,XXX  ,XXX  ,r3c4 ,r3c5  ,r3c6 ,r3c7 ,XXX  ,XXX  ,XXX   ,XXX   }  \
  }

#define KEYMAP_STACKED(                                                            \
      r0c0 ,r0c1 ,r0c2 ,r0c3 ,r0c4  ,r0c5                                          \
     ,r1c0 ,r1c1 ,r1c2 ,r1c3 ,r1c4  ,r1c5                                          \
     ,r2c0 ,r2c1 ,r2c2 ,r2c3 ,r2c4  ,r2c5                                          \
                             ,r3c4  ,r3c5                                          \
                                                                                   \
     ,r0c6 ,r0c7 ,r0c8 ,r0c9 ,r0c10 ,r0c11                                         \
     ,r1c6 ,r1c7 ,r1c8 ,r1c9 ,r1c10 ,r1c11                                         \
     ,r2c6 ,r2c7 ,r2c8 ,r2c9 ,r2c10 ,r2c11                                         \
     ,r3c6 ,r3c7                                                                   \
  )                                                                                \
  {                                                                                \
    { r0c0 ,r0c1 ,r0c2 ,r0c3 ,r0c4 ,r0c5 , r0c6 ,r0c7 ,r0c8 ,r0c9 ,r0c10 ,r0c11 }, \
    { r1c0 ,r1c1 ,r1c2 ,r1c3 ,r1c4 ,r1c5 , r1c6 ,r1c7 ,r1c8 ,r1c9 ,r1c10 ,r1c11 }, \
    { r2c0 ,r2c1 ,r2c2 ,r2c3 ,r2c4 ,r2c5 , r2c6 ,r2c7 ,r2c8 ,r2c9 ,r2c10 ,r2c11 }, \
    { XXX  ,XXX  ,XXX  ,XXX  ,r3c4 ,r3c5  ,r3c6 ,r3c7 ,XXX  ,XXX  ,XXX   ,XXX   }  \
  }

#include "kaleidoscope/hardware/key_indexes.h"

extern kaleidoscope::hardware::example::NumPad &NumPad;
{{< /code >}}

#### Wrapping up

{{< code numbered="true" >}}
#endif
{{< /code >}}

## Making the board available to Arduino

TODO
