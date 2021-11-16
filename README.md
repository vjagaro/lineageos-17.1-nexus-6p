# Install LineageOS 17.1 on Nexus 6P

The following are notes for installing [Unofficial LineageOS 17.1 for Nexus 6P (angler)].
This was done on August 5, 2020.

## Setup Phone and Host Computer

1. [Enable USB Debugging] on the phone.

2. [Turn on OEM unlocking] on the phone in **Developer Options**.

3. Install the `adb`, `fastboot` and `unzip` on the host computer:

   ```sh
   sudo apt install -y android-tools-adb android-tools-fastboot unzip
   ```

4. Connect the phone to the computer via USB.

5. Confirm that you can see the device:

   ```sh
   adb devices
   ```

## Unlock device

1.  Reboot into `bootloader`:

    ```sh
    adb reboot bootloader
    ```

2.  After rebooting into `bootloader`, look at the lower left corner of the phone
    and look for a **Device is LOCKED** message. If so, you will need to unlock the
    device:

    ```sh
    fastboot flashing unlock
    ```

    Then, choose **Yes** on the phone.

## Flash AOSP System Image

1.  Download the latest [Google Android Angler Image], then verify:

    ```sh
    aosp_dir="angler-opm7.181205.001"
    aosp_zip="angler-opm7.181205.001-factory-b75ce068.zip"
    aosp_sha256="b75ce068f23a0e793805f80fccbc081eca52861ef5eb080c47f502de4c3f9713"
    echo "$aosp_sha256  $aosp_zip" | shasum -c
    ```

4.  Unzip and install:

    ```sh
    rm -rf "$aosp_dir"
    unzip "$aosp_zip"
    cd "$aosp_dir"
    unzip *.zip
    fastboot flash bootloader bootloader-angler-angler-03.84.img
    fastboot reboot bootloader
    sleep 5
    fastboot flash radio radio-angler-angler-03.88.img
    fastboot reboot bootloader
    sleep 5
    fastboot flash vendor vendor.img
    fastboot reboot bootloader
    sleep 5
    fastboot flash system system.img
    fastboot flash:raw boot boot.img
    fastboot reboot bootloader
    cd ..
    ```

## Flash TWRP

1. Download [TWRP for `angler`], then verify:

   ```sh
   twrp_img="twrp-3.5.0_9-0-angler.img"
   twrp_sha256="0ee870432b0ed78753701b3a105e1ef6266be4afbaec15d96197db5821f24a46"
   echo "$twrp_sha256  $twrp_img" | shasum -c
   ```

2. Flash TWRP and reboot:

   ```sh
   fastboot flash recovery "$twrp_img"
   fastboot reboot bootloader
   ```

## Wipe the Phone

1. From the bootloader screen, press the **Down button** twice and press
   **Power** to start **Recovery** (TWRP). When asked to **Keep System Read
   only?**, choose **Swipe to Allow Modification**.

2. Choose **Wipe**, **Format Data**, then type **yes** to confirm. After the
   format, press the Home icon.

3. Choose **Wipe**, **Advanced Wipe**. Select **System**, **Data**, **Cache**
   and **Internal Storage**. Choose **Swipe to Wipe**. After the wipe, press the
   Home Icon.

4. Choose **Reboot**, then **Recovery**, then **Swipe to Reboot**.

## Install LineageOS

1. Download the latest [unofficial LineageOS build for `angler`] and verify:

   ```sh
   lineageos_zip="lineage-17.1-20200728-UNOFFICIAL-angler.zip"
   lineageos_sha256="d073e33a0e0007ba57cc7c7b11a0852c6335e7cef1bb7490f8b6f99ab5309c70"

   echo "$lineageos_sha256  $lineageos_zip" | shasum -c
   ```

2. Download the latest [Open GApps] (ARM64, 10.0, pico):

   ```sh
   open_gapps_zip="open_gapps-arm64-10.0-pico-20200805.zip"
   open_gapps_sha256="2763c9860f06d2eb2a01cd1e1124142b8cba19ca1e9224fd5cf6b7bc5d044214"
   open_gapps_dir="${open_gapps_zip#*pico-}"
   open_gapps_dir="${open_gapps_dir%.zip}"

   wget -O "$open_gapps_zip" "https://netactuate.dl.sourceforge.net/project/opengapps/arm64/$open_gapps_dir/$open_gapps_zip"
   echo "$open_gapps_sha256  $open_gapps_zip" | shasum -c
   ```

3. On the phone, go into `bootloader`. From `bootloader`, press the **Down
   button** twice and press **Power** to start **Recovery** (TWRP) again. When
   asked to **Keep System Read only?**, choose **Swipe to Allow Modification**.

4. Choose **Advanced**, **ADB Sideload**, then **Swipe to Start Sideload**.
   Then, install the [LineageOS] image:

   ```sh
   adb sideload "$lineageos_zip"
   ```

5 Choose **Back**, then repeat the above process for [Open GApps].

    ```sh
    adb sideload "$open_gapps_zip"
    ```

6. Choose **Reboot System** and **Swipe to Reboot**.

7. Wait patiently for the first boot (this might take several minutes...)

## Notes on Magisk

I wasn't able to get Magisk working but I kept the notes here...

1. Download the latest [Magisk]:

   ```sh
   magisk_version="20.4"
   magisk_sha256="5795228296ab3e84cda196e9d526c9b8556dcbf5119866e0d712940ceed1f422"
   magisk_zip="Magisk-v$magisk_version.zip"
   magisk_url="https://github.com/topjohnwu/Magisk/releases/download/v$magisk_version/$magisk_zip"

   wget -O "$magisk_zip" "$magisk_url"
   echo "$magisk_sha256  $magisk_zip" | shasum -c
   ```

   I received an error during the install of [Magisk] that it was unable to
   `mount /vendor`. I'm not sure about the resolution...

[unofficial lineageos 17.1 for nexus 6p (angler)]: https://forum.xda-developers.com/nexus-6p/orig-development/rom-lineageos-17-0-nexus-6p-angler-t4012099
[enable usb debugging]: https://developer.android.com/studio/debug/dev-options
[google android angler image]: https://developers.google.com/android/images#angler)
[google flash system image instructions]: https://developers.google.com/android/images
[turn on oem unlocking]: https://en.droidwiki.org/wiki/OEM-Unlock
[twrp for `angler`]: https://dl.twrp.me/angler/
[open gapps]: https://opengapps.org/
[lineageos]: https://lineageos.org/
[magisk]: https://github.com/topjohnwu/Magisk
[unofficial lineageos build for `angler`]: https://drive.google.com/drive/folders/1NS8ishNapMkjqZZAN2ypRs6tPThycQop
