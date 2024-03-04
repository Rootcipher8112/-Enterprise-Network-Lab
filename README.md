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
![4OUs](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/a946a84d-dac8-413e-85aa-76e9371d239f)

##
Next we will create some users to add to our OUs. We can do this in multiple ways. For example we go navigate into one of our OUs, right-click, select "new" and select "user". Then the user information such as name, username, and password can be set. This can be done when adding a single user but for this lab we are adding 100 users so this would take a long time.  Instead we are going to speed this process up and utilize a Power Shell script.

The first step is to create a txt document with 100 first and last names (or as many names as you like). This can be done with a quick Google search for a random name generator, nothing fancy. Save this as names.txt somewhere, like the desktop, on the Windows Server. 
For our script we want to create an OU of _USERS to put the users we generate in, then generate a username based on each of the names, give them all an initial password of "Password1" and have them change their password on their first login. The script will call on the names.txt file we just created. The script will look something like this:

    	# ----- Edit these Variables for your own Use Case ----- #
	$PASSWORD_FOR_USERS   = "Password1"
	$USER_FIRST_LAST_LIST = Get-Content .\names.txt
	# ------------------------------------------------------ #

	$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
	New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

	foreach ($n in $USER_FIRST_LAST_LIST) {
    		$first = $n.Split(" ")[0].ToLower()
    		$last = $n.Split(" ")[1].ToLower()
    		$username = "$($first.Substring(0,1))$($last)".ToLower()
    		Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    		New-AdUser -AccountPassword $password `
               		-GivenName $first `
               		-Surname $last `
               		-DisplayName $username `
               		-Name $username `
               		-EmployeeID $username `
               		-PasswordNeverExpires $true `
               		-Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
              		 -Enabled $true
	}


**Now we can run this in Power Shell as administrator to generate our users by changing directory to where the names.txt file is saved, in this case C:\Users\jrowe\Desktop\AD_PS-master.** 

![ps_script_ready](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/e41d3371-2e20-4140-857d-956e1a79491e)

**You know the script starts running when you can see the new usernames pop up in the Power Shell window.**

![ps_script_running](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/266401c5-69ff-494b-9e0c-16620b16d70c)

**After we can go back over to Active Directory Users and Computers to see our users created. The we can add them to our 4 departments.** 

![users_ou_added](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/9b53a405-1f38-4a56-82ba-ac09f6fa430e)
![users_present](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c0d858fd-74e1-444c-9364-bc04903cda8b)

For this lab we are going to split them nice and evenly just for demonstration purposes.
  - 5 for Admin
  - 5 for HR
  - 20 for Management
  - 70 for Sales

### Admins
![admins](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c672d632-d4f6-4804-ba02-ce0ddf91fbcc)

**Here we can see that in the Admins OU there is another OU labeled HR. This is to show how easy accidents can happen when doing this if you don't pay attention. As I was creating the 4 OUs I mistakenly right-clicked on Admins instead of the Domain when attempting to create the HR OU. By default there is box that is checked labeled "protect from accidental deletion". Since this was checked I do not have the ability to delete this so it will be there and remain empty.**

### HR
![hr](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/3eebb60f-6abf-48be-a7db-3f749b095a76)

### Management
![management](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/915b1e6d-55bb-4827-81e9-ad13c855acdd)

### Sales 
![sales](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/6ea8ce1f-8fbe-49d1-97f4-70cc4462cd49)


### The initial Active Directory server configuration is complete and we can move on to setting up each client machine

 
## Windows 10 Client Configuration



To connect our Windows 10 client machine to the domain we will take the following steps:

   1. Network Configuration:
       - Ensure that the Windows 10 machine is on the same network as the Windows Server 2019 domain controller. (This can be done in the network adapter settings. Since DHCP is set up we can see here we are given an IP address from the pool we set)
       - Verify that the Windows 10 machine can communicate with the domain controller. (We can do this by pinging DC-1 at 172.16.0.1)
      
   2. Verify DNS Settings:
       - Ensure that the DNS settings on the Windows 10 machine point to the IP address of the Windows Server 2019 domain controller. (You can set the DNS server in the network adapter settings.)

![dns](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/f4060888-a894-4fea-8fb4-243be4e7d0e8)

   3. Change Computer Name:
       - Under start go to settings.
       - Go into system > About > then down to "Rename PC (Advanced).
       - Select "Properties" and click on "Change settings" under the "Computer name, domain, and workgroup settings" section.
       - Click the "Change" button.
       - Enter a computer name and ensure that the "Domain" option is selected. Enter the domain name (for our lab it is rootdomain.com).
       - Click "OK" and provide domain administrator credentials when prompted.

![windows sysabout](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/58eb7054-5a72-499a-a0e5-302f4ed3c5b9)

![rename](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/66fe4dd0-5b01-41c4-a7e8-3e9d00bda012)

![rename2](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/82824c49-71a7-4198-97bd-ac7392bd700b)

![rename3](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/f79688df-d4fc-46e9-be10-90b9e23ff531)

   4. Restart the Computer:
       - You'll be prompted to restart the computer. Go ahead and restart it.

   5. Log in with Domain Credentials:
       - After the restart, log in to the Windows 10 machine using domain credentials. Make sure to select "other user"
   
   ![sign in](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/223b6806-2458-4951-ac9a-27f80e9e5dd3)

   6. Verify Domain Join:
       - Hover over the network icon in the bottom right corner.
       - Or open command prompt and type whoami

![im in](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/301b9841-f51a-40fe-8dcc-3bdbe503e537)

![im in2](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/bc028982-36ca-47f2-a20f-0276a8c0757b)


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

   ![realm_join](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/b4b4f292-6583-4826-831d-f5814c29c4b5)

   8. Join the domain
      - To join the domain as one of our domain admins we will run the following command:
            
            realm join -U jrowe ROOTDOMAIN.COM

      - You will be prompted to enter the Administrator password, then press enter.
      - Verify the connection using the following command:
            
            realm list

![realm list](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/d2df342b-3951-4bee-adff-300032ee996c)

   - We can also verify user connection using the following command:

    	    id jrowe@ROOTDOMAIN.COM
   
  ![verify_join](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/f4e6701b-0756-408e-ba86-408e989b0c86)

   9. Lastly we edit the common-session file to automatically create a home folder for the new user.
      - To edit the file we will run the following command:
            
            sudo nano /etc/pam.d/common-session

       - Next we will navigate to the line that reads "session optional" and insert the following:

             pam_mkhomedir.so skel=/etc/skel umask=077

      ![pams D](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/a361e698-81a6-43f0-8e65-3546dad1bbed)

       
       - Save the file and exit
    
### Now we can log in as a user on the domain.
  - Log out of the current session
  - At login screen select "other user"
  - Enter "[username]@DOMAIN.COM" (for this lab jrowe@rootdomain.com)
      
![success](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c8ab2525-01c6-4f1f-b2da-5127785cf130)

###   Success!!!


**Back over on our DC we can go to our Active Directory users and computers to verify the connection on this end.**

![both joined](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/93eb7b33-9456-4c8d-9bc5-342d8b25e5a7)


##
### Let's recap what we have set up so far:
  
  - Windows Server 2019 with Active Directory Services setup as Domain Controller with DNS, DHCP, and Remote Access enabled and configured
  - Orginizational Units for our different teams
  - 100 user accounts utilizing a Powershell Script
  - Windows 10 client configured and joined to the domain
  - Ubuntu 22.04 client configured and joined to the domain

**Next we will begin to dive into managing group policies for our different Orginizational Units.**


## Group Policy Overview

### Understanding Group Policy

**Group Policy is a feature of Microsoft Windows that provides centralized management and configuration of operating systems, applications, and users' settings in an Active Directory (AD) environment. It allows network administrators to implement specific configurations for users and computers within the scope of an AD domain.**

### How Group Policy Works

- **Group Policy Objects (GPOs)**: Administrators create GPOs using the Group Policy Management Console (GPMC). Each GPO consists of policy settings that control various aspects of user and computer behavior.
- **Scope of Management**: GPOs can be linked to AD domains, OUs, or sites, allowing administrators to apply policies selectively based on the AD hierarchy.
- **Inheritance and Precedence**: GPOs linked to parent containers are inherited by child containers, but specific settings can be overridden by GPOs linked at lower levels in the AD hierarchy.
- **Refresh Cycle**: Group Policy settings are periodically refreshed, ensuring that changes to GPOs are applied to all affected targets.

### Key Features of Group Policy

- **Security Settings**: Configure policies related to user authentication, permissions, and audit policies to enhance the security posture of the organization.
- **Software Deployment**: Automate the installation, update, or removal of software applications across the network.
- **User and Desktop Management**: Control various user and desktop settings, such as desktop themes, network configurations, and start menu options.
- **Scripts**: Assign scripts to run at user logon/logoff or computer startup/shutdown events.

### Best Practices for Group Policy Management

- **Least Privilege Principle**: Apply the minimal necessary policy settings to meet specific requirements, reducing the complexity and potential for unintended consequences.
- **Organizational Unit (OU) Design**: Structure OUs in a way that reflects the organization's functional or geographical layout, facilitating targeted policy application.
- **Testing and Validation**: Implement changes in a test environment before applying them to production, ensuring that new policies do not negatively impact users or systems.

**Group Policy is a powerful tool for administrators to enforce security policies, configure user environments, and manage network resources efficiently. Proper understanding and strategic implementation of Group Policy can significantly enhance the manageability and security of Windows-based networks.**

## Group Policy Implementation

**For this lab we will be adding 3 GPOs for each OU to demonstrate technically how to create them as well as understanding how they effect organizational roles and security best practices. Again, this is only for demonstration and in the real world there would be a far wider range of GPOs set not just for OUs but it could be Domain specific or even site specific. First, I will detail the 3 policies I will implement for each OU with a brief description of why. Then I will go through the process of creating the GPOs.**

### OU specific GPOs

1. **Admins OU:**

	- Policy: Enforce password complexity and change policies.
		- Why: Admin accounts have significant privileges and thus should adhere to strict password policies to prevent unauthorized access.
	- Policy: Enable audit logging for account logon events.
		- Why: To monitor and record activities performed by admin accounts, aiding in the identification of unauthorized access or internal misuse.
	- Policy: Restrict access to the server room and sensitive system settings.
		-Why: To ensure that only admins can access high-security areas and modify critical system settings.



2. **HR OU:**

	- Policy: Limit access to personnel files and sensitive data.
		- Why: HR staff require access to personal data, but this access should be restricted to ensure data privacy and compliance with data protection regulations.
	- Policy: Disable USB storage devices.
		- Why: To prevent the potential exfiltration of sensitive employee data through removable storage.
	- Policy: Enable screen saver lock with password protection.
		- Why: To ensure that unattended computers are not accessible, protecting sensitive information.



3. **Managers OU:**

	- Policy: Enforce session timeout and screen lock policies.
		- Why: To prevent unauthorized access to managerial information when a manager's workstation is left unattended.
	- Policy: Implement Application Control Policies using AppLocker.
		- Why: To ensure that only approved and necessary applications can be executed by the managers, enhancing the security of the systems by reducing the risk of malware execution and preventing the use of unauthorized software.
	- Policy: Enable audit logging for sensitive operations.
		- Why: To track changes and operations performed by managers, ensuring accountability and aiding in forensic investigations if needed.



4. **Sales OU:**

	- Policy: Restrict software installation.
		- Why: To prevent the installation of unauthorized or non-business-related software, reducing security risks and maintaining system performance.
	- Policy: Customize desktop settings and install sales-specific applications.
		- Why: To provide a tailored work environment that enhances productivity and ensures that sales staff have the necessary tools at their disposal.
	- Policy: Restrict access to specific Control Panel items.
		- Why: To prevent unauthorized changes to system settings that could impact security or productivity. By limiting access to certain Control Panel applets, you can ensure that sales personnel have only the necessary tools at their disposal, reducing the risk of accidental system misconfigurations.



### Creating GPOs

1. Access Group Policy Management:

	- On your Windows Server, open the Group Policy Management console. You can do this by opening the Server Manager, clicking on "Tools" in the top-right corner, and then selecting "Group Policy Management."

2. Navigate to the Correct Container:

	- In the Group Policy Management console, expand the forest and domain nodes to locate the Organizational Unit (OU) where you want to create the new GPO. If you're applying the GPO at the domain level, you'll right-click on the domain.

3. Create the GPO:

	- Right-click on the OU (or the domain) where you want to create the GPO, select "Create a GPO in this domain, and Link it here..." In the new window, enter a name for the GPO. Choose a name that clearly describes the purpose of the GPO, such as "HR Department Security Settings."

4. Edit the GPO:

	- Once the GPO is created, right-click on the new GPO under the chosen OU or domain and select "Edit..." This will open the Group Policy Management Editor. In the editor, you'll see two main sections: "Computer Configuration" and "User Configuration." Each section allows you to define policies that apply to computers or users, respectively.

5. Configure Policies:

	- Expand the "Computer Configuration" or "User Configuration" nodes, depending on which context you want to apply the policy. Then, navigate through the settings (Policies and Preferences) to configure the specific policies you want. For example, to enforce password complexity, navigate to Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Password Policy. Here, you can define the password complexity requirements.

6. Save and Apply the GPO:

	- Once you've configured the settings, close the Group Policy Management Editor. The policies will automatically be saved. The new GPO will now be linked to the selected OU, and the policies will be applied to the objects (users or computers) within that OU according to the AD's refresh cycle. You can force a policy update using the gpupdate /force command on the client machines.

7. Test and Verify:

	- It's crucial to test the policies to ensure they are applied as expected. Use the gpresult /r command on a client machine to verify the applied policies.

## Examples

### Admins Security Policies

##
![admin pw policy](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/d9836dbe-1c34-4720-8f62-557a60648946)


![admin logout](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c7147540-811e-492c-92f2-25b81ce4ac70)


![admin system](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/719529b3-510f-43f2-b2b2-fb3644af31f4)
##

### HR Security Policies

##
![hr data access](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/d9a9c72e-4fb1-4884-82f9-70e2350491bf)



![hrusb](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/e965a7ac-b2af-47c1-b2fb-3343ed9730cf)


![hrscrnsvr](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c0d71e4c-87bc-48ab-b064-e21102f55be9)
##

### Managment Security Policies

##
![managementscrnsvr](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/fa028ce2-117b-41a0-9268-8c78ffc36e8d)


![managementappcntrl](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/d5fed453-1ef0-4f7e-bd27-fbeec6333230)


![managementobjectaccess](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/c4baccfc-3b47-4056-9fd6-e55a5ef5f586)
##

### Sales Security Policies

##
![salessoftware](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/e807916c-4a7c-450f-9da8-1f0d6a0856cc)


![salesdesktop](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/70ced8c9-bf84-4a9b-b35d-0bd6595c682e)


![salesnosettings](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/0ac604f7-fe97-4602-a218-d4e5179e3fc1)
##


## gpresult /r

![gpresult](https://github.com/Rootcipher8112/Enterprise-Network-Lab/assets/123340212/ea975c03-98a0-4607-9088-c57c99bfbd83)



### Thank you for checking out my lab. It was an excellent experience putting this together to share my journey with others.
