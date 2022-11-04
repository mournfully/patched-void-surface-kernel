# Installing void-linux with linux-surface kernel on a surface laptop 2
> **Warning** This is not a guide you should blindly follow nor a standalone tutorial. This is just my attempt at documenting all the hoops I had to jump through to get a working kernel. So, please **do not copy and paste commands and expect them to 'just work'**. 

To make a kernel with a linux-surface patchset for your void-linux install with native xbps integration, and with void's kernel install scripts `/etc/kernel.d`. This is done by editing a `linuxv5.x` template of your choice from the void-packages repo.

```shell
git clone --depth=1 https://github.com/linux-surface/linux-surface
git clone --depth=1 https://github.com/void-linux/void-packages
cd void-packages
./xbps-src binary-bootstrap
```

In this folder, make a copy of `srcpkgs/linux5.18` and rename it `srcpkgs/linux-surface`. This new package title will be henceforth referred to as `<pkgname>` and the directory you've created above I would now call `srcpkgs/<pkgname>`

Edit the template and change the `pkgname` variable to reflect your directory title 
(`pkgname=linux5.18` -> `pkgname=linux-surface`)

Change the `_kernver` variable to reflect your modified patchset version
(`${version}_${revision}` -> `${version}_${revision}-<patchset name>` aka `${version}_${revision}-surface`)

Change the following expression to match your `_kernver` variable. 
(`sed -i -e "s|^\(CONFIG_LOCALVERSION=\).*|\1\"_${revision}\"|" .config` -> 
`sed -i -e "s|^\(CONFIG_LOCALVERSION=\).*|\1\"_${revision}-surface\"|" .config`)

> assumes user has vim

use `vim template` and hotkey `shift+g` to get to the bottom of your file and  change the subpackage titles to reflect your new package name. 
(`linux5.18-headers_package()` -> `linux-surface-headers_package()`)
(`linux5.18-dbg_package()` -> `linux-surface-dbg_package()`)

Then make symlink like so, while in the `void-packages/srcpkg` directory.
(`ln -s <pkgname> <pkgname>-headers` -> `ln -s linux-surface/ linux-surface-headers`)
(`ln -s <pkgname> <pkgname>-dbg` -> `ln -s linux-surface/ linux-surface-dbg`)

> forgets to tell user to delete other config files and change x86 config name to this
> cat surface5.19 >> x86_64-dotconfig-custom

#### ignore this part its so dumb
> you can also use the `merge_config.sh` script
```shell
# never fucking mind, oh my lord so many errors >:C
cd ~/Downloads/
git clone --depth 1 https://github.com/linux-surface/linux-surface 
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.17.tar.gz
tar -xzvf linux-5.19.17.tar.gz
# cat /proc/config.gz | gunzip > ~/base-config
scp testbench:/home/local/void-packages/srcpkgs/linux5.19/files/x86_64-dotconfig /home/local

cd ~/Downloads/linux-5.19.17 # if not `make config` will fail
make mrproper # clean this directory
. ~/Downloads/linux-5.19.17/scripts/kconfig/merge_config.sh ~/x86_64-dotconfig ~/Downloads/linux-surface/configs/surface-5.19.config
cp ~/Downloads/linux-5.19.17/.config ~/x86_64-dotconfig-custom


```

> make sure to use `vscode` to compare `x86_64-dotconfig` with `x86_64-dotconfig-custom` and... yeah idk what to do about them...
####

To ensure that all Surface related options are set, you should merge the base config `srcpkg/<pkgname>/files/x86_64-dotconfig-custom` with one of the surface config fragments in `linux-surface/configs/` (just copy and paste `linux-surface/configs/` at the bottom of `x86_64-dotconfig-custom`). 

Most distro's have these enabled by default unlike void-linux so make sure the following options are enabled in your merged config file `x86_64-dotconfig-custom`.
- `CONFIG_ACPI=y`
- `CONFIG_DMI=y`
- `CONFIG_SURFACE_GPE=y`
> i want vhost on as well `CONFIG_VHOST_NET=y`
> found this while stalking past matrix conversations, `CONFIG_SERIAL_DEV_BUS=y` and `SERIAL_DEV_CTRL_TTYPORT=y` - *so basically you'll get a prompt from xbps-src while it's reading config*

*essentially, what you're missing right now is the serial device (UART) that provides the communication with the EC/Surface Aggregator. that depends on the serdev subsystem bindings to ttyport, which is configured by SERIAL_DEV_CTRL_TTYPORT=y, and that is only accessible if CONFIG_SERIAL_DEV_BUS=y
you can verify that that is present once you have a new kernel via ls -l /sys/bus/serial/devices/, which should not be empty*

basically do `SERIAL_DEV_BUS=y` in kernel config and then before xbps starts compiling it'll show an interactive prompt asking you about setting up *new configs* and then do the rest there.
```
Broadcom protocol support (BT_HCIUART_BCM) [N/y/?] (NEW) y

Serial device bus (SERIAL_DEV_BUS) [Y/n/m/?] y
  Serial device TTY port controller (SERIAL_DEV_CTRL_TTYPORT) [Y/n/?] (NEW) Y
```

##### ignore this idiot
> find duplicate lines `cat x86_64-dotconfig-custom | awk -F '=' '{print $1}' | sort | uniq -cd | grep -v "#" | tail -n +2 | sed 's/ 2 //' | awk '{$1=$1}1'`
> and deal with them
####
Since, you will be needing custom patches, copy them from `linux-surface/5.18/*.patch` and put them in `srcpkgs/<pkgname>/patches/`.

> cant install xtools on ubuntu - expect checksum fail, and to modify template. okay maybe not?

> set some configs for xbps-src
```shell
$ vim ~/void-packages/etc/conf
XBPS_MAKEJOBS=6 # number of threads
XBPS_CCACHE=yes
```
*You need to make sure to re-bootstrap your environment, i.e. do ./xbps-src zap and ./xbps-src binary-bootstrap again to make the etc/conf (not etc/config AFAICT) change take effect.*


Install `sudo xbps-install xtools` and then while in the `srcpkgs/` use `xgensum -i <pkgname>` aka `xgensum -i linux-surface` to give your template the correct checksums for your source files.

> don't forget to tell xbps-src to use all your threads with with a file in `void-packages/etc/defaults.conf` containing i.e. `XBPS_MAKEJOBS=6` or` ./xbps-src -j6`.

Once you've done all of this, go to the root of your void-packages clone and run `./xbps-src pkg <pkgname>` -> `./xbps-src pkg linux-surface -j6` 

Once built, the package will be available in `hostdir/binpkgs` or an appropriate subdirectory (e.g. `hostdir/binpkgs/nonfree`). To install the package:

Go to the root dir of your cloned void packages git repo.  ~~(`xi <pkgname>-headers <pkgname>` -> `xi linux-surface-headers linux-surface`)~~
```shell
# use scp to send them from remote to local 
cd ~/Downloads/void-packages/hostdir/binpkgs
sudo xbps-rindex -a *.xbps
sudo xbps-install --repository=$PWD linux-surface linux-surface-headers
```

Voila! you now have the benefit of full integration with xbps including the package configuration step generating the initramfs as well as running your bootloader kernel install hooks.

---

- for external modules, you may want dkms instead
- for in-tree modules, ask dev team if they can enable it :D
- and how would you handle kernel updates

---



linux-surface/linux-surface: Linux Kernel for Surface Devices
https://github.com/linux-surface/linux-surface

New package: linux-surface 5.13.13 by RononDex · Pull Request #32823 · void-linux/void-packages
https://github.com/void-linux/void-packages/pull/32823/files#diff-11976dfa1b101f08cc798bfa749f176cedf383210b4c4bbe27c9d4f1cda05e03

Quick guide on how to make your own kernel patchset xbps packages : voidlinux
https://www.reddit.com/r/voidlinux/comments/m5432a/quick_guide_on_how_to_make_your_own_kernel/

void-linux/void-packages: The Void source packages collection
https://github.com/void-linux/void-packages

Element | Linux-Surface Support
https://app.element.io/?pk_vid=fd3467fea1790b0d1654682955d18ee1#/room/#linux-surface-support:matrix.org/$UWQpyX27LpYTemFrpnJfpSc3l4cjZZIMYOG_tukpV5Y

gamja IRC client | voidlinux
https://web.libera.chat/gamja/

~1.5 weeks of testing changes and recompiling the entire kernel on one underpowered laptop while I was in school lol - rip my sanity

How can i use all my Threads with xbps-src? : voidlinux
https://www.reddit.com/r/voidlinux/comments/kjevgv/how_can_i_use_all_my_threads_with_xbpssrc/
