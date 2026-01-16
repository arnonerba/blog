---
title: "Dell TPM 1.2 to 2.0 Upgrade Instructions"
modified_date: "2026-01-15"
last_modified_at: "2026-01-15"
---
If you have a Dell PC that was manufactured between 2015 and 2018 and originally shipped with TPM firmware version 1.2, you may be able to upgrade it to TPM firmware version 2.0. Several Dell models are capable of switching between TPM firmware versions 1.2 and 2.0 provided the necessary conditions are met.

## Prerequisites

First, your PC must support this upgrade. [Dell maintains a list of supported systems here.](https://www.dell.com/support/kbdoc/en-us/000132583/dell-systems-that-can-upgrade-from-tpm-version-1-2-to-2-0)

Second, your PC should be configured in UEFI Boot Mode instead of Legacy Boot Mode. **Note:** Switching boot modes on an existing system may require a reinstallation of Windows.

Third, while optional, it's wise to update your BIOS to the latest version before attempting this upgrade.

Once you're ready, you can clear the TPM and run the appropriate TPM firmware update utility. However, since Windows will automatically take ownership of a fresh TPM after a reboot by default, we have to take some additional steps to make sure the TPM stays deprovisioned throughout the upgrade process.

## Step-By-Step Instructions

1. Download the appropriate TPM firmware update utility for your system. In Windows, [locate your PC's serial number]({% post_url 2026-01-15-find-serial-number-windows %}), and use it to search [the Dell support website](https://www.dell.com/support) for the latest drivers and downloads for your PC. **You are looking for the latest version of a package named "Dell TPM 2.0 Firmware Update Utility"**.
2. Launch a PowerShell window with administrative privileges. Then, run the following command to disable TPM auto-provisioning (we'll turn it back on later):

    ```powershell
    Disable-TpmAutoProvisioning
    ```

3. Reboot and enter the BIOS setup menu. Navigate to `Security > TPM 1.2/2.0 Security`. If the TPM is turned off or disabled, enable it. Otherwise, click the "Clear" checkbox and select "Yes" to clear the TPM settings.
4. Reboot back to Windows and run the Dell TPM 2.0 Firmware Update Utility you downloaded in step 1. If successful, the utility will trigger another reboot, similar to a BIOS update.
5. When your PC boots back up, run the following command in another elevated PowerShell window:

    ```powershell
    Enable-TpmAutoProvisioning
    ```

6. Reboot your PC again so that Windows can automatically provision the TPM. While you're rebooting, you can take this opportunity to enter the BIOS again and ensure that Secure Boot is enabled (Legacy Option ROMs under `General > Advanced Boot Options` must be disabled first).
7. Finally, check `tpm.msc` or the Windows Security app to confirm that your TPM is active, upgraded, and provisioned.

---

**References:**
- [How to Successfully Update the TPM Firmware on your Dell Computer - Dell US](https://www.dell.com/support/kbdoc/en-us/000184894/how-to-successfully-update-the-tpm-firmware-on-your-dell-computer)
- [How to Troubleshoot and Resolve Common Issues with Trusted Platform Module (TPM) and BitLocker - Dell US](https://www.dell.com/support/kbdoc/en-us/000103639/how-to-troubleshoot-and-resolve-common-issues-with-tpm-and-bitlocker)
- [Trusted Platform Module (TPM) Upgrade or Downgrade Process for Windows 10 Operating System - Dell US](https://www.dell.com/support/kbdoc/en-us/000126503/trusted-platform-module-tpm-upgrade-downgrade-process-for-windows-7-and-10-operating-system-upgrade-downgrade)
- [Trusted Platform Module Common Questions for Windows 11 - Dell US](https://www.dell.com/support/kbdoc/en-us/000190999/trusted-platform-module-frequently-asked-questions-for-windows-11)
- [TPM is Unable to Change Between 1.2 or 2.0 Because TPM is Owned](https://www.dell.com/support/kbdoc/en-us/000139777/tpm-unable-to-change-between-1-2-or-2-0-because-tpm-is-owned)
