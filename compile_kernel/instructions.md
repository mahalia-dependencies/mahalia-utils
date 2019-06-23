patch kernel & install dependencies:

```
apt-get install build-essential git ncurses-dev gcc-arm-linux-gnueabihf bc kmod
mkdir -p $HOME/projects/mahalia
git clone git://git.ti.com/ti-linux-kernel/ti-linux-kernel.git $HOME/projects/mahalia/mahalia-kernel
cd $HOME/projects/mahalia/mahalia-kernel
git checkout origin/ti-rt-linux-4.14.y -b ti-rt-linux-4.14.y-cape4all
copy in 0001-cape4all-driver-V4.patch
git apply --reject --whitespace=fix 0001-cape4all-driver-V4.patch
make distclean
./ti_config_fragments/defconfig_builder.sh -t ti_sdk_am3x_rt_release
ARCH=arm make ti_sdk_am3x_rt_release_defconfig
ARCH=arm make menuconfig
```

Navigate thru menuconfig:

```
Setup real-time Kernel:
 > General Setup
	> Timers subsystem
		[ ] Old Idle dynticks config

 > Kernel Features
	> Preemption Model
		(X) Fully Preemptible Kernel (RT)

	> Timer frequency
		(X) 250 Hz


 > CPU Power Management
	> CPU Frequency scaling
		[ ] CPU Frequency scaling

	> CPU Idle
		[ ] CPU idle PM support
```

```
Compile the driver:
> Device Drivers
	> Sound card support > Advanced Linux Sound Architecture > ALSA for SoC audio support
		<M>   SoC Audio support for Boneblack Audio Extension

> File systems
	> Native language support
		<*> ASCII (United States)
```

```
Compile Bluetooth driver:

> Device Drivers
	> Character devices
		<*> Serial device bus

> Device Drivers
	> Misc devices > Texas Instruments shared transport line discipline
		<*> Shared transport core driver

> Networking support
	> Bluetooth subsystem support
		<*> RFCOMM protocol support
		[*] RFCOMM TTY support
		<*>     BNEP protocol support
		[*]       Multicast filter support
		[*]       Protocol filter support

> Networking support
	> Bluetooth subsystem support > Bluetooth device drivers
		< > Marvell Bluetooth driver support
		<M> Texas Instruments WiLink7 driver

		<M> HCI UART driver
		[*] HCILL protocol support


```
Enable power button driver:

INPUT_TPS65218_PWRBUTTON=y
> Device Drivers
	> Input device support > Miscellaneous devices
		<*> TPS65218 Power button driver

```
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 zImage modules dtbs
rm -rf compiled_modules && mkdir compiled_modules
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=compiled_modules make modules_install
```