# Motorola Edge 40 Neo (`manaus`) Development

Welcome to the **motorola-manaus** organization. This organization hosts the complete, unified source repositories for building Android 17 custom ROMs for the **Motorola Edge 40 Neo** (codename: `manaus`, SoC: MediaTek Dimensity 7020 / MT6879).

---

## Device Specifications

| Feature | Specification |
| :--- | :--- |
| **SoC** | MediaTek Dimensity 7030 (MT6879) |
| **CPU** | Octa-core (2x2.2 GHz Cortex-A78 & 6x2.0 GHz Cortex-A55) |
| **GPU** | IMG BXM-8-256 |
| **Architecture** | ARM64 (`aarch64`, 64-bit only) |
| **Display** | 6.55" Curved pOLED, 144Hz, HDR10+, 1080 x 2400 pixels |
| **Rear Cameras** | 50 MP (wide, OIS, omnidirectional PDAF) + 13 MP (ultrawide/macro) |
| **Front Camera** | 32 MP (wide) |
| **Battery** | 5000 mAh (68W TurboPower wired charging) |
| **Audio** | Stereo speakers, Dolby Atmos, Hi-Res Audio, LHDC v3/v5, LDAC, aptX HD |
| **Biometrics** | Under-display optical fingerprint scanner |

---

## Repositories

| Repository | Description | Source Tree Path |
| :--- | :--- | :--- |
| [**android_device_motorola_manaus**](https://github.com/motorola-manaus/android_device_motorola_manaus) | Device-specific makefiles, overlays & product definitions | `device/motorola/manaus` |
| [**android_device_motorola_mt6879-common**](https://github.com/motorola-manaus/android_device_motorola_mt6879-common) | Common platform tree (audio policy, HAL configs, init scripts) | `device/motorola/mt6879-common` |
| [**proprietary_vendor_motorola_manaus**](https://github.com/motorola-manaus/proprietary_vendor_motorola_manaus) | Device vendor blobs, radio firmware & patched `lk.img` *(Git LFS)* | `vendor/motorola/manaus` |
| [**proprietary_vendor_motorola_mt6879-common**](https://github.com/motorola-manaus/proprietary_vendor_motorola_mt6879-common) | Common vendor blobs + 64-bit Savitech LHDC encoders | `vendor/motorola/mt6879-common` |
| [**android_kernel_motorola_mt6879**](https://github.com/motorola-manaus/android_kernel_motorola_mt6879) | Linux kernel 5.10.x GKI source | `kernel/motorola/mt6879` |
| [**device_mediatek_sepolicy_vndr**](https://github.com/motorola-manaus/device_mediatek_sepolicy_vndr) | MediaTek vendor SEPolicy rules | `device/mediatek/sepolicy_vndr` |
| [**hardware_mediatek**](https://github.com/motorola-manaus/hardware_mediatek) | MediaTek HAL interfaces and hardware modules | `hardware/mediatek` |

---

## Local Manifest (`local_manifests/manaus.xml`)

To sync all necessary repositories into your ROM build tree at once, save the following XML to `.repo/local_manifests/manaus.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <remote name="motorola-manaus"
            fetch="https://github.com/motorola-manaus"
            revision="seventeen" />

    <!-- Device Trees -->
    <project name="android_device_motorola_manaus" path="device/motorola/manaus" remote="motorola-manaus" />
    <project name="android_device_motorola_mt6879-common" path="device/motorola/mt6879-common" remote="motorola-manaus" />

    <!-- Vendor Blobs -->
    <project name="proprietary_vendor_motorola_manaus" path="vendor/motorola/manaus" remote="motorola-manaus" />
    <project name="proprietary_vendor_motorola_mt6879-common" path="vendor/motorola/mt6879-common" remote="motorola-manaus" />

    <!-- Kernel Source -->
    <project name="android_kernel_motorola_mt6879" path="kernel/motorola/mt6879" remote="motorola-manaus" />

    <!-- MediaTek Hardware & SEPolicy -->
    <project name="device_mediatek_sepolicy_vndr" path="device/mediatek/sepolicy_vndr" remote="motorola-manaus" />
    <project name="hardware_mediatek" path="hardware/mediatek" remote="motorola-manaus" />
</manifest>
```

Then run:

```bash
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

---

## Patches: Fenrir Patched LK Support

The Motorola Edge 40 Neo uses a patched Little Kernel (`lk.img`, fenrir) to bypass boot restrictions. When booted with this patched bootloader, the kernel command line contains `fenrir=true`. 

To ensure fastbootd and recovery recognize the device as unlocked and allow dynamic partition flashing, apply the following two patches to your ROM source:

1. **`system/core`** — [Allow flashing images from fastbootd with fenrir patched LKs](https://github.com/yaap/system_core/commit/ca84e2ec0689f62ad6449dbb7f45dece5898a307)
   ```cpp
   // fastboot/device/utility.cpp in GetDeviceLockStatus():
   std::string fenrir;
   if (android::fs_mgr::GetKernelCmdline("fenrir", &fenrir) && fenrir == "true")
       return false;
   ```

2. **`bootable/recovery`** — [Return true for IsDeviceUnlocked() if fenrir=true](https://github.com/yaap/bootable_recovery/commit/b0630ff093f377c7c3158a11eae15bf4505cdc52)
   ```cpp
   // recovery_main.cpp in IsDeviceUnlocked():
   std::string fenrir;
   if (android::fs_mgr::GetKernelCmdline("fenrir", &fenrir) && fenrir == "true")
       return true;
   ```

---

## Community & Credits

* Motorola Mobility LLC for the stock kernel and firmware releases.
* The LineageOS Project & RisingOS team.
* Original device bringup maintainer (`rexix01`).
