~~~bash
# 如何在Linux系统下搭建开发环境

## Step 1：安装系统依赖

### 1️⃣ 基础系统更新

```bash
sudo apt update
sudo apt upgrade
```

### 2️⃣ 安装 Zephyr / ZMK 所需依赖

```bash
sudo apt install --no-install-recommends \
  git cmake ninja-build gperf ccache dfu-util device-tree-compiler \
  wget python3-dev python3-venv python3-tk xz-utils file make \
  gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```

验证一下：

```bash
cmake --version
python3 --version
dtc --version
```

------

## Step 2：获取 ZMK 源码

```bash
cd ~
git clone https://github.com/zmkfirmware/zmk.git
cd zmk
```

------

## Step 3：Python 虚拟环境

### 1️⃣ 安装 venv

```bash
sudo apt install python3-venv
```

### 2️⃣ 创建并激活虚拟环境

```bash
python3 -m venv .venv
source .venv/bin/activate
```

终端前缀应该变成：

```bash
(.venv)
```

以后只要你想编译 ZMK，就先：

```bash
cd ~/zmk
source .venv/bin/activate
```

------

## Step 4：west + Zephyr 拉取

### 1️⃣ 安装 west

```bash
pip install west
```

### 2️⃣ 初始化 Zephyr

```bash
west init -l app/
west update
```

### 3️⃣ 导出 Zephyr CMake 包

```bash
west zephyr-export
```

### 4️⃣ 安装 Python 依赖

```bash
west packages pip --install
```

------

## Step 5：安装 Zephyr SDK

```bash
cd ./zephyr
west sdk install
```

环境已经搭建完成！

# 烧录教程

将提供的 `keyboard.tar.gz` 拷贝到 ZMK 的 shield 目录：

```bash
cp ./keyboard.tar.gz ~/zmk/app/boards/shields
```

进入 shield 目录并解压：

```bash
cd ~/zmk/app/boards/shields
tar -zxvf ./keyboard.tar.gz
```

解压完成后，目录结构应类似于：

```bash
boards/shields/keyboard/
├── Kconfig.defconfig
├── Kconfig.shield
├── keyboard.conf
├── keyboard.keymap
├── keyboard.overlay
├── keyboard.zmk.yml
└── README.md
```

如果结构如上，说明 shield 放置正确。

------

## 三、编译固件

回到 ZMK 应用目录：

```bash
cd ~/zmk/app
```

执行编译命令：

```bash
west build -d build -p -b nrfmicro/nrf52840 -S nrf52840-nosd -- -DSHIELD=keyboard
```

### 参数说明（简要）

- `-b nrfmicro/nrf52840`
   指定使用 nrfmicro 的 nRF52840 变体
- `-S nrf52840-nosd`
   使用不包含 SoftDevice 的构建方案（ZMK 推荐）
- `-DSHIELD=keyboard`
   指定使用自定义的 `keyboard` shield

------

## 四、编译结果

如果编译成功，你将在以下路径看到固件文件：

```bash
ls ~/zmk/app/build/zephyr/zmk.uf2
```xxxxxxxxxx ls ~/zmk/app/build/zephyr/zmk.uf2bash
~~~