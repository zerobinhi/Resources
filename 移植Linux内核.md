克隆仓库

```bash
git clone --recursive https://github.com/nxp-imx/linux-imx.git
```

```bash
cd linux-imx/
touch build-mx6ull-alientek-emmc.sh
chmod 777 build-mx6ull-alientek-emmc.sh
vim build-mx6ull-alientek-emmc.sh
```

 输入以下内容

```sh
#!/bin/sh 
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- imx_alientek_emmc_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- all -j1
```

```bash
cp arch/arm/configs/imx_v7_defconfig arch/arm/configs/imx_alientek_emmc_defconfig
cp ./arch/arm/boot/dts/nxp/imx/imx6ull-14x14-evk-emmc.dts ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek-emmc.dts
cp ./arch/arm/boot/dts/nxp/imx/imx6ull-14x14-evk.dts ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek.dts
cp ./arch/arm/boot/dts/nxp/imx/imx6ul-14x14-evk.dtsi ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek.dtsi
```



```bash
vim arch/arm/configs/imx_alientek_emmc_defconfig
```

添加

```tex
CONFIG_EXTRA_FIRMWARE_DIR="firmware"
CONFIG_EXTRA_FIRMWARE="imx/sdma/sdma-imx6q.bin"
```

```tex
CONFIG_CFG80211=y 修改为 CONFIG_CFG80211=m
```



```bash
vim ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek-emmc.dts
```

```c
#include "imx6ull-alientek.dts" 
```



```bash
vim ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek.dtsi
```

`pinctrl_spi4`**节点中删除**

MX6UL_PAD_SNVS_TAMPER7__GPIO5_IO07  0x70a1 

MX6UL_PAD_SNVS_TAMPER8__GPIO5_IO08  0x80000000 

在`spi-4`节点中删除

cs-gpios = <&gpio5 7 GPIO_ACTIVE_LOW>; 

enable-gpios = <&gpio5 8 GPIO_ACTIVE_LOW>; 



替换`&fec2`节点的ethphy0: ethernet-phy@2

把2改成0



替换`&lcdif`

```tex
&lcdif {
	assigned-clocks = <&clks IMX6UL_CLK_LCDIF_PRE_SEL>;
	assigned-clock-parents = <&clks IMX6UL_CLK_PLL5_VIDEO_DIV>;
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





```bash
vim ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek.dts
```

```c
#include "imx6ull-alientek.dtsi" 
```

添加节点

```tex
/ {
	model = "I.MX6ULL Alientek ALPHA";
	compatible = "imx6ull alientek", "fsl,imx6ull";
};
```









```bash
vim ./arch/arm/boot/dts/nxp/imx/Makefile
```

在`imx6ull-14x14-evk-emmc.dtb`下面添加

```makefile
imx6ull-alientek-emmc.dtb \
```



打开网页

https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/imx/sdma/sdma-imx6q.bin

点击 plain 下载sdma-imx6q.bin

```bash
mkdir ./firmware/imx/sdma -p
```

将下好的sdma-imx6q.bin放进去





打开网页，下载最新的包

https://mirrors.edge.kernel.org/pub/software/network/wireless-regdb/

解压缩后需要将文件 regulatory.db 和 regulatory.db.p7s 复制到/lib/firmware中

```bash
mkdir ./lib/firmware -p
```



```bash
mkdir -r ~/linux/tftpboot/*
rm ~/linux/tftpboot/*
cp ./arch/arm/boot/dts/nxp/imx/imx6ull-alientek-emmc.dtb ~/linux/tftpboot/
cp ./arch/arm/boot/zImage ~/linux/tftpboot/
chmod 777 ~/linux/tftpboot/*
```



