# Networking: the hardest part of this project

Active Directory itself was short and simple once it was running. The hard part was getting two laptops to actually talk to each other over Wi-Fi. This took longer than everything else combined, so it gets its own file.

## The setup

Two laptops, each running one VM (server and client), both on VirtualBox's "Bridged Adapter" networking which makes each VM appear as its own independent device on whatever Wi-Fi network the laptop is connected to, instead of being boxed inside a private VirtualBox-only network.


## Symptom

Both the domain join (sysdm.cpl) and connecting Active Directory Users and Computers to lab.local failed with timeout / "domain could not be contacted" errors. Both VMs held correct IPs on the same subnet, and ping succeeded, so I knew the problem was more specific than basic reachability.

Diagnostics performed

Firewall: disabled on both server and client as a test. No change.

DNS: confirmed the client resolved lab.local correctly, including the SRV records (_ldap._tcp.lab.local, _kerberos._tcp.lab.local), both pointing to the correct server IP.

Server-side services: confirmed AD DS and DNS services were running, and confirmed the server was actively listening on port 389 (Get-NetTCPConnection).

With DNS, firewall, and the AD services all ruled out, I used Test-NetConnection to check the specific ports AD relies on: port 389 (LDAP) and port 88 (Kerberos) both failed, despite ping succeeding and the server confirmed listening on 389. That gap — ping works, specific ports don't — told me something between the two laptops was blocking traffic selectively, not a configuration issue on either machine.

## Root cause

Client/AP Isolation, a feature on the Wi-Fi extender I was using, which allows all connected devices to reach the internet but blocks them from reaching each other directly. Common on consumer routers/extenders/hotspots as a default security measure, and not always exposed as a toggle.

The extender I was using had no setting to disable this. I switched to a mobile hotspot instead, tested the same connection, and confirmed it worked, no isolation on that network.

## Results 

Since isolation behavior isn't visible in settings and has to be tested directly, I confirmed the hotspot connection worked for AD traffic specifically (not just ping), then kept the two machines on that one network for the rest of the build.

## Related fix: stale APIPA address

After switching from the extender to the hotspot, the server briefly held two IPv4 addresses on the same adapter — the manually-set one, and a 169.254.x.x address left over from the adapter momentarily losing its connection during the switch. I removed the stale one to avoid ambiguity:

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 169.254.200.171 -Confirm:$false
```

## Re-addressing after the network change

Switching from the extender to the hotspot meant both VMs needed their network settings redone to match the new address range.

On the server, removed the old static address, set a new one matching the hotspot's range, and pointed its own DNS at itself:

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress <old IP> -Confirm:$false
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress <new IP> -PrefixLength 24 -DefaultGateway <new gateway>
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses <new IP>
```

On the client,  pointed DNS at the server's new address. The client itself stayed on automatic (DHCP) addressing:

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses <server's new IP>
```

Once this was in place, the domain join succeeded, and I was able to connect to lab.local from Active Directory Users and Computers without further issues.

## Takeaway

Correct IPs, working DNS, and running services aren't enough to prove two machines can reach each other, isolation can sit at the Wi-Fi hardware level, invisible to any of those checks. Testing the actual connection, not just the configuration, is what found the real cause here.
