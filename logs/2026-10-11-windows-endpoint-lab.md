# Windows Endpoint Lab: Intune on a Proxmox VM

**Date:** 2026-10-10 to 2026-10-11
**Category:** Home lab
**Target:** A Windows 11 VM on my Proxmox home lab, managed by Microsoft Intune
**Tools:** `Proxmox VE` `Windows 11 Enterprise Evaluation` `Microsoft Intune` `Microsoft Entra ID` `BitLocker` `PowerShell` `Remote Desktop` `Uptime Kuma`

## Summary

My Intune experience so far was coursework, not a lab, and my home lab had no Windows machine. I built a Windows 11 VM, joined it to Entra ID, enrolled it in Intune, and got a compliance policy and a BitLocker policy working. The full tutorial and write-up are in the new [windows-endpoint-lab](https://github.com/kramer-sb/windows-endpoint-lab) repo, which is a work in progress. The firewall policy, app deployment, update ring and Conditional Access are still to come.

## Steps

1. **Build the VM**: UEFI, a virtual TPM 2.0, 4 cores, 100 GiB disk, VirtIO storage. The Windows installer showed no disks until I loaded the VirtIO driver.
2. **Protect it before enrolling**: took a snapshot and an encrypted backup to the Proxmox Backup Server on the USB drive. Gotcha: a snapshot name has to start with a letter.
3. **Remote Desktop**: the Proxmox console has no copy and paste, so I set up Remote Desktop. It failed at first with error 0x204 until a full reboot created the listener.
4. **Set up the Microsoft side**: a new tenant, a lab user, a break-glass admin, a security group, MFA with a TOTP app, and 30-day trials of Intune and Entra ID P1.
5. **Enroll**: joined the VM to Entra ID, then confirmed it with `dsregcmd /status` and in the Intune portal.
6. **Policies**: a compliance policy with eight settings, and a BitLocker policy with silent encryption and the recovery key backed up to Entra ID.
7. **Fit it into the lab**: static IP, DNS pointed at my CoreDNS server, IPv6 off on the adapter, and an Uptime Kuma monitor on the Remote Desktop port.

## Lessons Learned

- **The portal status and the device can disagree.** Intune showed my BitLocker policy as Succeeded for over an hour while the device was failing. The BitLocker Management event log on the VM held the real error: it had detected bootable media. I had left the Windows installer ISO attached to the VM. Detaching it fixed everything within minutes.
- **A compliant device is not always a protected one.** Windows had already started encrypting the drive on its own, but with no key protectors and protection off. Intune's check only asks whether the drive is encrypted, so it passed anyway.
- **Dashboards lag.** The policy's own status page showed the result well before the dashboard or the device page did.
- **The Intune trial does not include Entra ID P1**, which automatic enrollment needs. That was a separate trial.
- **`.lab` names stopped resolving until I turned off IPv6**, because Windows was asking my ISP's IPv6 DNS server. It was the same problem I had on my Windows host earlier.
- **Main lesson: when something looks fine in the portal and wrong on the machine, trust the machine's logs.**

## Notes

I used Claude as a step-by-step guide while building this. I asked it to correct itself where its instructions were wrong, and the repo's tutorial includes those fixes. My own pre-commit hooks scan the new repo for tenant names, IDs and keys before every commit, since it's public.

## References

- [windows-endpoint-lab repo](https://github.com/kramer-sb/windows-endpoint-lab)
- [My home lab docs](https://github.com/kramer-sb/proxmox-homelab)
