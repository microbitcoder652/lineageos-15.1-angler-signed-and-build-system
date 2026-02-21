# Nexus 6P — Build Instructions
These instructions describe how to set up your environment and build a full Android ROM for the Nexus 6P (angler).

These instructions will hopefully assist you in unlocking the bootloader (if necessary), and then downloading the required tools as well as the very latest source code for LineageOS (based on the Android operating system) for your device. Using these, you can build both a LineageOS installation zip and a LineageOS Recovery image and install them on your device.

It is difficult to say how much experience is needed to follow these instructions. These steps shouldn’t require a PhD in software development. Some readers will have no difficulty and breeze through the steps easily. Others may struggle with the most basic operation. Because people’s experiences, backgrounds, and intuitions differ, it may be a good idea to read through just to ascertain whether you feel comfortable or are getting over your head.

Remember, you are responsible for the risks of trying this, but you will reap the rewards! It’s pretty satisfying to boot into a fresh operating system you baked at home :). And once you’re an Android-building ninja, there will be no more need to wait for “nightly” builds from anyone. You will have the skills to build a full operating system from code and install it on a running device, whenever you want.
---

## 1. Host System Requirements
You will need:

- A Google Nexus 6P.
- A relatively recent x86_64 computer:
   - Linux, macOS, or Windows - these build instructions are only tested using Ubuntu 20.04 LTS, so we recommend going with that.
   - 8 GB of RAM minimum. I recommend 16 GB, but it works with 8. 
   - 130 GB of free space minimum. You might require more free space for enabling ccache or building for multiple devices. Using SSDs results in considerably faster build        times than traditional hard drives.
- A decent internet connection and reliable electricity. :)

# Build LineageOS

> **Note:** You only need to do these steps once.  
> If you already prepared your build environment and downloaded the source code, skip to **Prepare the device‑specific code**.

---

## 1. Install the platform-tools

If you haven’t previously installed `adb` and `fastboot`, download them from Google and extract:

```bash
curl -o platform-tools-latest-linux.zip https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip -d ~
```

Add them to your PATH by editing `~/.bashrc`:

```bash
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi
```

Reload your environment:

```bash
source ~/.bashrc
```

---

## 2. Install the build packages

Install required packages:

```bash
sudo apt update
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs \
gnupg gperf imagemagick protobuf-compiler python3-protobuf lib32readline-dev \
lib32z1-dev libdw-dev libelf-dev libgnutls28-dev lz4 libsdl1.2-dev libssl-dev \
libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc \
zip zlib1g-dev python-is-python2
```

### Ubuntu 23.10+  
Install `libncurses5` manually:

```bash
wget https://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2_amd64.deb && sudo dpkg -i libtinfo5_6.3-2_amd64.deb && rm -f libtinfo5_6.3-2_amd64.deb
wget https://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libncurses5_6.3-2_amd64.deb && sudo dpkg -i libncurses5_6.3-2_amd64.deb && rm -f libncurses5_6.3-2_amd64.deb
```

### Ubuntu < 23.10  
Install:

```
lib32ncurses5-dev libncurses5 libncurses5-dev
```

### Ubuntu < 20.04  
Also install:

```
libwxgtk3.0-dev
```

### Ubuntu < 16.04  
Install:

```
libwxgtk2.8-dev
```

---

## 3. Java Requirements


For JDK 8 builds, remove TLSv1 and TLSv1.1 from:

```
/etc/java-8-openjdk/security/java.security
```

---

## 4. Python Requirements

After downloading Python 2, run:

```bash
python --version
python2 --version
python3 --version
```
You should get:

```
Python 2.7.x
Python 2.7.x
Python 3.8.x
```
If not, run

```bash
sudo update-alternatives --config python
sudo update-alternatives --config python2
```
Choose Python 2 for those 2 popup screens

---

## 5. Create Directories

```bash
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

---

## 6. Install the repo tool

```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

Ensure `~/bin` is in `~/.bashrc`:

```bash
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

Reload:

```bash
source ~/.bashrc
```

---

## 7. Configure git

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git lfs install
git config --global trailer.changeid.key "Change-Id"
```

---

## 8. Enable ccache (recommended)

```bash
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
```

Add to `~/.bashrc`.

Set cache size:

```bash
ccache -M 25G
```

---

## 9. Configure Jack

```bash
export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4G"
```

Add to `~/.bashrc`.

---

## 10. Initialize LineageOS Source

Supported Nexus 6P branches:

- `cm-14.1`
- `lineage-15.1`

Initialize:

```bash
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-15.1 --git-lfs --no-clone-bundle
```

---

## 11. Sync the Source

```bash
repo sync
```

---

## 12. Prepare the Build Environment

```bash
cd ~/android/lineage
source build/envsetup.sh
croot
```

---

## 13. Prepare Device-Specific Code

```bash
rm -rf device/huawei/angler
rm -rf kernel/huawei/angler
rm -rf vendor/huawei
git clone https://github.com/microbitcoder652/lineageos-15.1-angler-signed-and-build-system device
git clone https://github.com/microbitcoder652/lineageos-15.1-angler-signed-and-build-system kernel
git clone https://github.com/microbitcoder652/lineageos-15.1-angler-signed-and-build-system vendor
breakfast angler
```

---

## 15. Start the Build

```bash
croot
brunch angler
```

---

## 16. Find Build Output

```bash
cd $OUT
```

Important files:

- `recovery.img`
- `lineage-15.1-YYYYMMDD-UNOFFICIAL-angler.zip`

---

## 17. Install the Build

Plug in the Google Nexus 6P and reboot to bootloader:

```bash
adb reboot bootloader
```

Unlock the bootloader

```bash
fastboot oem unlock
```

Flash:

```bash
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot reboot
```

Or flash the ZIP via recovery.

---

## 🎉 Success!

You’ve built LineageOS for the Nexus 6P.


## 10. Troubleshooting

### Build Fails at Jack Server
```bash
jack-admin kill-server
jack-admin start-server
```

### Out of Disk Space
Ensure you have at least 130 GB free.

### Missing Proprietary Blobs
Extract from a running device:

```bash
adb pull /system vendor/
