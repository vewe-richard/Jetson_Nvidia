## Looking for I2c devices

When an I2c device is connected to a linux board, the kernel usually automatically identifies and enumerates the I2c devices connected to the system and registers them on the system's I2C bus. Therefore, we can run the `i2cdetect` command to check whether new devices appear.

### 1. List the I2c bus
Use `i2cdetect -l` to list all the I2c buses
![img.png](image%2Fimg.png)
### 2.List the devices on the I2c bus
Use the `i2cdetect-y 0` command to list the devices on the `i2c-0` bus
![img_1.png](image%2Fimg_1.png)
There are devices on the `i2c-0` bus `0x50` and `0x57`, but I've verified that none of them are camera devices

### 3.How to operate
I recommend using `i2cdetect-y *`(0-11) after inserting the camera to check whether new devices appear on all buses, and then unplug the camera to confirm

I'm guessing it's on I2c buses 0 and 4