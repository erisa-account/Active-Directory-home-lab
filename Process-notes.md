# How this lab was built (and what went wrong)

## Problem 1: One laptop wasn't enough

My laptop only has 8GB of RAM, so running both a server VM and a client VM on it at once made everything sluggish.

**Fix:** split the two VMs across two laptops, connected over the same Wi-Fi using VirtualBox's "Bridged Adapter" mode. This makes each VM show up as its own device on the network, like a real physical PC, so each one gets a full laptop's resources instead of sharing.

## Problem 2: Server install froze at 2%

The first install attempt sat at 2% for over two hours with no disk activity. I assumed it was a RAM problem.

**Actual cause:** the ISO file was corrupted. It was smaller than it should have been.

**Fix:** redownloaded a clean ISO and the install finished normally. Lesson learned: check the file size before blaming the hardware.

## Problem 3: GUI version was too heavy

Even with a working ISO, the full Desktop Experience version of Windows Server felt slow on 2GB of RAM.

**Fix:** reinstalled using **Server Core** instead — same installer, just picking the edition without "(Desktop Experience)." No desktop at all, everything runs through `sconfig` and PowerShell. Lighter, and closer to how real servers are actually run.

## Problem 4: VM kept landing on the wrong network

`ipconfig` inside the VM kept showing an address starting with `10.0.2.15`. That's VirtualBox's signature for its default "NAT" mode, which boxes the VM off from the rest of the network — meaning server and client could never see each other like that.

**Fix:** switched the network adapter setting from NAT to **Bridged Adapter**, pointed at the laptop's real Wi-Fi. After that it picked up a normal home-network address and the two machines could reach each other.

## Problem 5: Server froze right before becoming a Domain Controller

After installing Guest Additions and testing drag-and-drop between host and VM, the VM stopped responding — mouse moved, nothing else did. This happened right as I was about to promote the server.

**Fix:** used VirtualBox's "Reset" (a forced restart, not a reinstall) instead of force-closing it. Turned off clipboard sharing and drag-and-drop, which seemed to be the actual cause, and left the VM alone while the next command ran. Worked fine the second time.

## The actual AD part was short

Once things were stable, turning the server into a Domain Controller only took two commands:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.local" -InstallDNS
```

The second one sets up DNS automatically and creates the domain `lab.local`. Checked it worked with:

```powershell
Get-ADDomain
whoami
```

`whoami` coming back as `LAB\Administrator` confirmed the server now belonged to its own new domain.

## Problem 6: Client couldn't find or join the domain

This was the longest part of the whole project. Networking issues, not AD, were the real cause — covered on their own in `NETWORKING-NOTES.md`, since it turned out to be the Wi-Fi quietly blocking the two laptops from reaching each other.

## Problem 7: Opened the wrong tool

Tried to create an OU and couldn't find a "New" option anywhere. Turned out I had opened **Active Directory Sites and Services** by mistake, not **Active Directory Users and Computers** — two different tools with similar names. Opening the right one (`dsa.msc`) fixed it right away.

## Problem 8: Password kept getting rejected

New user accounts kept failing with a vague "check password requirements" message. AD enforces a complexity rule by default — 8+ characters, a mix of upper/lowercase, a number, and a symbol, and it can't contain the username. Once the password met all of that, it went through.

## What actually got built

With the client joined to `lab.local`:

- Opened Active Directory Users and Computers (`dsa.msc`) from the client
- Created an OU structure: **IT**, **Finance**, **Employees**
- Created user accounts inside those OUs, each with a logon name and a password that met the domain's rules

## Biggest takeaway

The hard part was never really Active Directory — it was the setup around it: getting two machines to actually talk to each other, picking a Windows edition that fit the hardware, and figuring out when something was truly broken versus just slow. Once that was sorted, the AD steps themselves were quick.
