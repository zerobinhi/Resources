### 2) 安装系统依赖

打开终端逐条执行（整行复制即可）：

```bash
sudo apt-get update
sudo apt-get install --no-install-recommends -y \
  git cmake ninja-build gperf ccache dfu-util device-tree-compiler wget \
  python3-dev python3-pip python3-venv python3-setuptools python3-tk python3-wheel \
  xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```

### 2) 安装 Zephyr SDK（交叉编译工具链）

```bash
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.17.4/zephyr-sdk-0.17.4_linux-x86_64.tar.xz

# （可选）校验
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.17.4/sha256.sum | shasum --check --ignore-missing

# 解压并运行初始化脚本
tar xvf zephyr-sdk-0.17.4_linux-x86_64.tar.xz
cd zephyr-sdk-0.17.4
./setup.sh
```

* `./setup.sh` 会配置工具及检测依赖；若换位置，需重跑一次。

### 3) 下载 ZMK 并初始化工作区

```bash
cd ~
git clone https://github.com/zmkfirmware/zmk.git
cd zmk

# 用虚拟环境隔离 Python 依赖
python3 -m venv .venv
source .venv/bin/activate

# 安装 west 并获取 Zephyr/ZMK 相关模块
pip install -U pip
pip install west
west init -l app/
west update
west zephyr-export
pip install -r zephyr/scripts/requirements-base.txt
```

### 4) 编译固件

```bash
cd app
west build -d build -p -b nrfmicro_13 -- -DSHIELD=keyboard -DCONFIG_PICOLIBC=n -DCONFIG_NEWLIB_LIBC=y -DCONFIG_NEWLIB_LIBC_NANO=y
```

* 首次构建会比较久，结束后生成的固件在 `build/zephyr/zmk.uf2`。

### 5) 刷写固件

* **UF2 引导器**（多数 nRF52 小板支持）：双击复位进 UF2 磁盘，把 `zmk.uf2` 直接复制进去即可自动重启。
* **DFU/J-Link/OpenOCD** 等方式：可直接 `west flash`。是否可用取决于板子支持的刷写方式与你安装的工具。

```bash
west flash -d build
```

* `-d build/left` 指定构建目录（里面有生成的 `.hex` 文件）。

* `west` 会自动检测板子，如果配置正确，就把固件刷入。

### 直接 `cp`

假设板子挂载在 `/media/username/NANOBOOT`：

```bash
cp build/left/zephyr/zmk.uf2 /media/username/NANOBOOT/
sync
```

* `sync` 确保写入完成。
* 板子会自动重启。

## 推荐方法

* 最简单、最安全的方法就是 **让板子进入 UF2 模式，然后 `cp` `.uf2` 文件**。
* 如果想用 `west flash` 自动刷 UF2，可以在 Linux 上安装 `dfu-util` 并设置 `BOARD` 支持 DFU。

在 ZMK/Zephyr 下：

```bash
sudo apt install dfu-util
west flash -d build --board nrfmicro_13
```

* 如果配置正确，`west flash` 会自动调用 `dfu-util` 把 `.hex` 刷入。
* UF2 支持有时需要把 `.uf2` 转成 `.hex` 才能被 `dfu-util` 识别。

### 5) 遇到的问题

* **`west: command not found`**：确认你已激活虚拟环境 `source .venv/bin/activate`，或把 `~/.local/bin` 加到 `PATH`。
