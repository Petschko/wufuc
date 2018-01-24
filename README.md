# wufuc
[![Donate Bitcoin](https://cdn.rawgit.com/zeffy/wufuc/badges/bitcoin.svg)](https://admin.gear.mycelium.com/gateways/3554/orders/new) [![AppVeyor Builds](https://img.shields.io/appveyor/ci/zeffy/wufuc.svg?logo=appveyor&style=flat-square)][AppVeyor] [![Chat on Telegram](https://cdn.rawgit.com/zeffy/wufuc/badges/telegram.svg)](https://t.me/joinchat/HEo6LUvV_83O92WzbYXLeQ) [![All Releases](https://img.shields.io/github/downloads/zeffy/wufuc/total.svg?style=flat-square)][Latest]

This document is also available in [简体中文], [繁体中文], [русский], [Español] and [more...](https://github.com/zeffy/wufuc/wiki)

Disables the "Unsupported Hardware" message in Windows Update, and allows you to continue installing updates on Windows 7 and 8.1 systems with Intel Kaby Lake, AMD Ryzen, or other unsupported processors.

## Downloads 

**[Latest stable build][Latest] - Most people will want this version.**

[Unstable builds][AppVeyor] - Probably contains bugs; do not report issues with these builds.

## Donate

[**Click here for donation options!**](https://github.com/zeffy/wufuc/blob/master/DONATE.md)

## Preface

The release notes for Windows updates KB4012218 and KB4012219 included the following:

> Enabled detection of processor generation and hardware support when PC tries to scan or download updates through Windows Update.

These updates marked the implementation of a [policy change](https://blogs.windows.com/windowsexperience/2016/01/15/windows-10-embracing-silicon-innovation/) they announced some time ago, where Microsoft stated that they would not be supporting Windows 7 or 8.1 on next-gen Intel, AMD and Qualcomm processors.
This was essentially a big middle finger to anyone who decides to not "upgrade" to the steaming pile of :shit: known as Windows 10, especially considering the extended support periods for Windows 7 and 8.1 won't be ending until January 4, 2020 and January 10, 2023 respectively.

## Some people with older Intel and AMD processors are also affected!

I've received user reports of the following CPUs all being blocked from receiving updates:

- [Intel Atom Z530](https://github.com/zeffy/wufuc/issues/7)
- [Intel Atom D525](https://github.com/zeffy/wufuc/issues/34)
- [Intel Core i5-M 560](https://github.com/zeffy/wufuc/issues/23)
- [Intel Core i5-4300M](https://github.com/zeffy/wufuc/issues/24)
- [Intel Pentium B940](https://github.com/zeffy/wufuc/issues/63)
- [AMD FX-8350](https://github.com/zeffy/wufuc/issues/32)
- [AMD Turion 64 Mobile Technology ML-34](https://github.com/zeffy/wufuc/issues/80)

## Bad Microsoft!

If you are interested, you can read my original write up on discovering the CPU check [here](https://github.com/zeffy/wufuc/tree/old-kb4012218-19).

## Features

- Enables Windows Update on PCs with unsupported processors.
- Written in C, the best programming language. :sunglasses:
- Completely free (as in freedom) software.
- Does not modify any system files.
- Byte pattern-based patching, which means it will usually keep working even after new updates come out.
- No dependencies.

## How it works

Basically, inside a system file called `wuaueng.dll` there are two functions responsible for the CPU check: `IsDeviceServiceable(void)` and `IsCPUSupported(void)`. 
`IsDeviceServiceable` simply calls `IsCPUSupported` once, and then re-uses the result that it receives on subsequent calls.
My patch takes advantage of this behavior by patching a couple of boolean values and basically making Windows Update think that it has already checked your processor, and the result was that it is indeed supported.

- The installer registers a scheduled task that automatically starts wufuc on system boot/user log on.
- Depending on how the Windows Update service is configured to run, wufuc will:
    * **Shared process**: inject itself into the service host process that Windows Update will run in when it starts.
    * **Own process**: wait for the Windows Update service to start and then inject into it.
- After that, wufuc will install some API hooks when appropriate:
    * `LoadLibraryExW` hook will automatically hook the `IsDeviceServiceable()` function inside `wuaueng.dll` when it is loaded.
    * `RegQueryValueExW` hook is necessary to provide compatibility with [UpdatePack7R2](../../issues/100). This hook not applied when `wuauserv` is configured to run in its own process.

## FAQ

### If you get BSODs after installing certain updates on specific AMD CPUs (Updated 01/11/2018)

> Microsoft has resumed updating the majority of AMD devices with the Windows operating system security update to help protect against the chipset vulnerabilities known as Spectre and Meltdown.
>
> A small subset of older AMD processors remain blocked to avoid users getting into an unbootable state after installation of recent Windows operating system security updates. Microsoft is working with AMD to resolve this issue and should resume Windows OS security updates to this subset of AMD processors via Windows Update and WSUS by next week. For AMD device-specific information please refer to AMD’s Security Advisory.
>
> Microsoft has temporarily paused sending the following Windows operating system updates to this subset of older AMD processors:
>
> - January 3, 2018—KB4056897 (Security-only update)
> - January 9, 2018—KB4056894 (Monthly Rollup)
> - January 3, 2018—KB4056888 (OS Build 10586.1356)
> - January 3, 2018—KB4056892 (OS Build 16299.192)
> - January 3, 2018—KB4056891 (OS Build 15063.850)
> - January 3, 2018—KB4056890 (OS Build 14393.2007)
> - January 3, 2018—KB4056898 (Security-only update)
> - January 3, 2018—KB4056893 (OS Build 10240.17735)
> - January 9, 2018—KB4056895 (Monthly Rollup)

For more information, you can read the Microsoft Support article [here](https://support.microsoft.com/en-us/help/4073707/windows-os-security-update-block-for-some-amd-based-devices),
but basically Microsoft put out some updates recently that cause problems on older AMD processors, and blocked them on those CPUs, but apparently
wufuc bypasses the block and lets you uninstall these buggy updates which can cause problems.

Unfortunately as far as I know, there is no comprehensive list of the processors that should be blocked, but if you experience BSODs
after installing any of these updates, you can try the solution posted in this [comment](https://github.com/zeffy/wufuc/issues/132#issuecomment-356921433) to uninstall the updates and fix the crashes.
According to the Microsoft Support article, this issue should be resolved within the next week.

### How to deploy wufuc using Group Policy

[There is a tutorial on the Advanced Installer website that explains how to do this](http://www.advancedinstaller.com/user-guide/tutorial-gpo.html).

### How to use unattended feature in the batch setup scripts

`install_wufuc.bat` and `uninstall_wufuc.bat` both support two command line parameters that can be used alone, or combined to change the behavior of the scripts:

- `/NORESTART` - Automatically declines rebooting after the setup finishes.
- `/UNATTENDED` - Skips all prompts for user interaction, and automatically restarts unless `/NORESTART` is also specified.

These must be used from an elevated command line prompt.

### How to manually remove wufuc v0.8.0.x when it is impossible to uninstall it normally

This only applies to wufuc version 0.8.0.x, which was only available for download for a short period of time. Other versions are unaffected. 

There was a fundamental issue with the method I tried using in this version that caused very serious system instability, such as User Account Control breaking, getting a black screen with just a cursor at boot or after logging out, or very slow overall system performance from multiple services crashing repeatedly which would eventually end in a blue screen of death. Many of these issues unfortunately made uninstalling wufuc nearly impossible. I apologize for any inconvenience this version of wufuc may have caused.

#### To manually uninstall wufuc v0.8.0.x:

1. [Boot into Safe Mode with Command Prompt](https://support.microsoft.com/en-us/help/17419/windows-7-advanced-startup-options-safe-mode).
2. In the command prompt type `regedit` and press enter.
3. Navigate to the key `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`
4. Expand the `Image File Execution Options` tree.
5. Locate the `svchost.exe` sub key, right-click it and press **Delete**.
6. Reboot, and you should be able to log in normally again.
7. Open Add and Remove Programs, locate and run the normal wufuc uninstaller to complete the removal process.

## Sponsors

### [Advanced Installer](https://www.advancedinstaller.com/)

The installer packages are created with Advanced Installer using an [open source license](http://www.advancedinstaller.com/free-license.html). 
Advanced Installer's intuitive and friendly user interface allowed me to quickly create a feature complete installer with minimal effort. Check it out!

## Special thanks

- Wen Jia Liu ([@wj32](https://github.com/wj32)) for his awesome program [Process Hacker](https://github.com/processhacker2/processhacker) which has been absolutely instrumental in the development of wufuc, and also for his [`phnt`](https://github.com/processhacker2/processhacker/tree/master/phnt) headers.
- Duncan Ogilvie ([@mrexodia](https://github.com/mrexodia)) for his [`patternfind.cpp`](https://github.com/x64dbg/x64dbg/blob/development/src/dbg/patternfind.cpp) algorithm from [x64dbg](https://github.com/x64dbg/x64dbg).

[Latest]: https://github.com/zeffy/wufuc/releases/latest
[AppVeyor]: https://ci.appveyor.com/project/zeffy/wufuc
[简体中文]: https://github.com/zeffy/wufuc/wiki/README-(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
[繁体中文]: https://github.com/zeffy/wufuc/wiki/README-(%E7%B9%81%E9%AB%94%E4%B8%AD%E6%96%87)
[русский]: https://github.com/zeffy/wufuc/wiki/README-(%D1%80%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)
[Español]: https://github.com/zeffy/wufuc/wiki/README-(Espa%C3%B1ol)