# How to get this repository
## Method 1
	git clone https://github.com/ngttai/ESP8266_RTOS_DEV.git
	git submodule init
	git submodule update
## Method 2
	git clone --recurse-submodules https://github.com/ngttai/ESP8266_RTOS_DEV.git
# How to set environment
	cd ESP8266_RTOS_DEV 
	source set_environment

# OS host: UBUNTU 18.04.2

# Install needed dependencies (as root)
## 1. 32-Bit Debian (Linux)
`sudo apt install git autoconf build-essential gperf bison flex texinfo libtool libncurses5-dev wget gawk libc6-dev-i386 python-serial libexpat-dev`
## 2. 64-Bit Debian (Linux)
`sudo apt install git autoconf build-essential gperf bison flex texinfo libtool libncurses5-dev wget gawk libc6-dev-amd64 python-serial libexpat-dev`

# Install esptool
    pip install esptool
    sudo usermod -a -G dialout $USERNAME
    "Logout and login again"

# Befor build the toolchain, I recomment download from internet some file as below and store it to:
    crosstool-NG/.build/tarballs
## The is the name of them:
    binutils-2.24.tar.bz2
    cloog-0.18.1.tar.gz
    gcc-4.8.2.tar.bz2
    gdb-7.5.1.tar.bz2
    gmp-5.1.3.tar.xz
    isl-0.12.2.tar.bz2
    mpc-1.0.2.tar.gz
    mpfr-3.1.2.tar.xz
    newlib-2.0.0.tar.gz

# Build the toolchain
    mkdir -p $HOME/esp
    cd $HOME/esp
    git clone -b lx106-g++ git://github.com/jcmvbkbc/crosstool-NG.git

    cd crosstool-NG
    /bootstrap && ./configure --prefix=`pwd` && make && make install
    ./ct-ng xtensa-lx106-elf
    ./ct-ng build

## If you get an error when build as: 
"[ERROR] cfns.gperf:101:1: error: 'const char* libc_name_p(const char*, unsigned int)' redeclared inline with 'gnu_inline' attribute"
Solve found here: <https://github.com/esp8266/esp8266-wiki/issues/74>

## Solve is below:

### 1. Go to: 
    crosstool-NG/.build/src/gcc-4.8.2/gcc/cp/cfns.h
### 2. Change: 

    #ifdef __GNUC__
    __inline
    #ifdef __GNUC_STDC_INLINE__
    __attribute__ ((__gnu_inline__))
    #endif
    #endif

### To:

    #ifdef __GNUC_STDC_INLINE__
    __attribute__ ((__gnu_inline__))
    #endif

## NOTE: I recomment change it before build

# After build complete

    cd $HOME/esp
    cp -r $HOME/esp/crosstool-NG/builds/xtensa-lx106-elf .
    export PATH=`pwd`/xtensa-lx106-elf/bin:$PATH

# Get ESP_RTOS_SDK branch release/v2.xx

    cd $HOME/esp
    git clone -b release/v2.x.x https://github.com/espressif/ESP8266_RTOS_SDK.git
    export SDK_PATH=`pwd`/ESP8266_RTOS_SDK
    export BIN_PATH=`pwd`/ESP8266_BIN

# Build an example application

    cd $HOME/esp
    cp -r ESP8266_RTOS_SDK/examples/wifi_station_machine_demo .
    cd wifi_station_machine_demo
    ./gen_misc.sh
    # Enter Y 0 2 0 4

# Flash to ESP8266

    esptool.py -b 921600 write_flash \
        0x3FE000 $SDK_PATH/bin/blank.bin \
        0x3FC000 $SDK_PATH/bin/esp_init_data_default.bin \
        0x00000 $BIN_PATH/eagle.flash.bin \
        0x20000 $BIN_PATH/eagle.irom0text.bin
