
# Building a Custom ROM

This is a very beginner friendly tutorial on how to build a Custom ROM for your device. For this tutorial I will be taking AOSP Extended ROM for OnePlus 7 as an example.
## Requirements

- Your device with USB cable (OnePlus 7 for example)
- x64-bit decently powerful Linux PC or server (Recommended: 6 or more cores with 32GB RAM, although 16GB should do the job. HDD or SSD storage of 350GB or more). I will be considering Ubuntu to build in this tutorial.
- Unlimited internet connection
- Basic Git and GitHub knowledge. You will need some advanced knowledge too which you will learn along with time.

## Let's begin
## 1. Download and install platform-tools


Download the platform-tools for Linux from [here](https://developer.android.com/studio/releases/platform-tools).
Then go to the downloads location, right click in the directory and open in terminal. Use this command to unzip:
```bash
unzip <platform_tools_file_name>.zip -d ~
```
Rename "<platform_tools_file_name>" with the original file name you downloaded. Now you have to add adb and fastboot to your PATH. In the same terminal enter this:
```bash
cd ~/
gedit ~/.profile
```
A text file window will open-up. Just enter the following text at the bottom of the text file, save it and close.
```bash
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi
```
Then, run this to update your environment.
```bash
source ~/.profile
```
## 2. Install the build packages
Several packages are needed to build a Custom ROM and all those can be installed easily by using Akhil Narang's scripts. Run this:
```bash
cd ~/
git clone https://github.com/akhilnarang/scripts
cd scripts
./setup/android_build_env.sh
```
## 3. Create the directories
You’ll need to set up some directories in your build environment
To create them:
```bash
mkdir -p ~/bin
mkdir -p ~/android
```
The ~/bin directory will contain the git-repo tool (commonly named “repo”) and the ~/android directory will contain the source code of the Custom ROM


## 4. Install the repo command
Enter the following to download the repo binary and make it executable (runnable):
```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
## 5. Configure git
Given that repo requires you to identify yourself to sync Android, run the following commands to configure your git identity:
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
## 6. Initialize the source repository and download
Get into the android directory that we created and initialize the branch of repo that you wish to build. You can find the link to initialize the repo in the manifest of the particular ROM. As I am considering AEX ROM for example, you can look at their manifest from [here](https://github.com/AospExtended/manifest).
```bash
cd ~/android
repo init -u git://github.com/AospExtended/manifest.git -b 12.x
```
Download the source code by this command:
```bash
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```
Note that repo sync will take time depending on your internet connection.

## 7. Prepare the device-specific code
To build a ROM for a specific device, we usually need three device trees i.e., device trees, kernel and vendor trees. 
For OnePlus 7 we have 3, 1 and 2 directories respectively in device trees, kernel and vendor tree. The number of directories might vary from device to device. You can easily find that by checking your device specific trees from Lineage OS GitHub (if available). Note that device codename for OnePlus 7 is guacamoleb.  You need to clone all the directories below from GitHub into your source code. For OnePlus 7 the directories are:
## Device trees
- [android/device/oneplus/guacamoleb](https://github.com/shantanu-sarkar/android_device_oneplus_guacamoleb)
- [android/device/oneplus/sm8150-common](https://github.com/AospExtended-Devices/device_oneplus_sm8150-common)
- [android/device/oneplus/common](https://github.com/AospExtended-Devices/device_oneplus_common) <br />
Clone them to the respective directories
and rename the folders correctly. For example rename "android_device_oneplus_guacamoleb" to "guacamoleb" in "android/device/oneplus/" directory after cloning it from GitHub.
Click the directories to see the trees that I use for my build. If you are adapting trees from Lineage OS, you need to make a few changes in the trees as in [here](https://github.com/shantanu-sarkar/android_device_oneplus_guacamoleb/commit/1d83a5ca546a9ebb20175eb156fbf9888e4dc2d6) and [here](https://github.com/AospExtended-Devices/device_oneplus_sm8150-common/commit/813b0320384b95c5461b2776388c561a64868864). Refer other trees of your device to know what exact changes you need to make. The changes in this tutorial will for be same for your device.
## Kernel
- [android/kernel/oneplus/sm8150](https://github.com/AospExtended-Devices/kernel_oneplus_sm8150)
## Vendor Trees
- [android/vendor/oneplus/guacamoleb](https://github.com/shantanu-sarkar/android_vendor_oneplus_guacamoleb)
- [android/vendor/oneplus/sm8150-common](https://github.com/shantanu-sarkar/android_vendor_oneplus_sm8150-common)

## 8.Building the ROM
Next we can start building the ROM by using the following commands. These commands might be different for different ROMs. Check the ROM manifest for exact commands.
The usual format is:
```bash
source build/envsetup.sh
lunch aosp_device_codename-userdebug
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
m aex -j$(nproc --all) | tee log.txt
```
Above mentioned 3rd, 4th and 5th lines are only for first build. It will set ccache for building ROMs."50G" is to metion the amount of ccache you are allocating to build the ROM. 30 to 50 GB should be enough for building ROM for one device. ccache will increase the speed of building the ROM.
For OnePlus 7 the following commands to be used fot first build.
```bash
source build/envsetup.sh
lunch aosp_guacamoleb-userdebug
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
m aex -j$(nproc --all) | tee log.txt
```
For second build onwards:
```bash
source build/envsetup.sh
lunch aosp_guacamoleb-userdebug
m aex -j$(nproc --all) | tee log.txt
```
Now the build might take 4+ hours depending on your PC power. First build takes long time. Second buid onwards, time will significantly get reduced. To clean build a ROM for later builds use this command after calling the "lunch" command:
```bash
make clobber
```
For a dirty build, use:
```bash
make installclean
```
Dirty build is recommended till the android version changes.

## Credits
- [Lineage OS](https://lineageosroms.com/guacamoleb/)
- [Pixel Experience](https://wiki.pixelexperience.org/devices/guacamoleb/build/)
- [Akhil Narang](https://github.com/akhilnarang)
