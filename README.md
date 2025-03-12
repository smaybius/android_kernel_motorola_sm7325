# What do I do if LineageOS doesn't have a kernel repo for my device's SOC? (assuming it's SMx3xx, where the second digit is specifically 3)
- The second digit of the SOC's codename matters, but look for the closest. For example, the Snapdragon 4 Gen 1 is the SM4375, and the closest that LineageOS already offers is the SM6375. In that case, make a fork of LineageOS/android_kernel_motorola_sm6375. All the latest mainstream devices run GKI kernels, meaning that it shouldn't matter as long as the second digit of the SOC's codename is the same.
- Go to https://github.com/MotorolaMobilityLLC/kernel-msm/ and find the branch or tag for your device's build ID. For example, fogo (Moto G 5G 2024) has a build ID that starts with U1UFNS34. If there isn't one available, submit a request in the issues tab, such as https://github.com/MotorolaMobilityLLC/kernel-msm/issues/672 after reviewing all the kernel-*-devicetree repos under MotorolaMobilityLLC to make sure if there's a branch or tag for your device's build ID.
- In `arch/arm64/boot/configs/vendor`, copy both `debug-[holi/lahaina]-(devicename).config` and `moto-[holi/lahaina]-(devicename).config` into this repo's `arch/arm64/boot/configs/vendor`.
- If available, find the tag for your device's build ID in both kernel-devicetree and all kernel-*-devicetree repos, and download them. Delete each `.gitignore` in the downloads.
- First, extract the `qcom` folder of `kernel-devicetree-MMI-*` into `arch/arm64/boot/dts/vendor`, saying no to replacing any file.
- Compare the makefile in your download with the one that exists, and add anything that's new.
- Extract all subsequent `kernel-*-MMI-*` ZIPs into `arch/arm64/boot/dts/vendor/qcom`, while skipping the `bindings` folder and saying no to replacing any file.
- If there's a `.gitignore` in `arch/arm64/boot/dts/vendor`, check it if it includes any directories that are being pasted into. Delete them if yes.
- (re-)Generate the defconfig, whether it's `holi-qgki_defconfig` or `lahaina-qgki_defconfig`, by running this in your terminal, assuming that your board family is holi (SM43xx), or replace "holi" with "lahaina" if it's not a Snapdragon 4 series SOC:
```bash
PATH="/path/to/lineageos/prebuilts/tools-lineage/linux-x86/bin:/path/to/lineageos/prebuilts/clang/host/linux-x86/clang-r530567/bin:$PATH"
ARCH=arm64 \
CROSS_COMPILE=aarch64-linux-gnu- \
REAL_CC=clang \
CLANG_TRIPLE=aarch64-linux-gnu- \
LD=ld.lld \
AR=llvm-ar \
LLVM=1 \
LLVM_IAS=1 \
scripts/gki/generate_defconfig.sh \
vendor/holi-qgki_defconfig
```
- In the cases of symbols being undefined when they're clearly defined in the source code, compare your `lineage_(codename).config` with the others that exist while also referencing from a complete description of your device, such as via the deviceinfohw database. Ignore config options like `CONFIG_ARCH_(X)`, and also check to see if any `# CONFIG_* is not set` conflicts with the same configs being set.


# How do I submit patches to Android Common Kernels

1. BEST: Make all of your changes to upstream Linux. If appropriate, backport to the stable releases.
   These patches will be merged automatically in the corresponding common kernels. If the patch is already
   in upstream Linux, post a backport of the patch that conforms to the patch requirements below.

2. LESS GOOD: Develop your patches out-of-tree (from an upstream Linux point-of-view). Unless these are
   fixing an Android-specific bug, these are very unlikely to be accepted unless they have been
   coordinated with kernel-team@android.com. If you want to proceed, post a patch that conforms to the
   patch requirements below.

# Common Kernel patch requirements

- All patches must conform to the Linux kernel coding standards and pass `script/checkpatch.pl`
- Patches shall not break gki_defconfig or allmodconfig builds for arm, arm64, x86, x86_64 architectures
(see  https://source.android.com/setup/build/building-kernels)
- If the patch is not merged from an upstream branch, the subject must be tagged with the type of patch:
`UPSTREAM:`, `BACKPORT:`, `FROMGIT:`, `FROMLIST:`, or `ANDROID:`.
- All patches must have a `Change-Id:` tag (see https://gerrit-review.googlesource.com/Documentation/user-changeid.html)
- If an Android bug has been assigned, there must be a `Bug:` tag.
- All patches must have a `Signed-off-by:` tag by the author and the submitter

Additional requirements are listed below based on patch type

## Requirements for backports from mainline Linux: `UPSTREAM:`, `BACKPORT:`

- If the patch is a cherry-pick from Linux mainline with no changes at all
    - tag the patch subject with `UPSTREAM:`.
    - add upstream commit information with a `(cherry-picked from ...)` line
    - Example:
        - if the upstream commit message is
```
        important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>
```
        - then Joe Smith would upload the patch for the common kernel as
```
        UPSTREAM: important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>

        Bug: 135791357
        Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
        (cherry-picked from c31e73121f4c1ec41143423ac6ce3ce6dafdcec1)
        Signed-off-by: Joe Smith <joe.smith@foo.org>
```

- If the patch requires any changes from the upstream version, tag the patch with `BACKPORT:`
instead of `UPSTREAM:`.
    - use the same tags as `UPSTREAM:`
    - add comments about the changes under the `(cherry-picked from ...)` line
    - Example:
```
        BACKPORT: important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>

        Bug: 135791357
        Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
        (cherry-picked from c31e73121f4c1ec41143423ac6ce3ce6dafdcec1)
        [ Resolved minor conflict in drivers/foo/bar.c ]
        Signed-off-by: Joe Smith <joe.smith@foo.org>
```

## Requirements for other backports: `FROMGIT:`, `FROMLIST:`,

- If the patch has been merged into an upstream maintainer tree, but has not yet
been merged into Linux mainline
    - tag the patch subject with `FROMGIT:`
    - add info on where the patch came from as `(cherry picked from commit <sha1> <repo> <branch>)`. This
must be a stable maintainer branch (not rebased, so don't use `linux-next` for example).
    - if changes were required, use `BACKPORT: FROMGIT:`
    - Example:
        - if the commit message in the maintainer tree is
```
        important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>
```
        - then Joe Smith would upload the patch for the common kernel as
```
        FROMGIT: important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>

        Bug: 135791357
        (cherry picked from commit 878a2fd9de10b03d11d2f622250285c7e63deace
         https://git.kernel.org/pub/scm/linux/kernel/git/foo/bar.git test-branch)
        Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
        Signed-off-by: Joe Smith <joe.smith@foo.org>
```


- If the patch has been submitted to LKML, but not accepted into any maintainer tree
    - tag the patch subject with `FROMLIST:`
    - add a `Link:` tag with a link to the submittal on lore.kernel.org
    - if changes were required, use `BACKPORT: FROMLIST:`
    - Example:
```
        FROMLIST: important patch from upstream

        This is the detailed description of the important patch

        Signed-off-by: Fred Jones <fred.jones@foo.org>

        Bug: 135791357
        Link: https://lore.kernel.org/lkml/20190619171517.GA17557@someone.com/
        Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
        Signed-off-by: Joe Smith <joe.smith@foo.org>
```

## Requirements for Android-specific patches: `ANDROID:`

- If the patch is fixing a bug to Android-specific code
    - tag the patch subject with `ANDROID:`
    - add a `Fixes:` tag that cites the patch with the bug
    - Example:
```
        ANDROID: fix android-specific bug in foobar.c

        This is the detailed description of the important fix

        Fixes: 1234abcd2468 ("foobar: add cool feature")
        Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
        Signed-off-by: Joe Smith <joe.smith@foo.org>
```

- If the patch is a new feature
    - tag the patch subject with `ANDROID:`
    - add a `Bug:` tag with the Android bug (required for android-specific features)

# Vibrator driver for HHG device
## How to merge the driver into kernel source tree

 1. Copy \${this_project}/drivers/hid/hid-aksys.c into \${your_kernel_root}/drivers/hid/

 2. Compare and merge \${this_project}/drivers/hid/hid-ids.h into \${your_kernel_root}/drivers/hid/hid-ids.h :
 Add the following code before the last line of this file

    ```c
		#define USB_VENDER_ID_QUALCOMM  0x0a12
		#define USB_VENDER_ID_TEMP_HHG_AKSY 0x1234
		#define USB_PRODUCT_ID_AKSYS_HHG  0x1000
    ```

 3. Merge \${this_project}/drivers/hid/Kconfig into \${your_kernel_root}/drivers/hid/Kconfig :
Add the following code before the last line of this file

		config HID_AKSYS_QRD
    		tristate "AKSys gamepad USB adapter support"
    		depends on HID
    		---help---
    		Support for AKSys gamepad USB adapter

    	config AKSYS_QRD_FF
    		bool "AKSys gamepad USB adapter force feedback support"
    		depends on HID_AKSYS_QRD
    		select INPUT_FF_MEMLESS
    		---help---
    		Say Y here if you have a AKSys gamepad USB adapter and want to
    		enable force feedback support for it.
    		
 4. Merge \${this_project}/drivers/hid/Makefile into \${your_kernel_root}/drivers/hid/Makefile :
 Add the following code at the end of this file

		obj-$(CONFIG_HID_AKSYS_QRD)	+= hid-aksys.o
		
 5. Modify your kernel's default build configuration file. Add the following two lines:

        CONFIG_HID_AKSYS_QRD=m
        CONFIG_AKSYS_QRD_FF=y
