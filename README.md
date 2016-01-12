![SwiftyGPIO](https://github.com/uraimo/SwiftyGPIO/raw/master/logo.png)

**A Swift library to interact with Linux GPIOs, turn on your leds and more!**

<p>
<img src="https://img.shields.io/badge/os-linux-green.svg?style=flat" alt="Linux-only" />
<a href="https://developer.apple.com/swift"><img src="https://img.shields.io/badge/swift2-compatible-4BC51D.svg?style=flat" alt="Swift 2 compatible" /></a>
<a href="https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/LICENSE"><img src="http://img.shields.io/badge/license-MIT-blue.svg?style=flat" alt="License: MIT" /></a>
</p>

## Summary

This library provides an easy way to interact with digital GPIOs using Swift on Linux. You'll be able to configure a port attributes (direction,edge,active low) and read/write the current value.

It's built to run **esclusively on Linux ARM Boards** (RaspberryPi, BeagleBone Black, UDOO, Tegra, CHIP, etc...) with accessible GPIOs.

**Do you own an unsupported/untested board and would like to help? Check out issues  [#3](https://github.com/uraimo/SwiftyGPIO/issues/3) and [#4](https://github.com/uraimo/SwiftyGPIO/issues/4)**

## Supported Boards

Tested:
* C.H.I.P.

Not tested but should work:
* Raspberry Pi A,B Revision 1
* Raspberry Pi A,B Revision 2
* Raspberry Pi B+, Pi 2, Pi Zero
* BeagleBone Black
                     
## Installation

To use this library, you'll need a Linux ARM board with Swift 2.

You can either compile Swift yourself following [these instructions](http://www.housedillon.com/?p=2267) or use precompiled binaries following one of guides from [@hpux735](http://www.housedillon.com/?p=2293) or [@iachievedit](http://dev.iachieved.it/iachievedit/open-source-swift-on-raspberry-pi-2/) if you have a Raspberry Pi 2 or a C.H.I.P..

Once done, considering that at the moment the package manager is not available on ARM, you'll need to manually download Sources/SwiftyGPIO.swift: 

    wget https://raw.githubusercontent.com/uraimo/SwiftyGPIO/master/Sources/SwiftyGPIO.swift
    
(For sample projects that uses the package manager retrieving SwiftyGPIO from GitHub check the **Examples** directory)

Once downloaded, in the same directory create an additional file that will contain the code of your application (e.g. main.swift). 

When your code is ready, compile it with:

    swiftc SwiftyGPIO.swift main.swift

The compiler will create a **main** executable.

## Usage

Let's suppose we are using a CHIP board and have a led connected between the GPIO pin P0 and GND and we want to turn it on.

First, we need to retrieve the list of GPIOs available on the board and get a reference to the one we want to modify:

    let gpios = SwiftyGPIO.getGPIOsForBoard(.CHIP)
    var gp = gpios[.P0]!

The following are the possible values for the supported boards:
    
* .RaspberryPiRev1 (Pi A,B Revision 1)
* .RaspberryPiRev2 (Pi A,B Revision 2) 
* .RaspberryPiB2Zero (Pi B+,2,Zero with 40 pin header)
* .CHIP (the $9 C.H.I.P. computer).

The map returned by *getGPIOsForBoard* contains all the GPIOs of a specific board as described by [these diagrams](https://github.com/uraimo/SwiftyGPIO/wiki/GPIO-Pinout). 

Alternatively, if our board is not supported, each single GPIO object can be instantiated manually, using its SysFS GPIO Id:

    var gp = GPIO(name: "P0",id: 408)  // User defined name and GPIO Id
    
The next step is configuring the port direction, that can be either *GPIODirection.IN* or *GPIODirection.OUT*, in this case we'll choose .OUT:

    gp.direction = .OUT

Then we'll change the pin value to the HIGH value "1":
	
    gp.value = 1

That's it, the led will turn on.

Now, suppose we have a switch connected to P0 instead, to read the value coming in the P0 port, the direction must be configured as *.IN* and the value can be read from the *value* property:

    gp.direction = .IN
    let current = gp.value

The other properties available on the GPIO object (edge,active low) refer to the additional attributes of the GPIO that can be configured but you will not need them most of the times. For a detailed description refer to the [kernel documentation](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt)

## Under the hood

SwiftyGPIO interact with GPIOs through the sysfs file-based interface described [here](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt).

The GPIO is exported the first time one of the GPIO methods is invoked, using the GPIO id provided during the creation of the object (either provided manually or from the defaults). Most of the times that is will be different from the physical id of the pin. SysFS GPIO ids can usually be found in the board documentation, defaults will be provided soon.

At the moment GPIOs are never unexported, let me know if you could find that useful. Multiple exporting when creating an already configured GPIO is not a problem, successive attempts to export a GPIO are simply ignored.

## Examples

The following example, built to run on the $9 C.H.I.P., shows the current value of all the GPIO0 attributes, changes direction and value and then shows again a recap of the attributes:

```Swift
let gpios = SwiftyGPIO.getGPIOsForBoard(.CHIP)
var gp0 = gpios[.P0]!
print("Current Status")
print("Direction: "+gp0.direction.rawValue)
print("Edge: "+gp0.edge.rawValue)
print("Active Low: "+String(gp0.activeLow))
print("Value: "+String(gp0.value))

gp0.direction = .OUT
gp0.value = 1

print("New Status")
print("Direction: "+gp0.direction.rawValue)
print("Edge: "+gp0.edge.rawValue)
print("Active Low: "+String(gp0.activeLow))
print("Value: "+String(gp0.value))
```

This second example makes a led blink with a frequency of 150ms:

```Swift
import Glibc

let gpios = SwiftyGPIO.getGPIOsForBoard(.CHIP)
var gp0 = gpios[.P0]!
gp0.direction = .OUT

repeat{
	gp0.value = (gp0.value == 0) ? 1 : 0
	usleep(150*1000)
}while(true) 
```

Other examples are available in the *Examples* directory.

## TODO

- [x] Create Package.swift
- [x] Basic example w/ package import
- [x] Add GPIOs default configurations for supported boards
- [ ] Testing on the Raspberries
- [ ] Testing on the BeagleBone Black
- [ ] Refactoring
