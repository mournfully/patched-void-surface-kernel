# Installing void-linux with linux-surface kernel
### Patching Kernels
To make a kernel with a linux-surface patchset for your void-linux install with native xbps integration, and with void's kernel install scripts `/etc/kernel.d`. This is done by editing a `linuxv5.x` template of your choice from the void-packages repo.

```
git clone https://github.com/linux-surface/linux-surface
git clone https://github.com/void-linux/void-packages
cd void-packages
./xbps-src binary-bootstrap
```

In this folder, make a copy of `srcpkgs/linux5.18` and rename it `srcpkgs/linux-surface`. This new package title will be henceforth referred to as `<pkgname>` and the directory you've created above I would now call `srcpkgs/<pkgname>`

Edit the template and change the `pkgname` variable to reflect your directory title 
(`pkgname=linux5.18` -> `pkgname=linux-surface`)

Change the `_kernver` variable to reflect your modified patchset version
(`${version}_${revision}` -> `${version}_${revision}-<patchset name>` aka `${version}_${revision}-surface`)

Change the following expression to match your `_kernver` variable. 
(`sed -i -e "s|^\(CONFIG_LOCALVERSION=\).*|\1\"_${revision}\"|" .config` aka 
`sed -i -e "s|^\(CONFIG_LOCALVERSION=\).*|\1\"_${revision}-surface\"|" .config`)

use `vim template` and hotkey `shift+g` to get to the bottom of your file and  change the subpackage titles to reflect your new package name. 
(`linux5.18-headers_package()` -> `linux-surface-headers_package()`)
(`linux5.18-dbg_package()` -> `linux-surface-dbg_package()`)

Then make symlink like so, while in the `void-packages/srcpkg` directory.
(`ln -s <pkgname> <pkgname>-headers` -> `ln -s linux-surface/ linux-surface-headers`)
(`ln -s <pkgname> <pkgname>-dbg` -> `ln -s linux-surface/ linux-surface-dbg`)

To ensure that all Surface related options are set, you should merge the base config `srcpkg/<pkgname>/files/x86_64-dotconfig-custom` with one of the surface config fragments in `linux-surface/configs/` (just copy and paste `linux-surface/configs/` at the bottom of `x86_64-dotconfig-custom`). 

Most distro's have these enabled by default unlike void-linux so make sure the following options are enabled in your merged config file `x86_64-dotconfig-custom`.
- `CONFIG_ACPI=y`
- `CONFIG_DMI=y`
- `CONFIG_SURFACE_GPE=y`

Since, you will be needing custom patches, copy them from `linux-surface/5.18/*.patch` and put them in `srcpkgs/<pkgname>/patches/`.

Install `sudo xbps-install xtools` and then while in the `srcpkgs/` use `xgensum -i <pkgname>` aka `xgensum -i linux-surface` to give your template the correct checksums for your source files.

Once you've done all of this, go to the root of your void-packages clone and run `./xbps-src pkg <pkgname>` -> `./xbps-src pkg linux5.18-surface` 

Once built, the package will be available in `hostdir/binpkgs` or an appropriate subdirectory (e.g. `hostdir/binpkgs/nonfree`). To install the package:

Go to the root dir of your cloned void packages git repo.  (`xi <pkgname>-headers <pkgname>` -> `xi linux5.18-surface-headers linux5.18-surface`)

Voila! you now have the benefit of full integration with xbps including the package configuration step generating the initramfs as well as running your bootloader kernel install hooks.

### References
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

~1.5 weeks of testing changes and recompiling the entire kernel everytime on one underpowered laptop while I was in school lol
source: my sanity
