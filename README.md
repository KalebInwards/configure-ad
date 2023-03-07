<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Create two virtual machines in Microsoft Azure, a domain and a client
- Install Active Directory 
- Create an admin and normal user account in Active Directory
- Join client to domain
- Create new users and attempt to login to client with one

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/o96svG8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Start off by creating a virtual machine in Microsoft Azure, this will be the Domain Controller (DC) so for the OS choose a server. The region doesn't matter, just as long as the same region is used for the next VM. Make sure to set the DC's NIC Private IP address to be static. 
</p>
<br />

<p>
<img src="https://i.imgur.com/Rxa7PUz.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next create a second virtual machine, this one simply using windows 10. It's important for this VM to be on the same resource group and virtual network as the first.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now use Remote Desktop to log on to the client VM and attempt to ping the DC's private IP address. Use "ping -t", which is a perpetual ping. You can see that the request is timing out, next step is to determine why.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
After logging into the DC open the windows firewall settings. There are a couple of ways of accessing this, one is to simply type firewall into the windows search bar. In the firewall navigate to inbound rules and enable ICMPV4 traffic. At this point, if you go back to the client VM you can see that you are receiving replies from the ping request.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
In order to install Active Directory you need to go back to DC VM and navigate to the server manager, which should already be open by default. Click on add roles and features, then click next until Server Roles comes up, then select Active Directory Domain Services. Continue clicking next on each step to begin installation. 
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next you need to set the DC VM as a server. Click on the notification in the top right corner, then click where it says "promote this server to a domain controller". Select "add a new forest" then type in the domain name you want to use. Click next until the install button becomes available, then click that to finish installation. Once installation is finished the VM will automatically restart and it will be neccessary to log back in, this time using the domain name that was set previously. 
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now navigate to Active Directory Users and Computers (ADUC), which can be found under Windows Administrative Tools in the start menu. Right click on the domain name and select new, followed by organizational unit (OU) and create a new OU called "_EMPLOYEES". Follow the same steps to create a new OU called "_ADMINS". Right click on the _ADMINS group to add a new user named jane doe with the username jane_admin. Right click on jane doe and add the user to the domain admins group. Log out of the DC, then log back in as “mydomain.com\jane_admin”.
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Within the Azure Portal, go to the clients DNS settings and change it to use the private IP address of the DC. After this restart client then log in as the original admin for the client. Right click on the start menu and click on system, then select "Rename this PC". Type in the name of the domain, then enter the login information for jane_admin (this will cause the VM to restart).
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Go back to the DC and double check that client shows up in ADUC inside the “Computers” container on the root of the domain. Create a new OU named “_CLIENTS” and add client to this OU. Log on to client as jane_admin and open system properties, then click on remote desktop. Allow domain users remote access, this will let you log on to client as a non admin user.
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Within DC run Powershell ISE as an administrator, then create a new file and paste the follwing script into it:

 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
} 
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next, run the script and see the accounts being created. Once it's finished, open ADUC and observe the accounts in the appropriate OU. Attempt to log into client with one of the accounts using the password in the script.

<br />
