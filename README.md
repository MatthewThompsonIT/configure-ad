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

- [Setup Domain Controller and Client-1 in Azure](https://github.com/MatthewThompsonIT/configure-ad/tree/main?tab=readme-ov-file#setup-of-resource-group-virtual-network-domain-controller-and-client-1)
- [Install Active Directory and join Client-1 to it](https://github.com/MatthewThompsonIT/configure-ad?tab=readme-ov-file#deploying-active-directory)
- [Create a bunch of additional users to simulate a large business](https://github.com/MatthewThompsonIT/configure-ad?tab=readme-ov-file#creating-users-with-powershell)
- [Enabling, unlocking accounts, and resetting Passwords](https://github.com/MatthewThompsonIT/configure-ad?tab=readme-ov-file#group-policy-and-managing-accounts)

<h2>Setup of Resource Group, Virtual Network, Domain Controller and Client-1</h2>

<p>
  
- Lets begin by creating a resource Group, name it "Active-Directory-Lab" refer to [this tutorial](https://github.com/MatthewThompsonIT/creating-virtual-machines?tab=readme-ov-file#installation-steps---creating-a-resource-group) on creating + configuring resource groups and virtual machines.
<img src="https://i.imgur.com/R9BZG0w.png" alt="resource group"/>

- Create a virtual network for our domain controller and clients to be in.
- Make sure to put it in the same resource group you just made and name it "Active-Directory-VNet"
- Hit "Review + Create" and create the virtual network.
<img src="https://i.imgur.com/5kSPxMG.png" alt="create virtual network"/>

- Create a Virtual Machine (Ensure the settings are the same as mine or things might not work)
   - Put it in the "Active-Directory-Lab" resource group and name it "dc-1"
   - Ensure the region is the same as the virtual network (in my case (US) East US 2)

<img src="https://i.imgur.com/zAqrlXU.png" alt="create domain controller"/>

- For the "image" MAKE SURE to pick "Windows Server 2022 Datacenter: Azure Edition - x64 Gen2"
- When picking the size, pick anything with "2 vcpus"
<img src="https://i.imgur.com/DAGlaxS.png" alt="create domain controller 1"/>

- Create the Administrator Account
  - Name it something secure and write it down.

![image](https://github.com/user-attachments/assets/16f778ea-1d2c-4157-aa6a-e010b60c5c91)

- Check both boxes under "Licensing"
- Hit "Next: Disks" then again for "Networking"

![image](https://github.com/user-attachments/assets/2e26047d-108d-40a1-be7d-32d3b3cba340)

- Make sure to put it in the "Active-Directory-VNet"
- Leave everything else default
- Press "Review + Create" and create the VM

![image](https://github.com/user-attachments/assets/3ea643b6-5187-4432-bb56-7f77f01fbeb3)

- Now do the same thing again to create our client-1
  - Create a new Virtual Machine
  - Change the VM Name to "client-1"
  - Pick "Active-Directory-Lab" for Resource Group
  - Pick the same region you did before (East US 2)
  - For image pick "Windows 10 Pro, version 22H2 - x64 Gen2"
  - Pick anything with 2 vcpus for size ("Standard_D2s_v3")
  - Create a Username and password, make sure to write it down somewhere.
  - Check the licensing box at the bottom
  - Click next 2 times to get into the "Networking" section
    - Put the VM into the "Active-Directory-VNet"
    - Leave evetything else default
  - Hit "Review + Create" and create client-1

- You should now have 4 Resources on your home page

![image](https://github.com/user-attachments/assets/941376a9-43aa-4d6a-b469-a107c2b98c3e)

- Now we need to set our Domain Controller’s NIC Private IP address to be static.
  - Go to the Virtual Machine tab in Azure
  - Select dc-1
  - Open the "Networking" dropdown
  - Click on "Network Settings"
  - Click on the "Network interface / IP Configuration"

![image](https://github.com/user-attachments/assets/1c447626-6902-43be-8fc4-1d1a67da8521)

- Once inside click the "settings" dropdown
- Click on "IP Configurations"
- Click on "ipconfig1"

![image](https://github.com/user-attachments/assets/066ce5b2-ff68-4f77-ab22-a05d9917ff34)

- Change the "Private IP address settings from dynamic to static
- Hit save at the bottom
- (This ensures that dc-1's private IP address never changes)

![image](https://github.com/user-attachments/assets/7bc40b51-50ab-42c7-8260-7688e16814b4)

</p>


<h2>Connecting to dc-1 and Disabling the Firewall</h2>

<p>

- Now lets connect to dc-1, follow [this tutorial](https://github.com/MatthewThompsonIT/creating-virtual-machines?tab=readme-ov-file#how-to-connect-to-the-virtual-machine) if unsure how to use remote desktop.
- In short, Open remote desktop, get the public ip address of dc-1, use that ip address and the username + password you created when making the VM to log into the domain controller
</p>

<p>

- Right click the windows icon on the bottom left of the screen and hit "run" or press the windows key + R to open run

![image](https://github.com/user-attachments/assets/c56b0d2c-f20a-46f3-b2a3-01898846a89b)
  
  - type "wf.msc" and hit enter, this will open the windows firewall

![image](https://github.com/user-attachments/assets/3dc21cd6-e39f-430f-b77a-b7d944bc3592)

- For this tutorial we are going to disable the firewall on dc-1
  - Go through each of the tabs except "IPsec Settings" and change the "firewall state" to off
  - Hit "Apply"

![image](https://github.com/user-attachments/assets/8f1bcd2f-ccdc-401f-9766-94c45c8c44ca)

</p>

<br />

<h2>Configuring client-1</h2>

- We need to first get the private IP address of dc-1
  - Go back to the virutal machine tab
  - Click on dc-1
  - Make sure you are in "Overview"
  - Copy the "Private IP address"

![image](https://github.com/user-attachments/assets/fca6efc7-5a0d-4c2c-bbbf-6873dec03249)

- Click on client-1, go to network settings, then click on the "Network interface / IP Configuration"

![image](https://github.com/user-attachments/assets/877fe5c3-f5dd-4ba6-b95a-b00f77da3080)

- Open the "Settings" dropdown
- Click on "DNS servers"
  - Change the "DNS server" to "custom" instead of "imherit from virtual network"
  - Paste in the Private IP address of dc-1 (for me 10.0.0.4)
- Hit save at the top

![image](https://github.com/user-attachments/assets/0dbb1b24-f59c-4cae-9797-cde6e63ae6f3)

- Restart client-1
  - In the VM portal click the check box next to client-1
  - Hit restart at the top

![image](https://github.com/user-attachments/assets/56c315f8-12dd-4184-bd4d-ef69330ede1f)

- Lets try to ping dc-1 from client-1
- Copy the public IP address of client-1 and remote desktop connect to it
- Get dc-1's private IP address, it is the same one we pasted in earlier (10.0.0.4)
- Open up "powershell" in client-1 by typing in powershell into the start menu in the bottom left
  - (Note: if you get a timeout or it says "destination host unreachable" make sure that dc-1 and client-1 are in the same Virtual Network, that you copied the private IP address of dc-1, and that you properly disabled the firewall in dc-1.)
- If you successfully see the response ping then continue if not, go back and check the steps in the note above.

![image](https://github.com/user-attachments/assets/6a224bb3-705b-424a-bfa6-80bd93fd55b0)

- Lets check that the DNS of client-1 was properly changed to dc-1
- type "ipconfig /all" into the powershell of client-1

![image](https://github.com/user-attachments/assets/963e677d-cf80-49e1-880f-39dc8f257112)

- Scroll through the wall of text until you see "DNS Servers" near the bottom
- Ensure that its the same address as dc-1's private IP address. (10.0.0.4)

![image](https://github.com/user-attachments/assets/1f124749-298d-4fd0-a1c5-270da9541bd4)
  


<h2>Deploying Active Directory</h2>




<h2>Creating Users with PowerShell</h2>



<h2>Group Policy and Managing Accounts</h2>
















