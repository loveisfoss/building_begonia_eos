# Building /e/ os for begonia

1. Create your working directory, e.g. "~/building/begonia/eos" and open a terminal window on this location.

2. Initialize your local repository. Examples:
```
repo init -u https://gitlab.e.foundation/e/os/releases.git -b refs/tags/v0.9.4-pie
repo init -u https://gitlab.e.foundation/e/os/releases.git -b v1-q
```

3. Create/edit your local manifest in ".../.repo/local_manifests/eos.xml". At the time of writing, it should like this:
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <project name="LineageOS/android_device_xiaomi_begonia" path="device/xiaomi/begonia" remote="github" />
  <project name="LineageOS/android_kernel_xiaomi_mt6785" path="kernel/xiaomi/mt6785" remote="github" />
  <project name="LineageOS/android_device_xiaomi_mt6785-common" path="device/xiaomi/mt6785-common" remote="github" />
  <project name="LineageOS/android_hardware_mediatek" path="hardware/mediatek" remote="github" />
  <project name="LineageOS/android_device_mediatek_sepolicy" path="device/mediatek/sepolicy" remote="github" />
  <project name="HyperTeam/proprietary_vendor_firmware" path="vendor/firmware" remote="github" />
  <project name="FishOnTheGround/proprietary_vendor_xiaomi" path="vendor/xiaomi" remote="github" />
  <project path="vendor/e" name="steadfasterX/android_vendor_e" remote="e" revision="v1-q" />

</manifest>
```
Note: I've used my own repo "FishOnTheGround/proprietary_vendor_xiaomi" because I don't know how to include gitlab in the local_manifest.xml right now. The really needed repo here is: https://gitlab.com/the-muppets/proprietary_vendor_xiaomi/-/tree/lineage-17.1 and the folders that are needed are "begonia" and "mt6785-common". So, check every time, before building, to see if the needed folders/files in the "the-muppets" repo have been updated.

4. Sync the repos
```
repo sync -j8
```
The resulting files in your work directory should be >100G in size.

5. Set up /e/
In your working directory, go to ".../device/xiaomi/begonia/vendorsetup.sh". Create it if it doesn't exist. Example of parameters that can be included here:
```
export EOS_DEVICE=begonia
export EOS_SIGNATURE_SPOOFING=restricted
export EOS_BRANCH_NAME=v1-q
export EOS_RELEASE_TYPE=UNOFFICIAL
export EOS_SIGN_BUILDS=true
export TEMPORARY_DISABLE_PATH_RESTRICTIONS=true
```

For their default values look into ".../vendor/e/vendorsetup.sh". Do not change variables in ".../vendor/e/vendorsetup.sh" though. Just find out what the proper variable name is and set that EOS_xxx variable within your ".../device/xiaomi/begonia/vendorsetup.sh".

Note: Not sure that the last parameter belongs here. It didn't cause any problems though. See "Disable path restrictions" down below.

6. lineage.mk

In order to make use of SteadfasterX's /e/ vendor repo you have to include it in your ".../device/xiaomi/begonia/lineage_begonia.mk". Add this:
```
# inherit vendor e
$(call inherit-product, vendor/e/config/common.mk)
```

7. Proceed with building. Use these commands:
```
source build/envsetup.sh
lunch lineage_begonia-userdebug
mka eos
```


# Permissive SELinux (needed at the tme of writing)
Open ".../device/xiaomi/mt6785-common/BoardConfigCommon.mk" and edit line 64. \
Instead of:
```
BOARD_KERNEL_CMDLINE := bootopt=64S3,32N2,64N2
```
Make it lok like this:
```
BOARD_KERNEL_CMDLINE := bootopt=64S3,32N2,64N2 androidboot.selinux=Permissive
```
Source: https://review.lineageos.org/c/LineageOS/android_device_xiaomi_mt6785-common/+/298385

# Disable path restrictions (don't know if needed and which of the following ways is the one that's probaly working. I implemented both and the build succeeded.) 

A. Set 
```
export TEMPORARY_DISABLE_PATH_RESTRICTIONS=true
```
in your build environment. I added it in ".../device/xiaomi/begonia/vendorsetup.sh".

B. One can modify ".../build/make/core/config.mk" to temporarily allow using PATH by commenting out this line:
```
$(KATI_obsolete_var PATH,Do not use PATH directly. See $(CHANGES_URL)#PATH)
```

# Possible hinders
When trying to build you may fail sometimes at the beginning because of various reasons. 

* ccache \
Had to install this package. One thing that I had to do was create a "/ccache/eos" directory on my computer as root and then chown it to my normal user.

* Some other packages that turned out to be necesasry for the building process: 
```
python2
m4
libncurses5
bc
zip
```
