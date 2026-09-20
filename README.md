# Active Directory Home Lab

A small home lab I built to gain hands-on experience with common IT support tasks, focusing on user account management, Active Directory organization, and basic system configuration as preparation for a junior IT support role.
## What this is

Two virtual machines, running on two separate physical laptops, connected over the same home network:

- **DC01** – Windows Server 2022 (Server Core, no GUI) – Domain Controller for the domain `lab.local`
- **Client01** – Windows 10 Pro – joined to the domain, used to manage AD and test logins

## Why two laptops instead of one

Both laptops only have 8GB of RAM, which isn't enough to comfortably run a server and a client VM on the same machine at once. Splitting them across two laptops, connected over Wi-Fi using VirtualBox's bridged networking, let each virtual machine run with proper resources and made the setup behave more like a real small office network — a server and a separate client machine talking to each other over the network, instead of two VMs boxed into one laptop.

## What's done so far

- Windows Server 2022 installed in Server Core mode (command-line only, no desktop)
- Static IP, gateway, and DNS configured on the server
- Server promoted to a Domain Controller, domain `lab.local` created
- Windows 10 Pro client installed and networked

## What's next

- Install RSAT on the client to manage Active Directory remotely
- Create an OU structure (e.g. IT, Finance, Employees)
- Create user accounts inside those OUs
- Join the client to the domain and log in as one of the created users
- Apply a Group Policy or two (e.g. desktop background, password policy)

## Why Server Core instead of the usual Desktop GUI

Server Core has no graphical interface — everything is done through PowerShell. It uses less memory, which matters on 8GB laptops, and it's also how a lot of real production servers are actually run. Active Directory itself is still managed visually, just from the client machine using RSAT, rather than from the server directly.

See `PROCESS-NOTES.md` for the full story of how this was built, including what went wrong along the way and how it got fixed.
