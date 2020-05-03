patch kernel & install dependencies:

```
apt-get install build-essential git ncurses-dev gcc-arm-linux-gnueabihf bc kmod
mkdir -p $HOME/projects/mahalia 
git clone git@github.com:mahalia-dependencies/mahalia-kernel $HOME/projects/mahalia/mahalia-kernel
cd $HOME/projects/mahalia/mahalia-kernel
git checkout ti-rt-linux-4.14.y-cape4all
# Branch ti-rt-linux-4.14.y-cape4all follows branch ti-rt-linux-4.14.y from
# https://git.ti.com/git/ti-linux-kernel/ti-linux-kernel.git and additionally
# contains the Cape4all driver from 0001-cape4all-driver-V4.patch and some
# later modifications.  If branch ti-rt-linux-4.14.y sees further development
# at the git.ti.com repository, then we should be able to merge those changes
# into our branch.
make distclean
./ti_config_fragments/defconfig_builder.sh -t ti_sdk_am3x_rt_release
ARCH=arm make ti_sdk_am3x_rt_release_defconfig
# Instead of  ARCH=arm make menuconfig  we can copy an existing .config:
cp cape4all_config .config
```

This is how we used menuconfig before:

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