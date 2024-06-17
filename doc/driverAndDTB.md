## 1.Describe the device tree and load it
The `0x50` device on `i2c-0` is used as an example.

All operations are on the jetson development board
### 1.Create the dts overlay file ov9282.dts
```
/dts-v1/;
/plugin/;

/ {
    compatible = "nvidia,p3768-0000+p3767-0001\0nvidia,p3767-0001\0nvidia,tegra234\0nvidia,tegra23x";
    overlay-name = "My OV9282 Overlay Example";
    jetson-header-name = "Jetson 40pin Header";

    fragment@0 {
        target-path = "/i2c@3160000/";
        __overlay__ {
            status = "okay";
            #address-cells = <1>;
            #size-cells = <0>;
            my-value = "test";
            camera@50 {
                compatible = "ovti,ov9282";
                reg = <0x50>;
                assigned-clock-rates = <24000000>;
            };
        };
    };
};
```
### 2.Compile and load
Compile the DTS source file as an overlay file.
```
dtc -O dtb -o ov9282.dtbo -@ ov9282.dts
```
copy the new overlay file to the /boot directory
```
sudo cp ov9282.dtbo /boot
```
jetson-io tool finds the overlay file
```
sudo /opt/nvidia/jetson-io/config-by-hardware.py -l
```
![img_2.png](image%2Fimg_2.png)

Load the "My OV9282 Overlay Example" overlay file
```
sudo /opt/nvidia/jetson-io/config-by-hardware.py -n "My OV9282 Overlay Example"
```
![img_3.png](image%2Fimg_3.png)

After the reboot, you can see the new device node

![img_4.png](image%2Fimg_4.png)

## 2.Use the device driver to get device information
### 1.compile driver
Modify ov9282.c to temporarily mask the parts that failed to compile and print the device information after the device is found
```
static int ov9282_probe(struct i2c_client *client)
{
        struct ov9282 *ov9282;
        int ret, i;
        printk("Found device ov9282");
        u8 buf[256];
        ret = i2c_master_recv(client, buf, sizeof(buf));

        if (ret < 0) {
                dev_err(&client->dev, "Failed to read from device\n");
                return ret;
        }
        char output[256];
        output[0] = '\0';
        for (i = 0; i < ret; i++) {
                sprintf(output + strlen(output), "%c", buf[i]);
        }
        printk("%s\n", output);
        return 0;
}
```
Here is just some sample code,full code in [ov9282.c](tmp%2Fov9282.c)

Create Makefile with below contents
```
obj-m += ov9282.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
Make the module
```
make
```
### 2.Insert the module and check the kernel print
Insert the module
```
sudo insmod  ov9282.ko
sudo dmesg
```
Below is a print of the module
```
[16759.742345] Found device ov9282
[16759.748918] 
               \xb0Hv:\xd9-\xb0H1421023065870NVCB\xffM1v:\xd9-\xb0H\x91\xfe\xff699-13767-0001-300 N.2
```

### 3.check I2c device
When the driver loads successfully, we can see that `0x50` shows `UU`

![img_6.png](image%2Fimg_6.png)


#### ps: next step(1.Write the device tree for the camera. 2.Modify the driver module to compile and register the camera as a v4l2 device)