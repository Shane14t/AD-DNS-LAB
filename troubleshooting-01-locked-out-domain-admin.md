# Troubleshooting Log: Locked Out of the Domain Administrator Account

## Summary
After promoting DC01 to a domain controller, I forgot the `LAB\Administrator` password and had no VirtualBox snapshot from before it was set. Recovered access using an offline Windows Recovery Environment (WinRE) technique, without rebuilding the VM or losing any lab progress.

## Why this isn't a simple local password reset
On a standalone Windows machine, you can often reset a forgotten local account password by booting offline and editing the SAM database directly. That doesn't work here, because **domain account credentials live in the Active Directory database (`ntds.dit`)**, not the local SAM, and `ntds.dit` is only mounted when the NTDS service is running — which it isn't while the machine is offline/booted from media.

The fix instead relies on getting a SYSTEM-level command prompt *after* the server has booted normally, while NTDS is already running, so `net user /domain` can talk directly to Active Directory.

## Root cause
Password for `LAB\Administrator` set during the `dcpromo`/AD DS promotion step was not recorded, and no snapshot existed from before that point.

## Resolution steps

1. **Booted from the Windows Server 2025 installation ISO** and chose *Repair your computer* → *Troubleshoot* → *Advanced options* → *Command Prompt* to reach WinRE's offline shell.
2. **Identified the correct OS drive letter.** WinRE assigns its own temporary letters, so the real Windows install was not on `C:` as expected — it was `D:`. Confirmed with:
   ```
   dir D:\Windows\System32\utilman.exe
   ```
   A real file listing (vs. "cannot find the path") confirmed the drive.
3. **Backed up and replaced the Ease of Access binary** so it launches a command shell instead of the accessibility menu:
   ```
   copy D:\Windows\System32\utilman.exe D:\Windows\System32\Utilman.exe.bak
   copy D:\Windows\System32\cmd.exe D:\Windows\System32\utilman.exe
   ```
4. **Removed the ISO and rebooted normally** into the regular domain controller logon screen (not WinRE, not Safe Mode).
5. **Clicked the Ease of Access icon** on the logon screen, which opened `cmd.exe` running as SYSTEM — on a *normally booted* DC, with NTDS already running.
6. **Reset the domain account password directly against Active Directory:**
   ```
   net user Administrator NewP@ssw0rd1! /domain
   ```
7. **Logged on** with the new password to confirm access.
8. **Reverted the Ease of Access binary** (closing the hole this technique opens) by booting back into WinRE and running:
   ```
   copy D:\Windows\System32\Utilman.exe.bak D:\Windows\System32\utilman.exe /Y
   ```
9. **Took a VirtualBox snapshot** (`Post-Password-Recovery`) so this never has to be repeated.

## Gotchas encountered
- **Drive letter mismatch:** `C:` in WinRE is often a recovery/EFI partition, not the OS. Use `dir <drive>:\Windows\System32\utilman.exe` to confirm before acting.
- **Typing one command across two lines:** pasting isn't available between host and WinRE (VirtualBox clipboard sharing requires Guest Additions, which doesn't run inside WinRE), so commands have to be typed by hand — splitting a long command across two Enter presses silently ran a harmless partial copy into the WinRE RAM disk (`X:\sources`) instead of failing loudly.
- **Executing a path instead of listing it:** typing a bare file path (no `dir` in front) tries to *run* the file, producing `The system cannot execute the specified program.` instead of a directory listing.

## Why I reverted the Ease of Access swap
Leaving `cmd.exe` in place of `utilman.exe` is a known, documented local-privilege-escalation technique (it gives anyone with physical/console access to the logon screen a SYSTEM shell with no credentials). Restoring the original binary immediately after use is standard practice, even in an isolated lab — it's the habit that matters for real environments.

## Key takeaway
Record service account / DSRM / domain admin passwords in a password manager *before* moving on to the next phase, and snapshot the VM right after any step that sets a new credential.
