# Part List
| Item | Model | Notes |
| --- | --- | --- |
| CPU | Intel i7 8700 |  |
| MotherBoard | ASRock Z390 Phantom Gaming-ITX/ac |  |
| WIFI / BLE | BCM94360CS2 | With adapter |
| Memory | Asgard 32GB * 2 | OC doesn't require remapping 32GB single memory like Clover did. |
# Configs
## BIOS Settings
Based on default settings.
| Item | Location | Value | Reason and Result |
| --- | --- | --- | --- |
| CSM | Boot | Disabled |  |
| VT-d | Advanced - CPU | Disabled |  |
| XHCI Hand-off | Advanced -  USB | Enabled |  |
## BootLoader
OpenCore
### ACPI Fix
#### ACPI SSDT Patch
- SSDT-PLUG ( fix XCPM )

   XCPM was said to bring better performance ( or sth related). I can't say for sure but so far it's harmless.
   
   You could verify it by ```sysctl -n machdep.xcpm.mode```

   You might need to change the code inside from PR00 to CPU0, depending on your MB model.
   
   Reference: [SKL+平台XCPM+HWP完整原生电源管理探究](https://www.misonsky.cn/102.html)

- SSDT-PPMC ( fix Preferences - Energy Saver options)

    This SSDT fix seems to be harmless. I'll look into it later.

    You could verify it by looking into Preferences.
#### ACPI Hotpatch
- RTC Fix
    
    macOS requires RTC._STA to return 0x0F. This hot patch replaced the ```If ((STAS == One))``` of RTC._STA with ```If ((0xFF || 0xFFFF))``` , so the RTC._STA will always return 0x0F.
    
    If you are using this hotpatch and failed to boot into other systems, you may replace this patch with either ```SSDT-AWAC``` or ```SSDT-RTC0```. You should use only one of them.
