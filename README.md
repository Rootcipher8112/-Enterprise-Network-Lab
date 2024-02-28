# Enterprise-Network-Lab

### Objective

The Enterprise Network Lab project aimed to create a simulated real-world enterprise environment for hands-on experience in networking and systems administration. The primary focus was on building a lab network that included a Windows Server 2019 acting as the domain controller, DHCP server, and DNS server, along with Windows and Linux clients. Key tasks involved creating users, connecting client hosts, and managing group policies to enhance network security.

### Skills Learned

- Advanced understanding of setting up and configuring a Windows Server 2019 environment.
- Proficiency in managing domain services such as DNS, DHCP, and Active Directory.
- Ability to create and manage user accounts within a domain.
- Hands-on experience in connecting Windows and Linux clients to the domain.
- Knowledge of implementing and enforcing security policies through Group Policy.

### Tools Used

- Windows Server 2019 for domain controller, DHCP, and DNS services.
- Windows and Linux clients for simulating diverse end-user systems.
- Active Directory for user account management and authentication.
- Group Policy for implementing and managing security policies.

## Steps
![networklab](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/9b88d82e-d114-4715-917b-cab2123d57a9)

For this lab I used my virtualization method of choice which is Virtual Box. I like the ease of use and the customization settings for each virtual machine. The first step was to download the ISO files for each of the machines I would need Windows Server 2019, Ubuntu 22.04, and Windows 10. These can be found through simple google search. One thing to now is to use the official sites for each of these to ensure the files are not corrupted.

After creating/spinning up each machine and going through the installation process my next step was to create the domain and set up the Windows Server 2019 box as the domain controller. In the configuration I gave this machine 2 NICs (network interface card). One was configured for NAT to acces the internet and the other set to internal network (I will discuss more on this later).
##
![win_server_dashboard](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/be3931c3-1dce-432c-9b4e-271be56aa11f)
## 

**Before setting up the domain controller I had to configure my internal network NIC on the Windows server machine.** 

**The NIC which is configured for NAT has a private IP address of 10.0.2.15 from the virtual DHCP server.** 

**The NIC connected to the internal network I manually assigned an IP address of 172.16.0.1. This is also a private network address but is destinct from the other one which will make things easier later.** 

**I also set the Default DNS (domain name system) server as the loopback address of 127.0.0.1. This will allow me to set the default DNS server on my client machines to 172.16.0.1 which will be essential for them to connect to the domain as well as access the internet if needed.**


**DNS more specifically converts the fqdn (fully qualified domain name) to an IP address which is necessary for computers and network devices to send traffic over the network as well as on the internet.** 

**This will be essential for our client machines to connect to the domain and ultimately the internet.**
##
![dns_server](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/55b3f986-0a99-4319-b750-62e9dbd97aa6)

## Setting up the Domain on Windows Server:

### Initial Configuration and Tools
- Change the computer name in the advanced settings (I chose to name mine DC-1 for simplicity)

- Setup the local server with Active Directory Domain Services (this is where the server is configured as the Domain Controller)

- Added Remote Access Management (this is to allow the clients I would be connecting to use NAT and access the internet through the server)

- Added DHCP tool

Now that the initial configuration is completed, the **rootdomain.com** domain is created. **DC-1** (the Windows server 2019 machine) is configured as the domain controller and the DNS server is deployed. I can start configuring the other necessary tools within the Windows server manager. First up was the Remote Access Management

### Remote Access and Routing
![remote_access_NAT](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/e252da43-0930-4bdb-b895-a0f95017e84b)
##
Again, this is to allow the client machines to access the interent through the Windows Server using NAT (network address translation). 

Basically it allows the router to take information coming from the internet to the public IP address and be routed to correct local machine's private IP address and vice versa.  

In this step I selected the NAT NIC to be used as the public facing connection for routing. Other than that there are no other configurations need for this tool.

### DHCP Server Configuration
![dhcp_sever_with_router](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/36054694-19e2-46a7-8cbc-c87074db0829)
##
DHCP (dynamic host configuration protocol) is used with networks to assign each device with an IP address in order to communicate with other computers or devices on a local network or the internet. 

Instead of the user trying to choose their own IP address most hosts are configured to do this automatically through the use of a DHCP server or can be set up manually in some cases such as a printer where you would want it to be static. 

The server leases out an IP address for a predetermined amount of time and can either renew the lease or assign a new IP address when the lease is up. 

The IP addresses are leased based on a "pool" which is determined by the network administrator based on multiple factors such as the size of the subnet. This pool is known as the scope.

For this lab I have configured a scope of 100 addresses ranging from 172.16.0.100 - 172.16.0.200 and a default lease time of 8 days. This will be the pool the DHCP server uses to assign addresses to the client machines. 
##
![address_pool](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/dd1c5539-5f01-43c6-b825-a7c6428f3931)
##
In addition during DHCP configuration the default gateway and DNS server were set to 172.16.0.1 which again will be necessary for connecting to the domain as well as routing ability.

### Creation of Orginizational Units and User accounts 

Organizational Units (OUs) are containers within Active Directory that allow for the organization and management of objects such as user accounts, computer accounts, and groups. OUs provide a hierarchical structure, enabling administrators to group and apply Group Policy settings to sets of objects based on their organizational or administrative needs. OUs simplify the management of Active Directory by offering a way to delegate administrative authority and apply policies to specific groups of objects. 

In this lab we will use OUs to create 4 departments which we will use to demonstrate how group policy works later on and establish an important security implementation - least privilege. The four departments we are creating are:
  - Admins (This will be network administrators or other high level users that will have the most access)
  - HR (This department will have access to user accounts in order to complete onboarding and offboarding, training, employee records, and benefits information)
  - Management (This department will have high level access to company financials, operational data, customer feedback, legal files and strategic planning)
  - Sales (This department will have access to customer data CRM materials, training material, and other sales associated files/ folders)

To create these orginizational units we will:
  - Navigate to Active Directory Users and Computers
  - Right-click on the domain (for this lab rootdomain.com)
  - Select "New"
  - Select "Orginizational Unit"
  - Name the orginational unit

**This will need to be done for each of our 4 departments.**

Next we will create some users to add to our OUs. We can do this in multiple ways. For example we go navigate into one of our OUs, right-click, select "new" and select "user". Then the user information such as name, username, and password can be set. This can be done when adding a single user but for this lab we are adding 100 users so this would take a long time.  Instead we are going to speed this process up and utilize a Power Shell script.

For our script we want to randomly create names, generate a username based on their name, give them all an initial password of "12345" and have them change their password on their first login. The script will look something like this:

    $PASSWORD_FOR_USERS   = "12345"
    $NUMBER_OF_ACCOUNTS_TO_CREATE = 100
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
                   -PasswordNeverExpires $false `
                   -UserMayNotChangePassword $false
	           -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
                   -Enabled $true
        $count++
    }


**Now we can run this in Power Shell as administrator to generate our users.  After we can go back over to Active Directory Users and Computers to see our users created. The we can add them to our 4 departments.** 

  - 5 for Admin
  - 5 for HR
  - 20 for Management
  - 70 for Sales


### The initial Active Directory server configuration is complete and we can move on to setting up each client machine

 
## Windows 10 Client Configuration



To connect our Windows 10 client machine to the domain we will take the following steps:

   1. Network Configuration:
       - Ensure that the Windows 10 machine is on the same network as the Windows Server 2019 domain controller. (This can be done in the network adapter settings. Since DHCP is set up we can see here we are given an IP address from the pool we set)
       - Verify that the Windows 10 machine can communicate with the domain controller. (We can do this by pinging DC-1 at 172.16.0.1)
      
   2. Verify DNS Settings:
       - Ensure that the DNS settings on the Windows 10 machine point to the IP address of the Windows Server 2019 domain controller. (You can set the DNS server in the network adapter settings.)

   3. Change Computer Name:
       - Under start go to settings.
       - Go into system, then down to "Rename PC (Advanced).
       - Select "Properties" and click on "Change settings" under the "Computer name, domain, and workgroup settings" section.
       - Click the "Change" button.
       - Enter a computer name and ensure that the "Domain" option is selected. Enter the domain name (for our lab it is rootdomain.com).
       - Click "OK" and provide domain administrator credentials when prompted.

   4. Restart the Computer:
       - You'll be prompted to restart the computer. Go ahead and restart it.

   5. Log in with Domain Credentials:
       - After the restart, log in to the Windows 10 machine using domain credentials. Make sure to select "other user"
   6. Verify Domain Join:
       - Hover over the network icon in the bottom right corner.
       - Or open command prompt and type whoami


## Ubuntu 22.04 Client Configuration


Connecting an Ubuntu client can be more involved than a Windows client. You can add an Ubuntu client to a domain during a fresh install in certain cases however for this lab I felt it beneficial to show an example for when the install has already taken place.  There are a few ways to accomplish this, realm or winbind but for this lab I chose to use realm. Here are the steps:

   1. Network Configuration:
       - Ensure that the NIC of your Ubuntu virtual machine is set to an internal network.
       - Verify that the Ubuntu VM can communicate with the Windows Server 2019 VM.

   2. Update Ubuntu:
       - Open a terminal on your Ubuntu VM.
       - Run the following commands to update the system:

             sudo apt update
             sudo apt upgrade

   3. Install Required Packages:
       - Install the necessary packages for joining a Windows domain:

             sudo apt install realmd sssd sssd-tools adcli krb5-user packagekit samba-common samba-common-bin samba-libs samba-dsdb-modules samba-vfs-modules

   4. Set up DNS to point to Domain Controller:
       - In the terminal use the comman     ip a  to record IP address and the network adapter name.
       - Edit the networking file by running the following command
           
             sudo nano /etc/netplan/01-network-manager-all.yaml
       
       - In this file we are going to add the following: 
           
             network:
               version: 2
               renderer: networkd
               ethernets:
                 enp0S:
                   dhcp4: yes

                   nameservers:
                     addresses: [172.16.0.1]
       
       Here we enter the network adapter name (enp0S) and the IP address of our DC. In some cases if DHCP is not enabled a static IP will need to be added for the connection to work but that is not the case for this lab so DHCP is marked yes.
    
### It's important after you save the file to run the following command so the changes take place:

          sudo netplan apply
    
   6. Change the host name:
       - To change our host name run the following command:
            
              sudo hostnamectl set-hostname client-2.rootdomain.com
       
       - To verify the change happened run the "hostname" command.
    
   7. Check to see if the domain is visible to the Ubuntu machine:
      - Run the following command to attempt a connection to the domain:
            
            realm discover ROOTDOMAIN.COM

   8. Join the domain
      - To join the domain as one of our domain admins we will run the following command:
            
            realm join -U jrowe ROOTDOMAIN.COM

      - You will be prompted to enter the Administrator password, then press enter.
      - Verify the connection using the following command:
            
            realm list

      - We can also verify user connection using the following command:

    	    id jrowe@ROOTDOMAIN.COM
   
   9. Lastly we edit the common-session file to automatically create a home folder for the new user.
      - To edit the file we will run the following command:
            
            sudo nano /etc/pam.d/common-session

       - Next we will navigate to the line that reads "session optional" and insert the following:

             pam_mkhomedir.so skel=/etc/skel umask=077
      
       - Save the file and exit
    
### Now we can log in as a user on the domain.
  - Log out of the current session
  - At login screen select "other user"
  - Enter "[username]@DOMAIN.COM" (for this lab jrowe@rootdomain.com)
      
###   Success!!!


**Back over on our DC we can go to our Active Directory users and computers to verify the connection on this end.**


##
### Let's recap what we have set up so far:
  
  - Windows Server 2019 with Active Directory Services setup as Domain Controller with DNS, DHCP, and Remote Access enabled and configured
  - Orginizational Units for our different teams
  - 100 user accounts utilizing a Powershell Script
  - Windows 10 client configured and joined to the domain
  - Ubuntu 22.04 client configured and joined to the domain

**Next we will begin to dive into managing group policies for our different Orginizational Units.**

##
### Group Policy Implementation



