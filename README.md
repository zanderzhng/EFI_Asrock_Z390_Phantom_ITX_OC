# Part List
Item | Model | Notes
--- | --- | --- |
CPU | Intel i7 8700
MotherBoard | ASRock Z390 Phantom Gaming-ITX/ac
WIFI / BLE | BCM94360CS2 | With M.2. adapter
Memory | Asgard 32GB * 2 | OC doesn't require remapping 32GB single memory like Clover did.
# Configs
## BIOS Settings (BIOS Version 4.30)
Location | Item | Default Value | My Value | Reason and Result
--- | --- | --- | --- | ---
Advanced | Fast Boot | Disabled | Disabled | DK.
Advanced | CFG Lock | Disabled | Disabled | Need more resarch
Advanced | VT-d | Enabled | Enabled | **Enable quirk ```DisableIoMapper``` then no need to disable VT-d.**
Advanced | CSM | Enabled | **Disabled** | Everybody kept saying disable it but it seems to be no harm except for the super high resolution during booting on my 4K monitor. It's also no harm disabling it so up to you.
Advanced | VT-x | Enabled | Enabled | Need more research
Advanced | Above 4G decoding | Disabled | Disabled | Need more research
Advanced | Hyper Threading | Enabled | Enabled | Need more research
Advanced | Execute Disable Bit | Enabled | Enabled | Need more research
Advanced | EHCI/XHCI Hand-off | Disabled | **Enabled** | Boot stalls if with USB devices connected. Seems to be the only necessary change in BIOS.
Others | OS type | No option | No option | No option
Advanced | Intel SGX | Enabled | Enabled | Need more research
Security | Intel(R) Platform Trust Technology | Disabled | Disabled | Need more resarch
Advanced | Thunderbolt | Enabled | Enabled | Need more resarch
Advanced | iGPU Multi-Monitor | Disabled | Disabled | Probably necessary for those using dGPU.



## OpenCore Config Details
**Not all the configs were explained here**
### ACPI
#### ACPI SSDT Patch
<details><summary>SSDT-AMAC fix RTC bug</summary>

macOS requires `RTC._STA` to return `0x0F`, meaning RTC enabled. While Asrock BIOS (ASUS and some other compaines too) enabled AWAC and disabled RTC by default (Probably for Windows).  
After reviewing the codes, I replaced the old renaming hotpatch with SSDT-AWAC SSDT patch.
The renaming hot patch replaced the ```If ((STAS == One))``` of `RTC._STA` in DSDT with ```If ((0xFF || 0xFFFF))``` , so the RTC._STA will always return `0x0F`. It might lead to a conflict in the future, since both RTC and AWAC are enabled at the same time.  
If you failed to boot, you may find the renaming method in an older commit, or use ```SSDT-RTC0``` instead.  
Reference: [
FIX for boot hangs after BIOS update (ACPI PATCH)](https://www.tonymacx86.com/threads/fix-for-boot-hangs-after-bios-update-acpi-patch.275091/page-7#post-1972443), [ACPISamples](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples)

</details>
<details><summary>SSDT-PLUG enables XCPM</summary>

XCPM was said to bring better performance ( or sth blah blah ). At least it's harmless.  
Check status by ```sysctl -n machdep.xcpm.mode```.  
You might need to change PR00 to CPU0, depending on your MB model. You could query this by ```ioreg -p IODeviceTree -c IOACPIPlatformDevice -k cpu-type -k clock-frequency | egrep name | sed -e 's/ *[-|="<a-z>]//g'```  
You might need to enable Intel SpeedStep in your BIOS. For my MB it's enabled by default.  
Reference: [SKL+平台XCPM+HWP完整原生电源管理探究](https://www.misonsky.cn/102.html), [macOS Native CPU/IGPU Power Management](https://www.tonymacx86.com/threads/macos-native-cpu-igpu-power-management.222982/), [SSDT-PLUG.dsl](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-PLUG.dsl)

</details>
<details><summary>SSDT-PPMC fix energy saver options</summary>

This SSDT fix seems to be harmless.  
You could verify it by looking into Preferences. You should be able to see 5 options (including power nap) instead of 2.
</details>

### Kernel
#### Quirk
- DisableIoMapper = YES.
>This option is a preferred alternative to dropping DMAR ACPI table and disabling VT-d in ﬁrmware preferences, which does not break VT-d support in other systems in case they need it.
