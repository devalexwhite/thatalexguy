---
title: "Fixing System Freezes with System76 Pang12 + Arch"
date: 2025-06-26
draft: false
tags: ["linux"]
---

More as a reference for myself in the future, here's how to debug and resolve issues with complete system lockups when running Arch on the System76 Pang12 laptop.

On my computer, I was experiencing complete system freezes that required hard rebooting the computer. This turned out to be related to the `amdgpu` driver. To check if this is your issue as well, run the following after the system freezes:

`sudo journalctl --since=today | grep amdgpu` (make sure to run as sudo or you'll miss the needed log entries)

Look for a set of lines like the following:

```journalctl
Jun 26 09:14:41 pangarch kernel: snd_hda_intel 0000:04:00.1: bound 0000:04:00.0 (ops amdgpu_dm_audio_component_bind_ops [amdgpu])
Jun 26 12:42:08 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:08 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:09 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:09 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:09 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:09 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:10 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* dc_dmub_srv_log_diagnostic_data: DMCUB error - collecting diagnostic data
Jun 26 12:42:20 pangarch kernel: amdgpu 0000:04:00.0: [drm] *ERROR* [CRTC:79:crtc-0] flip_done timed out
```

The `flip_code timed out` is the key here. On the Arch Wiki, they specifically call out this issue in [section 6.11 of the AMDGPU page](https://wiki.archlinux.org/title/AMDGPU). The fix for this is to add either the `amdgpu.dcdebugmask=0x10` or `amdgpu.dcdebugmask=0x12` kernel parameter.

In my case, I use GRUB as my bootloader. To add the param with GRUB, edit `/etc/default/grub` and add the kernel parameter to `GRUB_CMDLINE_LINUX_DEFAULT`. Here's what mine looks like:

```grub
...
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet amdgpu.dcdebugmask=0x10"
...
```

Finally, run `grub-mkconfig -o /boot/grub/grub.cfg` to regenerate your GRUB configuration file.

Reboot your PC, and press 'e' when GRUB appears. You should be able to see the newly added kernel parameter in GRUB. Boot your system as normal, the issue should (hopefully) be resolved. If you still experience crashes, try switching to the other kernel parameter listed by the Arch wiki.

