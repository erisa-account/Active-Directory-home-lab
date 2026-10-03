# Group policy, security groups, and password policy

After the OUs and users were in place, I wanted to go beyond basic account creation and show some real system configuration work, done directly on the server.

Problem: dsa.msc doesn't exist on Server Core

I tried running dsa.msc directly on the server, to create OUs and users without going through the client. It's a graphical tool, and Server Core has no graphical framework to run it in, even with the AD management tools installed — so it just failed with "term not recognized."

Fix: I used the AD PowerShell cmdlets directly on the server instead, with the same result:

```powershell
New-ADOrganizationalUnit -Name "IT" -Path "DC=lab,DC=local"
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" -SamAccountName "jsmith" -UserPrincipalName "jsmith@lab.local" -Path "OU=IT,DC=lab,DC=local" -AccountPassword (ConvertTo-SecureString "Passw0rd!23" -AsPlainText -Force) -Enabled $true
Group Policy: default wallpaper for a department
```

I created and linked a GPO to push a default desktop background to everyone in the Employees OU:

```powershell
New-GPO -Name "Employees-Background" -Comment "Sets default wallpaper for Employees OU"
New-GPLink -Name "Employees-Background" -Target "OU=Employees,DC=lab,DC=local"
Security groups
```

I grouped users by department instead of managing permissions per account:

```powershell
New-ADGroup -Name "IT-Admins" -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
Add-ADGroupMember -Identity "IT-Admins" -Members "jsmith"
```

My first attempt at Add-ADGroupMember failed with "cannot find an object with identity." I'd used the full UPN (jsmith@lab.local) instead of the SamAccountName (jsmith), which is what -Members actually expects. I checked the correct value with Get-ADUser -Filter * and it worked after that.

Fine-grained password policy

I set a stricter password policy for the Finance group specifically, instead of relying on the one default policy for the whole domain:

```powershell
New-ADFineGrainedPasswordPolicy -Name "Finance-StrictPolicy" -Precedence 10 -MinPasswordLength 12 -PasswordHistoryCount 5 -ComplexityEnabled $true -LockoutThreshold 3
Add-ADFineGrainedPasswordPolicySubject -Identity "Finance-StrictPolicy" -Subjects "Finance-Staff"
```

I ran into a syntax issue here too — I'd split the command across multiple lines using the backtick line-continuation character, but left a trailing space after one of the backticks, which breaks it silently. Writing the whole command on one line avoided the issue entirely.
