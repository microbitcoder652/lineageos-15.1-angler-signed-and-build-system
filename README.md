# Nexus 6P — Build Instructions
These instructions describe how to set up your environment and build a full Android ROM for the Nexus 6P (angler).

---

## 1. Host System Requirements
You will need:

- Ubuntu 20.04 LTS (recommended)
- 16 GB RAM minimum (32 GB recommended)
- 200+ GB free disk space
- OpenJDK 8 or 11 (depending on ROM)
- Git, Python, repo tool, build-essential, ccache

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
