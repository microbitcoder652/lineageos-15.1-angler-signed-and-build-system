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

Absolutely, Arya — I can turn your entire **Build LineageOS.txt** into a clean, structured, GitHub‑ready **Markdown README**.  
Below is a fully converted, polished, and properly formatted **Markdown version** of the document you provided.  
Everything is preserved, but now it’s readable, navigable, and ready for your Nexus 6P build repo.

---

# ✅ **Build LineageOS — Markdown Version**

```markdown
# Build LineageOS

> **Note:** You only need to do these steps once.  
> If you already prepared your build environment and downloaded the source code, skip to **Prepare the device‑specific code**.

---

## 1. Install the platform-tools

If you haven’t previously installed `adb` and `fastboot`, download them from Google and extract:

```bash
unzip platform-tools-latest-linux.zip -d ~
```

Add them to your PATH by editing `~/.profile`:

```bash
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi
```

Reload your environment:

```bash
source ~/.profile
```

---

## 2. Install the build packages

Install required packages:

```
bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs \
gnupg gperf imagemagick protobuf-compiler python3-protobuf lib32readline-dev \
lib32z1-dev libdw-dev libelf-dev libgnutls28-dev lz4 libsdl1.2-dev libssl-dev \
libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc \
zip zlib1g-dev
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

| LineageOS Version | Required JDK |
|------------------|--------------|
| 18.1+ | OpenJDK 11 (included) |
| 16.0–17.1 | OpenJDK 9 (included) |
| 14.1–15.1 | OpenJDK 8 (`openjdk-8-jdk`) |
| 11.0–13.0 | OpenJDK 7 |

For JDK 8 builds, remove TLSv1 and TLSv1.1 from:

```
/etc/java-8-openjdk/security/java.security
```

---

## 4. Python Requirements

| LineageOS Version | Python |
|------------------|--------|
| 17.1+ | Python 3 |
| 11.0–16.0 | Python 2 |

If you need Python 2:

```bash
virtualenv --python=python2 ~/.lineage_venv
source ~/.lineage_venv/bin/activate
```

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

Ensure `~/bin` is in PATH:

```bash
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

Reload:

```bash
source ~/.profile
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
ccache -M 50G
```

Enable compression:

```bash
ccache -o compression=true
```

---

## 9. Configure Jack (for 14.1–15.1 builds)

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
breakfast angler
```

If vendor files are missing, extract blobs first.

---

## 14. Extract Proprietary Blobs

Connect your Nexus 6P (ADB + root enabled):

```bash
cd ~/android/lineage/device/huawei/angler
./extract-files.py
```

Or:

```bash
./extract-files.sh
```

Blobs will populate:

```
~/android/lineage/vendor/huawei
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

Reboot to bootloader:

```bash
adb reboot bootloader
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

For help:

- IRC: `#LineageOS-dev` on Libera.Chat  
- Discord: bridged with IRC

```

---

If you want, I can **tailor this Markdown specifically for your GitHub repo**, including:

- your signing workflow  
- your Python signing scripts  
- your WSL‑based environment  
- your exact directory layout  
- your Nexus 6P build automation  

Just say the word and I’ll refine it to perfection.
Install base packages:

```bash
sudo apt update
sudo apt install bc bison build-essential curl flex g++-multilib gcc-multilib git \
    gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev \
    liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils \
    lzop openjdk-8-jdk pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

---

## 2. Create the Working Directory

```bash
mkdir -p ~/android/rom
cd ~/android/rom
```

---

## 3. Initialize the Repo

Replace the manifest URL and branch with the ROM you are building (LineageOS example shown):

```bash
repo init -u https://github.com/LineageOS/android.git -b lineage-15.1
```

---

## 4. Sync the Source

```bash
repo sync --force-sync --current-branch --no-tags --no-clone-bundle -j$(nproc)
```

This may take a long time depending on your connection.

---

## 5. Clone Device Trees

```bash
git clone https://github.com/<yourname>/android_device_huawei_angler device/huawei/angler
git clone https://github.com/<yourname>/android_kernel_huawei_angler kernel/huawei/angler
git clone https://github.com/<yourname>/android_vendor_huawei_angler vendor/huawei/angler
```

Adjust URLs to match your repos.

---

## 6. Set Up the Build Environment

```bash
source build/envsetup.sh
lunch lineage_angler-userdebug
```

---

## 7. Start the Build

```bash
mka bacon -j$(nproc)
```

Output will appear in:

```
out/target/product/angler/
```

Artifacts include:

- `lineage-*.zip` — flashable ROM
- `boot.img`
- `system.img`
- `vendor.img`

---

## 8. Signing (Optional but Recommended)

If you are using private signing keys:

```bash
./build/tools/releasetools/sign_target_files_apks \
    -o -d ~/keys \
    out/target/product/angler/obj/PACKAGING/target_files_intermediates/*.zip \
    signed-target_files.zip

./build/tools/releasetools/ota_from_target_files \
    -k ~/keys/releasekey \
    signed-target_files.zip \
    signed-ota.zip
```

---

## 9. Flashing the ROM

Reboot to bootloader:

```bash
adb reboot bootloader
```

Flash images:

```bash
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot reboot
```

Or flash the OTA ZIP from recovery.

---

## 10. Troubleshooting

### Build Fails at Jack Server
```bash
jack-admin kill-server
jack-admin start-server
```

### Out of Disk Space
Ensure you have at least 200 GB free.

### Missing Proprietary Blobs
Extract from a running device:

```bash
adb pull /system vendor/
```

---

## License
MIT or whatever applies to your project.
```
