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
  
  <remote name="gitlab" fetch="https://gitlab.com" />

  <project name="LineageOS/android_device_xiaomi_begonia" path="device/xiaomi/begonia" remote="github" />
  <project name="LineageOS/android_device_xiaomi_begoniain" path="device/xiaomi/begoniain" remote="github" />
  <project name="LineageOS/android_kernel_xiaomi_mt6785" path="kernel/xiaomi/mt6785" remote="github" />
  <project name="LineageOS/android_device_xiaomi_mt6785-common" path="device/xiaomi/mt6785-common" remote="github" />
  <project name="LineageOS/android_hardware_mediatek" path="hardware/mediatek" remote="github" />
  <project name="LineageOS/android_device_mediatek_sepolicy" path="device/mediatek/sepolicy" remote="github" />
  <project name="HyperTeam/proprietary_vendor_firmware" path="vendor/firmware" remote="github" />
  <project name="the-muppets/proprietary_vendor_xiaomi" path="vendor/xiaomi" remote="gitlab" revision=lineage-17.1 />
  <project path="vendor/e" name="steadfasterX/android_vendor_e" remote="e" revision="v1-q" />

</manifest>
```

4. Sync the repos
```
repo sync -j8
```
The resulting files in your work directory should be ~200G in size.

5. Set up /e/
In your working directory, go to ".../device/xiaomi/begonia/vendorsetup.sh". Create it if it doesn't exist. Example of parameters that can be included here:
```
export EOS_DEVICE=begonia
export EOS_SIGNATURE_SPOOFING=restricted
export EOS_BRANCH_NAME=v1-q
export EOS_RELEASE_TYPE=UNOFFICIAL
export EOS_SIGN_BUILDS=true
```

For their default values look into ".../vendor/e/vendorsetup.sh". Do not change variables in ".../vendor/e/vendorsetup.sh" though. Just find out what the proper variable name is and set that EOS_xxx variable within your ".../device/xiaomi/begonia/vendorsetup.sh".

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

# Useful Information

## Permissive SELinux (needed at the tme of writing)
Open ".../device/xiaomi/mt6785-common/BoardConfigCommon.mk" and edit line 64. \
Instead of:
```
BOARD_KERNEL_CMDLINE := bootopt=64S3,32N2,64N2
```
Make it lok like this:
```
BOARD_KERNEL_CMDLINE := bootopt=64S3,32N2,64N2 androidboot.selinux=permissive
```
Source: https://review.lineageos.org/c/LineageOS/android_device_xiaomi_mt6785-common/+/298385

## Disable path restrictions
This one is probably needed for the build to succeed. One can modify ".../build/make/core/config.mk" to temporarily allow using PATH by commenting out this line:
```
$(KATI_obsolete_var PATH,Do not use PATH directly. See $(CHANGES_URL)#PATH)
```

## Possible hinders
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

## Solution to version naming problem
Repo sync does not touch vendor/lineage.

1. In some previous build, the past vendor/e had changed the version number in vendor/lineage (which it does not anymore, because - as I understand - it detects the previous change and leaves it untouched). That means the repo is in an "unclean" state after building.
2. repo sync will only try to update vendor/lineage if there are changes online which need to be pulled
3. There are rarely online changes in vendor/lineage so that does not happen often
4. IF there are changes in vendor/lineage and one starts repo sync it would FAIL syncing (even when using force option).

All the above is the standard repo behavior so nothing special with vendor/e or vendor/lineage actually.

The version naming can easily be updated by executing the following command before doing a repo sync.
```
cd .../vendor/lineage 
git reset --hard 
```

## swap
Remember that building an android rom is a memory demanding process. Enable swap on your system in order not to face problems while building!
