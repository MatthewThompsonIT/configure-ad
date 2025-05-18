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

- Open up the "Server Manager" in dc-1
- Hit "Add roles and features"

![image](https://github.com/user-attachments/assets/6c4bb56a-a1f7-4f1b-aa61-2b1323ebf744)

- Leave everything default and hit next until you get to "Server Roles"
  - Check the box next to "Active Directory Domain Services"
    - Hit "Add Features"

![image](https://github.com/user-attachments/assets/ebac6fb8-98b7-443d-bfe0-89a53a45b1b1)

- Continue hitting next until you get to "Confirmation"
  - Check the box next to "Restart the desination server automatically if required"
- Hit "Install"
- Active Directory is now installed so lets configure it.

- Click the flag in the top right of the Server Manager
- Click "Promote this server to a domain controller"

![image](https://github.com/user-attachments/assets/23e5304e-7414-4a86-bffc-cf6eec3e0f74)

- In the "Active Directory Domain Services Configuration Wizard"
  - Change the "deplotment operation" to "Add a new forest"
  - Name the domain whatever you would like it to be called (I use "mydomain.com" but it can be anything you like")

![image](https://github.com/user-attachments/assets/83f506f5-1810-4cec-880c-ff1d141ad625)

- Hit next
- Under Domain Controller Options, enter in a password for the "Directory Services Restore Mode" and write it down just incase you need it (you wont need it for this tutorial)
- Hit next and make sure "Create DNS delegation" is UNCHECKED
- Comtinue clicking "next" until you get to "installation"
- Hit Install
- dc-1 will restart
- When logging back into dc-1 we will now be adding the domain infront of our username (mydomain.com\labuser)
> [!NOTE]
> Make sure you are using "\" and not "/" when trying to log in.


![image](https://github.com/user-attachments/assets/ad715702-da81-49ca-b7f9-6ade7cd745d3)

- Now lets create a Domain Admin user within the domain (Pay attention of this part as you do NOT want to mess it up)
- In Active Directory Users and Computers (ADUC)
  - Type "Active Directory Users and Computers" into the search bar
- Lets create two Organizational Units (MAKE SURE YOU SPELL THESE EXACTLY AS I DO)

![image](https://github.com/user-attachments/assets/7648ae2c-edca-41f0-beda-51151e860a98)

- Create an Organizational Unit (OU) called “_EMPLOYEES”


![image](https://github.com/user-attachments/assets/2cb905cb-9ab8-4338-9a9a-56929f56faa9)


- Create a new OU named “_ADMINS”

![image](https://github.com/user-attachments/assets/296721ce-906a-4a7d-8bd1-5441328de3da)

- Create a new employee named “Jane Doe” with the username “jane_admin”
  - Click on the _ADMINS folder and right click in the blank space to the right
  - Choose new, then User

![image](https://github.com/user-attachments/assets/833efa7d-8101-445c-a7c9-19a4bcc8c1ef)

  - Fill out the First and Last name, and create a username
  - Hit next then create password
  - Uncheck all 4 boxes under the password
  - Hit next and then finish

![image](https://github.com/user-attachments/assets/29a7ffbd-36a6-406d-9d69-559c4a8386f3)

- Add jane_admin to the “Domain Admins” Security Group
  - Right click the new account you created and hit "Properties"

![image](https://github.com/user-attachments/assets/50065342-cfee-46fc-bf3a-5f6dd35ad750)

  - Choose the "Member Of" tab at the top
  - Hit "Add..."
  - Enter in "Domain Admins"
  - Hit "Check Names"
  - Press "OK" then "Apply"

![image](https://github.com/user-attachments/assets/5d2775d8-ac7b-496f-9b21-4d2e314a4c04)


- Log out / close the connection to DC-1 and log back in as “mydomain.com\jane_admin”
- Use jane_admin as your admin account from now on


- Now lets join client-1 to our domain
- Open up the system settings (Right click the windows icon in the bottom left and hit system) on client-1 and go to the bottom section "About"
- Click on "Rename this PC (advanced) on the right

![image](https://github.com/user-attachments/assets/31f99cc4-8c77-4572-9f03-b784a4fd73f3)

- Click on "Change..." under "Computer Name"
- Change the name to whatever you named your domain earlier (mydomain.com)
> [!NOTE]
> If you get an error message instead of a log in panel then you entered in the wrong domain or set up the DNS settings wrong.


![image](https://github.com/user-attachments/assets/405bf414-28ed-42dc-90e3-eb0dd2e93b2f)

- Enter in the info you used for your admin account (mydomain.com\jane_admin)
- You will get a welcome message and be asked to restart client-1
- Restart client-1 and log back in with the admin account
-Ensure that client-1 was added to the computers folder inside of the Active Directory

![image](https://github.com/user-attachments/assets/3521a7ba-7439-4f22-868b-84b54b2ded9b)

- Create one more OU called "_CLIENTS" and put client-1 in it

- Open up system once more and this time click on "Remote Desktop"

![image](https://github.com/user-attachments/assets/0898b654-9bf6-4645-94d2-eb7073bb2893)

- Click on "Select users that can remotely access this PC" under "User Accounts"

![image](https://github.com/user-attachments/assets/3c0a0cc8-7e79-412e-b1bc-caadca74a198)

- Click on "Add..."
- Type "Domain Users" and hit "Check Names"
- Hit "OK"

![image](https://github.com/user-attachments/assets/478042a6-9c5b-4c9e-ab98-0b838a8a75a4)


<h2>Creating Users with PowerShell</h2>

- Open PowerShell ISE as an administrator (Type it in the search bar, right click it, and hit "run as administrator")
- Hit Yes on the prompt
- At the top of the PowerShell hit the file icon to create a new file

![image](https://github.com/user-attachments/assets/fd152bf7-5631-4426-8a7b-6a0ac632e2f6)

- Hit Control + S on your keyboard to save it (File --> Save As)
  - Name it create-users
  - Save it on the desktop or just anywhere you can easily find it

- Copy and paste this script into PowerShell
  - This will generate around 10000 users with random names for us to use

https://github.com/MatthewThompsonIT/configure-ad/blob/8b02bc53fcc3987e5a49636828eb099155f17848/Generate-Names-Create-Users.ps1#L1-L46

> [!NOTE]
> The path on line 43 MUST be the same name as what you named the employees folder in the Active Directory or the script will not work
> ![image](https://github.com/user-attachments/assets/f5b3a600-2477-4d8d-ab9e-4bc770d98f5b)


- Run the script by clicking on the green arrow at the top

![image](https://github.com/user-attachments/assets/52056d93-99d5-46d3-acfa-2a6028e01c75)

- It will begin creating 10000 random users for us to use

![image](https://github.com/user-attachments/assets/adb32e33-51ef-46ea-951d-c00900884dca)

- Refresh the _EMPLOYEES OU by right clicking the folder and clicking on "Refresh"

![image](https://github.com/user-attachments/assets/b78f2dd1-46a3-4103-827c-1dbc36b8834c)

- You will begin seeing all the random employees being created
- Pick any random one of them thats easy to remember and write it down
- The password for all users is "Password1"

![image](https://github.com/user-attachments/assets/57c9bc40-c48b-4a09-a4c9-26eb3925b03e)

<h2>Group Policy and Managing Accounts</h2>

- Lets set up Group Policy
- On dc-1, right click the start menu and hit "run"
  - Type in gpmc.msc, this will open the group policy management

![image](https://github.com/user-attachments/assets/8ee9cf37-9e57-4e4c-bff8-f8fffaeb374a)

- Open up the dropdowns until you see "mydomain.com" (or whatever you named your domain)
- Right click on the "Default Domain Policy" and hit "Edit..."

![image](https://github.com/user-attachments/assets/86cba1fd-8aff-4280-8ab9-2e7388e2c915)

- In the Group Policy Management Editor, expand the following
  - Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy

![image](https://github.com/user-attachments/assets/fcd8bea8-0a2d-4935-9726-eb61deb1467e)

- Open the Account Lockout Duration Properties
  - Check the "Define this policy setting"
  - Set it to however long you want the account to be locked out for (30 minutes)
  - Change the "allow administrator account lockout" to enabled

![image](https://github.com/user-attachments/assets/4af406e6-2820-49dd-9272-b4685041dac3)

> [!NOTE]
> It should prompt you to change the "account Lockout threshold" and "reset account lockout counter after" hit yes
> If it does not prompt you then set those manually to what ever you like (I leave it as the prompt said)
> ![image](https://github.com/user-attachments/assets/311cc51f-97d0-47c2-9555-f529eb44fcf5)

- Log into client-1 as your admin account (mydomain.com\jane_admin)
- Open up the Command Prompt by typing cmd into the search bar
- type "gpupdate /force" (This will force the changed we just made to take effect on client_1)

![image](https://github.com/user-attachments/assets/f019cc96-d32b-4bee-bbe9-5fbfecb630d5)

- Log out of the admin account and try to log in with any of the random accounts made earlier
- Type the wrong password in 6 times and you should be locked out of the account

![image](https://github.com/user-attachments/assets/457d6677-87fe-4647-b697-e2deb1d07aeb)

- Go back to dc-1 and in active directory
- Right click on "mydomain.com" and hit "Find..."
- Type in the name of the account you got locked out of
- Click on their name
- Go to "Account"
- Check the box next to "Unlock account"
- Hit Apply and OK

![image](https://github.com/user-attachments/assets/4eadf278-735a-41cc-8f23-24f4bed6a925)

- You will now be able to log into the account with the right password

- To reset a password its the same steps
- Search the account in the Active Directory
- Right click on the name of the account and hit "Reset Password"

![image](https://github.com/user-attachments/assets/b9a4f182-585b-441f-9050-e4c4c4d09d12)

- You can now change the password for them and unlock their account

![image](https://github.com/user-attachments/assets/292c8688-7330-45f1-9e85-e0caf6eacf5a)

- To Disable/Enable accounts
- Search the account in the Active Directory
- Right click on the name of the account and hit "Disable Account" or "Enable Account"

![image](https://github.com/user-attachments/assets/831cdffb-8558-4c99-b3a0-b76535d6bb3a)

- You will no longer be able to log into that account until it is re-enabled


- To see all the activity we just did
- Type "eventvwr.msc" to open event viewer on dc-1

![image](https://github.com/user-attachments/assets/39eaa551-034e-446f-a563-30fb6287c188)

- Expand "Windows Logs" and then right click on "Security"
- Click on "Find..."

![image](https://github.com/user-attachments/assets/be185b00-c800-4ded-ae61-92ea83bfca41)

- Type in the name of the user account you were just logging in and out of
- Hit "Find Next"
- Go through the logs and see all the logs generated around that account


> [!NOTE]
> The same thing can be done with an admin account on client-1
> On client-1 you will be able to see all of the failed login attempts
![image](https://github.com/user-attachments/assets/298a99f4-c8ca-464e-9148-8eef559de714)


<h2>Clean up</h2>

- Dont forget to turn off the virtual machines / delete them when done with the tutorial to save money
- Follow [this guide](https://github.com/MatthewThompsonIT/creating-virtual-machines?tab=readme-ov-file#cleanupexiting-the-vm) to learn how to delete / disable VMs in Azure

