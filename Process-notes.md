# How this lab was built (and what went wrong along the way)

## Problem 1: Not enough memory for one laptop to run both VMs

My laptop has 8GB of RAM. Running a Windows Server VM and a Windows client VM on it at the same time left barely anything for the host machine, and both VMs felt sluggish.

**Fix:** I split the setup across two laptops — one running the server, one running the client — and connected them over the same home Wi-Fi network using VirtualBox's "Bridged Adapter" networking mode. This makes each VM appear as its own separate device on the network, the same way a real physical machine would, so each VM gets a whole laptop's worth of resources instead of sharing one.

## Problem 2: Windows Server install kept freezing

The first install attempt got stuck at 2% for over two hours with no disk activity. I initially assumed it was a RAM issue.

**Actual cause:** the ISO file was corrupted / incomplete. It was noticeably smaller than a normal Windows install file.

**Fix:** deleted the VM, redownloaded a fresh ISO, and the install completed normally in a reasonable time. Lesson: check the downloaded file size before blaming the hardware.

## Problem 3: GUI version of Windows Server ran too slow

Even after fixing the ISO, the full Desktop Experience (GUI) version of Windows Server felt heavy on 2GB of RAM.

**Fix:** reinstalled using **Server Core** instead — the same installer, just choosing the edition without "(Desktop Experience)" at setup. Server Core has no graphical desktop at all; everything is done through a command-line tool called `sconfig` and PowerShell. It runs noticeably lighter, and it's also a realistic way real servers are managed in production.

## Problem 4: VM defaulted to the wrong network, twice

After setting up networking, `ipconfig` inside the VM kept showing an address like `10.0.2.15` instead of a normal home network address. That address is a signature VirtualBox uses for its default "NAT" networking mode, which isolates the VM from the rest of the network — meaning the server and client would never have been able to see each other.

**Fix:** changed the VM's network adapter setting in VirtualBox from NAT to **Bridged Adapter**, pointed at the laptop's real Wi-Fi adapter. After that, the VM picked up a normal address on the home network (`192.168.1.x`), and the server and client could reach each other.

## Problem 5: Server froze mid-setup, right before promoting it to a Domain Controller

After installing the Guest Additions (a VirtualBox tool that improves mouse/clipboard integration) and briefly testing drag-and-drop between host and VM, the VM became unresponsive — mouse moved, but nothing else responded. This happened right as I was about to run the command that turns the server into a Domain Controller.

**Fix:** used VirtualBox's "Reset" option (the VM equivalent of pressing a restart button, not a reinstall) rather than force-closing it. Disabled clipboard sharing and drag-and-drop, which seemed to be the actual cause of the freeze, and retried the setup without touching the VM window while the command ran. It completed normally the second time.

## What actually worked, once things were stable

Once the environment was stable, the Active Directory part itself was short — really just two PowerShell commands:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.local" -InstallDNS
```

The second command installs DNS automatically and promotes the server to a Domain Controller for a new domain, `lab.local`. Verified it worked with:

```powershell
Get-ADDomain
whoami
```

`whoami` returning `LAB\Administrator` confirmed the machine was now part of the domain it had just created.

## Biggest takeaway

Most of the real difficulty wasn't Active Directory itself — it was the groundwork around it: getting networking right between two machines, picking the right Windows edition for limited hardware, and telling the difference between "this is actually broken" and "this is just slow." Once that groundwork was solid, the AD-specific steps were short and straightforward.
