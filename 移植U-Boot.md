克隆仓库

```bash
git clone --recursive https://github.com/nxp-imx/uboot-imx.git
```

```bash
cd uboot-imx/
touch build-mx6ull-alientek-emmc.sh
chmod 777 build-mx6ull-alientek-emmc.sh
vim build-mx6ull-alientek-emmc.sh
```

输入以下内容

```sh
#!/bin/bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_alientek_emmc_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make V=1 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j10
```

`defconfig`文件拷贝文件和修改

```bash
cp configs/mx6ull_14x14_evk_emmc_defconfig configs/mx6ull_alientek_emmc_defconfig
vim configs/mx6ull_alientek_emmc_defconfig
```

修改

```tex
CONFIG_TARGET_MX6ULL_ALIENTEK_EMMC=y
CONFIG_DEFAULT_DEVICE_TREE="imx6ull-alientek-emmc"
添加
CONFIG_CMD_NFS=y
CONFIG_CMD_DNS=y
```



头拷贝文件

```bash
cp include/configs/mx6ullevk.h include/configs/mx6ull_alientek_emmc.h
```

```bash
vim include/configs/mx6ull_alientek_emmc.h
```

修改

```c
#ifndef __MX6ULL_ALIENTEK_EMMC_CONFIG_H
#define __MX6ULL_ALIENTEK_EMMC_CONFIG_H
```



拷贝整个文件夹

```bash
cp -r board/freescale/mx6ullevk board/freescale/mx6ull_alientek_emmc
```

修改c文件，Makefile，MAINTAINERS，Kconfig，imximage.cfg共计五项内容

```bash
mv board/freescale/mx6ull_alientek_emmc/mx6ullevk.c board/freescale/mx6ull_alientek_emmc/mx6ull_alientek_emmc.c
vim board/freescale/mx6ull_alientek_emmc/mx6ull_alientek_emmc.c
```

打开C文件，修改`checkboard`函数为

```c
int checkboard(void)
{
	if (is_mx6ull_9x9_evk())
		puts("Board: MX6ULL 9x9 EVK\n");
	else if (is_cpu_type(MXC_CPU_MX6ULZ))
		puts("Board: MX6ULZ 14x14 EVK\n");
	else
		puts("Board: MX6ULL ALIENTEK EMMC\n");
	return 0;
}
```



`Makefile`的修改

```bash
vim board/freescale/mx6ull_alientek_emmc/Makefile
```

```tex
obj-y  := mx6ull_alientek_emmc.o
```

`MAINTAINERS`修改

```bash
vim board/freescale/mx6ull_alientek_emmc/MAINTAINERS
```

修改为

```tex
MX6ULL_ALIENTEK_EMMC BOARD
M:	Peng Fan <peng.fan@nxp.com>
S:	Maintained
F:	board/freescale/mx6ull_alientek_emmc/
F:	include/configs/mx6ull_alientek_emmc.h
F:	configs/mx6ull_alientek_emmc_defconfig
F:	configs/mx6ull_14x14_evk_plugin_defconfig
F:	configs/mx6ulz_14x14_evk_defconfig
```

`Kconfig`修改

```bash
vim board/freescale/mx6ull_alientek_emmc/Kconfig
```

改为

```tex
if TARGET_MX6ULL_ALIENTEK_EMMC

config SYS_BOARD
	default "mx6ull_alientek_emmc"

config SYS_VENDOR
	default "freescale"

config SYS_CONFIG_NAME
	default "mx6ull_alientek_emmc"

config IMX_CONFIG
	default "board/freescale/mx6ull_alientek_emmc/imximage.cfg"

config TEXT_BASE
	default 0x87800000
endif
```

`imximage.cfg`修改

```bash
vim board/freescale/mx6ull_alientek_emmc/imximage.cfg
```

```c
#ifdef CONFIG_USE_IMXIMG_PLUGIN
/*PLUGIN    plugin-binary-file    IRAM_FREE_START_ADDR*/
PLUGIN	board/freescale/mx6ull_alientek_emmc/plugin.bin 0x00907000
```



`imximage_lpddr2.cfg`的修改

```bash
vim board/freescale/mx6ull_alientek_emmc/imximage_lpddr2.cfg
```

```c
#ifdef CONFIG_USE_IMXIMG_PLUGIN 
/*PLUGIN    plugin-binary-file    IRAM_FREE_START_ADDR*/ 
PLUGIN  board/freescale/mx6ull_alientek_emmc/plugin.bin 0x00907000 
```



```bash
vim arch/arm/mach-imx/mx6/Kconfig
```

Kconfig中搜索`TARGET_MX6ULL_14X14_EVK`，在下面添加

```tex
config TARGET_MX6ULL_ALIENTEK_EMMC
	bool "Support mx6ull_alientek_emmc"
	depends on MX6ULL
	select BOARD_LATE_INIT
	select DM
	select DM_THERMAL
	select IOMUX_LPSR
	select IMX_MODULE_FUSE
	select OF_SYSTEM_SETUP
	imply CMD_DM
```

在末尾添加

```tex
source "board/freescale/mx6ull_alientek_emmc/Kconfig"
```



拷贝文件

```bash
cp arch/arm/dts/imx6ull-14x14-evk.dts arch/arm/dts/imx6ull-alientek-emmc.dts
cp arch/arm/dts/imx6ul-14x14-evk.dtsi arch/arm/dts/imx6ul-alientek-emmc.dtsi
vim arch/arm/dts/imx6ull-alientek-emmc.dts
```

头文件换成

```tex
#include "imx6ull.dtsi"
#include "imx6ul-alientek-emmc.dtsi"
#include "imx6ul-14x14-evk-u-boot.dtsi"
```





```bash
vim arch/arm/dts/imx6ull-alientek-emmc.dts
```

```tex
/ {
	model = "I.MX6ULL Alientek ALPHA ";
	compatible = "fsl,imx6ull-alientek", "fsl,imx6ull";
};
```

添加节点

```tex
&usdhc2 {
	pinctrl-names = "default", "state_100mhz", "state_200mhz";
	pinctrl-0 = <&pinctrl_usdhc2_8bit>;
	pinctrl-1 = <&pinctrl_usdhc2_8bit_100mhz>;
	pinctrl-2 = <&pinctrl_usdhc2_8bit_200mhz>;
	bus-width = <8>;
	non-removable;
	status = "okay";
};
```





修改`lcdif`节点的信息

```bash
vim arch/arm/dts/imx6ul-alientek-emmc.dtsi
```

```tex
&lcdif {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_lcdif_dat
		     &pinctrl_lcdif_ctrl>;
	display = <&display0>;
	status = "okay";

	display0: display@0 {
		bits-per-pixel = <24>;
		bus-width = <24>;

		display-timings {
			native-mode = <&timing0>;
			
			timing0: timing0 {
			clock-frequency = <51200000>;
			hactive = <1024>;
			vactive = <600>;
			hfront-porch = <160>;
			hback-porch = <140>;
			hsync-len = <20>;
			vback-porch = <20>;
			vfront-porch = <12>;
			vsync-len = <3>;
			hsync-active = <0>;
			vsync-active = <0>;
			de-active = <1>;
			pixelclk-active = <0>;
			};
		};
	};
};
```



把`ethphy0`节点的名称改成`ethernet-phy@0`，`reg`改成0



清空环境变量

```bash
env default -a
saveenv
```



```tex
setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs nfsroot=192.168.1.18:/home/zuozhongkai/linux/nfs/rootfs,proto=tcp rw ip=192.168.1.21:192.168.1.18:192.168.0.1:255.255.254.0::eth0:off'
setenv ipaddr 192.168.1.21
setenv serverip 192.168.1.18
setenv gatewayip 192.168.0.1
setenv netmask 255.255.254.0
setenv ethaddr b8:ae:1d:13:A3:a0
setenv eth1addr b8:ae:1d:12:33:a0
setenv bootcmd 'tftp 80800000 zImage; tftp 83000000 imx6ull-alientek-emmc.dtb; bootz 80800000 - 83000000' 
saveenv
```

















