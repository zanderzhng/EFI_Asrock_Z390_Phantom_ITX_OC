# Part List
Item | Model | Notes
--- | --- | --- |
CPU | Intel i7 8700
MotherBoard | ASRock Z390 Phantom Gaming-ITX/ac
WIFI / BLE | BCM94360CS2 | With M.2. adapter
Memory | Asgard 32GB * 2 | OC doesn't require remapping 32GB single memory like Clover did.
# Configs
## BIOS Settings (BIOS Version 4.20)
Item | Default Value | My Value | Reason and Result
--- | --- | --- | ---
XHCI Hand-off | Disabled | **Enabled** | Boot stalls with USB devices connected. Seems to be the only necessary change in BIOS.
CSM | Enabled | Enabled | Everyone instruction says disable  it but it seems to be no harm except for the resolution during booting.
VT-d | Enabled | Enabled | Enable quirk ```DisableIoMapper``` then no need to disable VT-d.
## OpenCore Config Details
**Not all the configs were explained here**
### ACPI
#### ACPI SSDT Patch
- SSDT-PLUG ( fix XCPM )

   XCPM was said to bring better performance ( or sth blah blah ). At least it's harmless.
   
   You could verify it by ```sysctl -n machdep.xcpm.mode```

   You might need to change PR00 to CPU0, depending on your MB model. You could query it by ```ioreg -p IODeviceTree -c IOACPIPlatformDevice -k cpu-type -k clock-frequency | egrep name | sed -e 's/ *[-|="<a-z>]//g'```

   You might need to enable Intel SpeedStep in your BIOS.
   
   Reference: [SKL+平台XCPM+HWP完整原生电源管理探究](https://www.misonsky.cn/102.html), [macOS Native CPU/IGPU Power Management](https://www.tonymacx86.com/threads/macos-native-cpu-igpu-power-management.222982/), [SSDT-PLUG.dsl](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-PLUG.dsl)

- SSDT-PPMC ( fix Preferences - Energy Saver options)

    This SSDT fix seems to be harmless. I'll look into it later.

    You could verify it by looking into Preferences.
#### ACPI Hotpatch
- RTC Fix
    
    macOS requires RTC._STA to return 0x0F. This hot patch replaced the ```If ((STAS == One))``` of RTC._STA with ```If ((0xFF || 0xFFFF))``` , so the RTC._STA will always return 0x0F.
    
    If you are using this hotpatch and failed to boot into other systems, you may replace this patch with either ```SSDT-AWAC``` or ```SSDT-RTC0```. You should use only one of them.
### Kernel
#### Quirk
- DisableIoMapper = YES.
>This option is a preferred alternative to dropping DMAR ACPI table and disabling VT-d in ﬁrmware preferences, which does not break VT-d support in other systems in case they need it.