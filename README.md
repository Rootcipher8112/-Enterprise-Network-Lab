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

![win_server_dashboard](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/be3931c3-1dce-432c-9b4e-271be56aa11f)
## 

**Before setting up the domain controller I had to configure my internal network NIC on the Windows server machine.** 

**The NIC which is configured for NAT has a private IP address of 10.0.2.15 from the virtual DHCP server.** 

**The NIC connected to the internal network I manually assigned an IP address of 172.16.0.1. This is also a private network address but is destinct from the other one which will make things easier later.** 

**I also set the Default DNS (domain name system) server as the loopback address of 127.0.0.1. This will allow me to set the default DNS server on my client machines to 172.16.0.1 which will be essential for them to connect to the domain as well as access the internet if needed.**


**DNS more specifically converts the fqdn (fully qualified domain name) to an IP address which is necessary for computers and network devices to send traffic over the network as well as on the internet.** 

**This will be essential for our client machines to connect to the domain and ultimately the internet.**

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

Again, this is to allow the client machines to access the interent through the Windows Server using NAT (network address translation). 

Basically it allows the router to take information coming from the internet to the public IP address and be routed to correct local machine's private IP address and vice versa.  

In this step I selected the NAT NIC to be used as the public facing connection for routing. Other than that there are no other configurations need for this tool.

### DHCP Server Configuration
![dhcp_sever_with_router](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/36054694-19e2-46a7-8cbc-c87074db0829)

DHCP (dynamic host configuration protocol) is used with networks to assign each device with an IP address in order to communicate with other computers or devices on a local network or the internet. 

Instead of the user trying to choose their own IP address most hosts are configured to do this automatically through the use of a DHCP server or can be set up manually in some cases such as a printer where you would want it to be static. 

The server leases out an IP address for a predetermined amount of time and can either renew the lease or assign a new IP address when the lease is up. 

The IP addresses are leased based on a "pool" which is determined by the network administrator based on multiple factors such as the size of the subnet. This pool is known as the scope.

For this lab I have configured a scope of 100 addresses ranging from 172.16.0.100 - 172.16.0.200 and a default lease time of 8 days. This will be the pool the DHCP server uses to assign addresses to the client machines. 

![address_pool](https://github.com/Rootcipher8112/-Enterprise-Network-Lab/assets/123340212/dd1c5539-5f01-43c6-b825-a7c6428f3931)

In addition during DHCP configuration the default gateway and DNS server were set to 172.16.0.1 which again will be necessary for connecting to the domain as well as routing ability.

### Now the configuration is complete and we can move on to setting up each client machine
## 

### Windows 10 Client Configuration

