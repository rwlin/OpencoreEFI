# Hackintosh
EFI for i3-4130 MSI B85M-P33
- OpenCore 0.9.9-Debug
- Catalina (10.15)
## OpenCore
- [Install Guide](https://dortania.github.io/OpenCore-Install-Guide)
- [Download OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)

## Utils
- [ProperTree](https://github.com/corpnewt/ProperTree) - edit .plist files
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) - For generating our SMBIOS data
- [Sanity Checker](https://sanitychecker.ocutils.me/) - OpenCore Sanity Checker

## Doc
### Finding your hardware
<details>
<summary>My Hardware</summary>

|Device|Model|
|---|---|
|Motherboard|MSI B85M-P33 V3|
|CPU|Intel(R) Core(TM) i3-4130 CPU (Haswell)|
|GPU|Intel HD Graphics 4400|
|Audio|Realtek ALC887 (Supported by [AppleALC](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs))|
|Wired|Realtek 8111G |
|USB|4xUSB3 + 6xUSB2|
</details>

### [USB Creation](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/windows-install.html)

<details><summary>Windows</summary>

- Download BaseSystem or RecoveryImage files

    ```shell
    cd /path/to/OpenCore-0.9.9-Debug/Utilities/macrecovery
    # Catalina(10.15)
    python ./macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download
    ```

    you'll either get BaseSystem or RecoveryImage files.

- [Making the installer](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/windows-install.html#making-the-installer)

</details>

### Gathering files

<details>
<summary>config.plist</summary>

```shell
cp /path/to/OpenCore-0.9.9-Debug/Docs/Sample.plist /path/to/your/EFI/OC/config.plist
```

</details>
<details>
<summary>EFI/OC/ACPI</summary>

- [SSDTs required](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#ssdts)
- [Haswell and Broadwell](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-methods/ssdt-prebuilt.html#desktop-haswell-and-broadwell) need [SSDT-PLUG-DRTNIA](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml) & [SSDT-EC-DESKTOP](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-DESKTOP.aml)

</details>
<details>
<summary>EFI/OC/Drivers/</summary>

- [HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) - Needed for seeing HFS volumes ([vs. OpenHfsPlus.efi](https://www.insanelymac.com/forum/topic/348108-differences-openhfsplusefi-and-hfsplusefi/))
- OpenRuntime.efi - used as an extension for OpenCore to help with patching boot.efi for NVRAM fixes and better memory management.
```shell
cp /path/to/OpenCore-0.9.9-Debug/X64/EFI/OC/Drivers/{OpenHfsPlus.efi,OpenRuntime.efi} /path/to/EFI/OC/Drivers/
```

</details>
<details>
<summary>EFI/OC/Kexts/</summary>

- [Lilu](https://github.com/acidanthera/Lilu/releases)  
  A kext to patch many processes, required for AppleALC, WhateverGreen, VirtualSMC and many other kexts. Without Lilu, they will not work.
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases)  
Emulates the SMC chip found on real macs, without this macOS will not boot
Alternative is FakeSMC which can have better or worse support, most commonly used on legacy hardware.
- SMCProcessor.kext  
Used for monitoring CPU temperature, doesn't work on AMD CPU based systems
- SMCSuperIO.kext  
Used for monitoring fan speed, doesn't work on AMD CPU based systems
- [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases) - Used for graphics patching DRM, boardID, framebuffer fixes, etc, all GPUs benefit from this kext.
- [AppleALC](https://github.com/acidanthera/AppleALC/releases) - Used for AppleHDA patching, allowing support for the majority of on-board sound controllers
- [RealtekTRL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
- ~~[USBInjectAll](https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/)~~
- ~~[VoodooPS2Controller.kext](https://github.com/acidanthera/VoodooPS2/releases) - PS2 keyboard~~

</details>

### plist
[Haswell](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html)

- DeviceProperties
  - Add  
    **PciRoot(0x0)/Pci(0x2,0x0) - iGPU**  
    [0D00E20A](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-hd-graphics-4200-5200-haswell-processors) - We have only 2 connectors (DSUB/DVI). 0300220D is not working
    |key|type|value|
    |---|---|---|
    |AAPL,ig-platform-id|Data|[0D00E20A]((https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-hd-graphics-4200-5200-haswell-processors))|
    |device-id|Data|12040000|

    **PciRoot(0x0)/Pci(0x1b,0x0) - AppleALC**  
    Applies AppleALC audio injection, you'll need to do your own research on which codec your motherboard has and match it with AppleALC's layout. For us, we'll be using the boot-arg alcid=xxx instead to accomplish this. alcid will override all other layout-IDs present.
    |Key|Type|Value|
    |---|---|---|
    |layout-id| Data|1|

- Kernel
  - Quirks
    |Key|Value|Comment|
    |---|---|---|
    |AppleCpuPmCfgLock|false||
    |AppleXcpmCfgLock|true||
    |DisableIoMapper|true||
    |LapicKernelPanic|false||
    |PanicNoKextDump|true||
    |PowerTimeoutKernelPanic|true||
    |XhciPortLimit|false|false for 11.3+(BigSur/Monterey/Ventura)|

- Misc
  - Debug
    |Key|Value|
    |---|---|
    |AppleDebug|true|
    |ApplePanic|true|
    |DisableWatchDog|true|
    |Target|67|

  - Security
    |Key|Value|
    |---|---|
    |AllowSetDefault|true|
    |BlacklistAppleUpdate|true|
    |ScanPolicy|0|
    |SecureBootModel|Default or Disabled|
    |Vault|Optional|

- NVRAM
  - Add  
    7C436110-AB2A-4BBB-A880-FE41995C9F82
    |Key|Type|Value|
    |---|---|---|
    |boot-args|String|-v keepsyms=1 debug=0x100 alcid=1|for debug. Can add -wegnoegpu as well. |
    |prev-lang:kbd|Data|&lt;empty&gt;|

- PlatformInfo
  - Generic
    |Key|Type|Value|
    |---|---|---|
    |MLB|String||
    |ROM|Data||
    |SystemProductName|String|iMac14,4|
    |SystemSerialNumber|String||
    |SystemUUID|String||

- UEFI
  - APFS
    |Key|Type|Value|
    |---|---|---|
    |MinDate|Integer|-1|
    |MinVersion|Integer|-1|

  - Quirks
    |Key|Value|
    |---|---|
    |IgnoreInvalidFlexRatio|true|

## BIOS

Intel BIOS settings
Note: Most of these options may not be present in your firmware, we recommend matching up as closely as possible but don't be too concerned if many of these options are not available in your BIOS
### Disable
- Fast Boot
- ~~Secure Boot~~
- Serial/COM Port
- Parallel Port
- ~~VT-d (can be enabled if you set DisableIoMapper to YES)~~
- CSM
- ~~Thunderbolt(For initial install, as Thunderbolt can - cause issues if not setup correctly)~~
- ~~Intel SGX~~
- ~~Intel Platform Trust~~
- ~~CFG Lock (MSR 0xE2 write protection)(This must be off, if you can't find the option then enable AppleCpuPmCfgLock under Kernel -> Quirks. Your hack will not boot with CFG-Lock enabled)~~
### Enable
- VT-x
- ~~Above 4G decoding~~
- ~~Hyper-Threading~~
- ~~Execute Disable Bit~~
- EHCI/XHCI Hand-off
- ~~OS type: Windows 8.1/10 UEFI Mode~~
- DVMT Pre-Allocated(iGPU Memory): 64MB
- SATA Mode: AHCI