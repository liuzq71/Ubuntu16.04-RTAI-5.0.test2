# Ubuntu16.04-RTAI-5.0.test2

## -Download and Unzip-
	cd /usr/src
	curl -L --insecure https://www.rtai.org/userfiles/downloads/RTAI/rtai-5.0-test2.tar.bz2 | tar xj
	curl -L https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.20.tar.xz | tar xJ
	curl -L http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.18.20-vivid/linux-image-3.18.20-031820-generic_3.18.20-031820.201508081633_i386.deb -o linux-image-3.18.20-generic-i386.deb
	dpkg-deb -x linux-image-3.18.20-generic-i386.deb linux-image-3.18.20-generic-i386

## -Link(optional)-
	rm linux
	rm rtai
	ln -s linux-3.18.20 linux
	ln -s rtai-5.0-test2 rtai
	
## -General-
	apt-get -y install cvs subversion build-essential git-core g++-multilib gcc-multilib ncurses-dev vim

## -Rtai-
	apt-get -y install libtool automake libncurses5-dev kernel-package
	/*Copy ubuntu kernel 3.18 original config:*/
	cp /usr/src/linux-image-3.18.20-generic-i386/boot/config-3.18.20-031820-generic /usr/src/linux/.config
	/*Patch for kernel:*/
	cd /usr/src/linux
	patch -p1 < /usr/src/rtai/base/arch/x86/patches/hal-linux-3.18.20-x86-6.patch
	
## -Config-
	make menuconfig
	/*Set up the kernel settings as follows:*/
	Enable Loadable Module Support
			-> Module Versioning Support = DISABLE IT
	Processor type and features
			-> SMT (Hyperthreading) scheduler support = DISABLE IT
	Power Management and ACPI options
			-> ACPI (Advanced Configuration and Power Interface) Support -> PROCESSOR = DISABLE IT
			-> CPU Frequency scaling -> CPU Frequency scaling = DISABLE IT. *(“SpeedStep” in BIOS should be also taken into consideration)
			-> Cpu idle --> CPU idle PM support --> DISABLE IT
			
## -Build-
	make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-rtai 
	/*Install:*/
	dpkg -i ../linux-image-3.18.20-rtai_3.18.20-rtai-1_i386.deb
	dpkg -i ../linux-headers-3.18.20-rtai_3.18.20-rtai-1_i386.deb

## -Modify /etc/default/grub-
	gedit /etc/default/grub
	/*Set GRUB_CMDLINE_LINUX_DEFAULT as:*/
	#GRUB_HIDDEN_TIMEOUT=0  
	GRUB_TIMEOUT=-1  %This setting will make grub menu prompt and stop for user selection.%
	GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash lapic=notscdeadline”
	
## -Apply grub config-
	update-grub
	
## -Now, reboot your system. choose the RTAI in GRUB boot menu.-

## -RTAI installation-
	/*Now that the system has restarted, make sure you booted on your RTAI kernel*/
	uname -r 
	/*It should print:*/
	3.18.20-rtai
	
## -Config RTAI-
	cd /usr/src/rtai
	make menuconfig
	/*Update the CPUs number according to your configuration.*/
	
## -Build-
	make -j `getconf _NPROCESSORS_ONLN`

## -Install-
	make install
	/*Add following line to your ~/.bashrc (if you use bash):*/
	gedit ~/.bashrc
	export PATH=/usr/realtime/bin:$PATH 
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/realtime/lib
	
## -Use new environment variable-
	source ~/.bashrc
	/*Create a file /etc/ld.so.conf.d/rtai.conf with the content*/
	/usr/realtime/lib
	/*Run ldconfigin order to take it into account:*/
	ldconfig
	
## -Disable iTOC watchdog-
	/*Add following in file /etc/modprobe.d/blacklist-watchdog.conf*/
	blacklist iTCO_vendor_support
	/*Reboot and check “lsmod | grep iTCO”, there should be no output in terminal.*/
	
## -Now, run latency test-
	cd /usr/realtime/testsuite/kern/latency
	./run
