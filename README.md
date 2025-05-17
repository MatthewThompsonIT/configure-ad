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

- Setup Domain Controller and Client-1 in Azure
- Install Active Directory and join Client-1 to it
- Create a bunch of additional users to simulate a large business
- Enabling, unlocking accounts, and resetting Passwords

<h2>Setup of Resource Group, Virtual Network, Domain Controller and Client-1</h2>

<p>
  
- Lets begin by creating a resource Group, name it "Active-Directory-Lab" refer to [this tutorial](https://github.com/MatthewThompsonIT/creating-virtual-machines?tab=readme-ov-file#installation-steps---creating-a-resource-group) on creating and configure resource groups and virtual machines.
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

- Now do the same thing again to create our Client-1
  - Create a new Virtual Machine
  - Change the VM Name to "Client-1"
  - Pick "Active-Directory-Lab" for Resource Group
  - Pick the same region you did before (East US 2)
  - For image pick "Windows 10 Pro, version 22H2 - x64 Gen2"
  - Pick anything with 2 vcpus for size ("Standard_D2s_v3")
  - Create a Username and password, make sure to write it down somewhere.
  - Check the licensing box at the bottom
  - Click next 2 times to get into the "Networking" section
    - Put the VM into the "Active-Directory-VNet"
    - Leave evetything else default
  - Hit "Review + Create" and create Client-1

- You should now have 4 Resources on your home page

![image](https://github.com/user-attachments/assets/941376a9-43aa-4d6a-b469-a107c2b98c3e)

</p>


<h2>aaaaaaaaaaaaaaaaaa</h2>


<p>

</p>
<br />

<p>
<img src=""t=""/>
</p>
<p>

</p>
<br />

<p>
<img src=""t=""/>
</p>
<p>

</p>
<br />
