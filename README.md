# Sensirion I2C SFX6XXX embedded Library

This document explains how to set up a sensor of the SFX6XXX family to run on an embedded device using the I²C interface.

<img src="images/product-image-sfx6xxx.png" width="300px">

Click [here](https://sensirion.com/sfc6000) to learn more about the Sensirion SFX6XXX sensor family.



## Supported sensor types

| Sensor name   | I²C Addresses  |
| ------------- | -------------- |
|[SFC6000](https://sensirion.com/products/catalog/SFC6000/)| **0x24**, 0x23, 0x22, 0x21, 0x20, 0x42, 0x41|
|[SFC6000D-5SLM](https://sensirion.com/products/catalog/SFC6000D-5slm/)| ****|
|[SFC6000D-50SLM](https://sensirion.com/products/catalog/SFC6000D-50slm/)| ****|
|[SFC6000D-20SLM](https://sensirion.com/products/catalog/SFC6000D-20slm/)| ****|
|[SFM6000](https://sensirion.com/products/catalog/SFM6000)| **0x24**, 0x23, 0x22, 0x21, 0x20, 0x42, 0x41|
|[SFM6000D-20SLM](https://sensirion.com/products/catalog/SFM6000D-20slm)| ****|
|[SFM6000D-50SLM](https://sensirion.com/products/catalog/SFM6000D-50slm)| ****|
|[SFM6000D-5SLM](https://sensirion.com/products/catalog/SFM6000D-5slm)| ****|

The following instructions and examples use a *SFC6000*.



## Setup Guide

### Connect the Sensor

Your sensor has 5 pins that need to be connected to your board: VDD, GND, SCL, ADDR, SDA.
Use the following description to connect your SFX6XXX:

<img src="images/product-pinout-i2c-sfx6xxx.png" width="300px">

| *Pin* | *Cable Color* | *Name* | *Description*  | *Comments* |
|-------|---------------|:------:|----------------|------------|
| 1 | red | VDD | Supply Voltage | +24V
| 2 | black | GND | Ground |
| 3 |  | NC | Do not connect |
| 4 | yellow | SCL | I2C: Serial clock input |
| 5 | purple | ADDR |  | Leave floating for default i2c address 0x24
| 6 | green | SDA | I2C: Serial data input / output |



The recommended voltage is 24V.

> Make sure to use pull-up resistors if your setup requires it. Learn more about pull-up resistors [here](https://github.com/Sensirion/arduino-i2c-different-buses-example/blob/main/i2c-pull-up-resistors-intro.md).

### Configure the code

In order to use the provided code we need to adjust two files according to your platform.

### Edit `sensirion_i2c_hal.c`

This file contains the implementation of the sensor communication, which
depends on your hardware platform. We provide function stubs for your
hardware's own implementation.
Sample implementations are available for some platforms:
[`sample-implementations`](sample-implementations). For Linux based platforms
like Raspberry Pi you can just replace the unimplemented HAL template with the
implementation in `sample-implementations/linux_user_space/`:

```
cp sample-implementations/linux_user_space/sensirion_i2c_hal.c ./
```

### Edit `sensirion_config.h`

Skip this part for Linux based platforms since everything is already setup for
this case.

Otherwise you need to check if the libraries `<stdint.h>` and `<stdlib.h>` are
provided by your toolchain, compiler or system. If you have no idea on how to
do that you can skip this step for now and come back when you get errors
related to these names when compiling the driver.
The features we use from those libraries are type definitions for integer sizes
from `<stdint.h>` and `NULL` from `<stdlib.h>`. If they are not available you
need to specify the following integer types yourself:

* `int64_t` = signed 64bit integer
* `uint64_t` = unsigned 64bit integer
* `int32_t` = signed 32bit integer
* `uint32_t` = unsigned 32bit integer
* `int16_t` = signed 16bit integer
* `uint16_t` = unsigned 16bit integer
* `int8_t` = signed 8bit integer
* `uint8_t` = unsigned 8bit integer

In addition to that you will need to specify `NULL`. For both we have a
detailed template where you just need to fill in your system specific values.

## Choose the i2c address to use with your product

The provided example is working with a SFC6000, I²C address 0x24.
In order to use the code with another product or I²C address you need to change it in the call sfx6xxx_init(ADDRESS) in
`sfx6xxx_i2c_example_usage.c`. The list of supported I²C-addresses is found in the header
`sfx6xxx_i2c.h`.


Now we are ready to compile and run the example usage for your sensor.

## Compile and Run

Pass the source `.c` and header `.h` files in this folder into your C compiler
and run the resulting binary. This step may vary, depending on your platform.
Here we demonstrate the procedure for Linux based platforms:

1. Open up a terminal.
2. Navigate to the directory where this README is located.
3. Navigate to the subdirectory example-usage.
4. Run `make` (this compiles the example code into one executable binary).
5. Run the compiled executable with `./sfx6xxx_i2c_example_usage`
6. Now you should see the first measurement values appear in your terminal. As
   a next step you can adjust the example usage file or write your own main
   function to use the sensor.

## Compile and Run Tests

The testframekwork used is CppUTest. Pass the source `.cpp`, `.c`  and header `.h`
files from the tests and top level folder into your CPP compiler and run the
resulting binary. This step may vary, depending on your platform.
Here we demonstrate the procedure for Linux based platforms:

1. Open up a terminal.
2. Install CppUTest framework `apt install cpputest`.
3. Navigate to the directory `tests`.
4. Run `make` (this compiles the test code into one executable binary).
5. Run the compiled executable with `./sfx6xxx_test_hw_i2c`.
6. Now you should see the test output on your console.

# Background

## Files

### sensirion\_i2c.[ch]

In these files you can find the implementation of the I2C protocol used by
Sensirion sensors. The functions in these files are used by the embedded driver
to build the correct frame out of data to be sent to the sensor or receive a
frame of data from the sensor and convert it back to data readable by your
machine. The functions in here calculate and check CRCs, reorder bytes for
different byte orders and build the correct formatted frame for your sensor.

### sensirion\_i2c\_hal.[ch]

These files contain the implementation of the hardware abstraction layer used
by Sensirion's I2C embedded drivers. This part of the code is specific to the
underlying hardware platform. This is an unimplemented template for the user to
implement. In the `sample-implementations/` folder we provide implementations
for the most common platforms.

### sensirion\_config.h

In this file we keep all the included libraries for our drivers and global
defines. Next to `sensirion_i2c_hal.c` *it's the only file you should need to
edit to get your driver working.*

### sensirion\_common.[ch]

In these files you can find some helper functions used by Sensirion's embedded
drivers. It mostly contains byte order conversions for different variable
types. These functions are also used by the UART embedded drivers therefore
they are kept in their own file.


## Notes
### Implications of 24V power requirement

Since most development boards won't provide the required 24V, the power will be supplied by an external source, therefore **the sensor will be unnafected by an development board power cycle**.

If a measurement is running at the time when a board reset is performed, the sensor will still be measuring when the board and its MCU starts back up.

That is the reason why the provided example includes a `stopContinuousMeasurement()` before the call to `startO2ContinuousMeasurement()`. That will stop a potentially running measurement.


## Contributing

**Contributions are welcome!**

This Sensirion library uses
[`clang-format`](https://releases.llvm.org/download.html) to standardize the
formatting of all our `.c` and `.h` files. Make sure your contributions are
formatted accordingly:

The `-i` flag will apply the format changes to the files listed.

```bash
clang-format -i *.c *.h
```

Note that differences from this formatting will result in a failed build until
they are fixed.


# License

See [LICENSE](LICENSE).