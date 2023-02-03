![](images/activedirect.png)
# Microsoft Active Directory | Corporate Network Environment

### Summary
Ative Directory (AD) is Microsoft's service to manage Window's domain networks. It is what most enterprises build their IT departments on.
The Active Directory database contains critical information about the environment and the services control much of the activity that goes on within the IT environment.
It connects users to the network resources they need by implementing organization and security measures. It authenticates users with the user ID and password,
and authorizes them to access only the data they are allowed to by using permission rules.
This is how the same set of credentials can be used to log into any Windows workstation within a given institution. 
This lab's purpose is to understand how Active Directory and Windows networking work by setting up a virtual environment with 1,000+ users.

### Learning Objectives:
- How to provision multiple virtual machines on VirtualBox
- Install and configure Windows Server 2019
- Create a Windows 10 ISO with Microsoft's media creation tool
- Install and configure Windows 10
- Install and configure Microsoft's Active Directory Domain Services (AD DS) to create a basic Windows network
- Create 1,000+ users with Windows PowerShell

### Tools and Requirements:
1. Oracle VirtualBox
2. Windows Server 2019 ISO
3. Windows 10 ISO
4. Directory service: Active Directory Domain Services (AD DS)
5. PowerShell


### Overview:
![](images/win_active_dir.png)

## Step 1: Download and Install [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Select the appropriate download based on the current host OS
- Locate the VirtualBox installer file with File Explorer
- Install VirtualBox with the Wizard defaults or desired preferences
- Download the VM VirtualBox Extension Pack
- Click on downloaded Extension Pack and hit **Install** when prompted
> NOTE: This setup will use a Windows 11 host OS.

![](images/virtualbox_download.png)

## Step 2: Download [Windows Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- Select the appropriate language (English)
- Under ISO downloads select 64-bit edition
- Save to desired location

![](images/win_server_download.png)

## Step 3: Download Microsoft's [Media Creation Tool](https://www.microsoft.com/en-us/software-download/windows10) and Windows 10 ISO
> NOTE: In order to download the Windows 10 ISO the assistance of the media creation tool is required.

### Create Windows 10 Installation Media 
- Select **Download Now**
- Locate the executable installer and install the Media Creation Tool
- Follow the Wizard intall prompts
- Once it installs follow the the prompts to download the Windows 10 ISO file.

![](images/create_win_inst.png)

### What do you want to do?
- Select **Create installation media (USB flas drive, DVD, or ISO file) for another PC**
- Click **Next**

![](images/create_media.png)

### Select language, architecture and edition
- **Language:** English (United States)
- **Edition:** Windows 10
- **Architecture:** 64-bit (x64)
- Click **Next**
> NOTE: Keep recommended options, otherwise uncheck button to edit selections.

![](images/create_media_2.png)

### Choose which media to use
- Select **ISO file**
- Click **Next**
- Select the destination folder to save the Windows 10 ISO file
- Hit **Save** and wait for download to finish
- Click **Finish** to exit the program

![](images/create_media_3.png)

## Step 4: Create and Configure Domain Controller Virtual Machine
### Virtual Machine Name and Operating System
- Open VirtualBox and select **Machine** to bring a drop down menu
- **Name:** Domain Controller
- **Type:** Microsoft Windows
- **Version:** Other Windows (64-bit)
- Hit **Next**

![](images/new_vm.png)

### Hardware
- **Base Memory:** 2048 MB (2GB)
- **Processors:** 2 CPUs
- Click **Next**
> NOTE: Options depend on host computers available resources.
### Virtual Hard Disk
- Leave default Create a Virtual Hard Disk Now 
- **Disk Size:** Default - 20.00 GB
- Select **Next**
### Summary
- Review and confirm selections are correct
- Click **Finish** to complete VM creation
### Settings
- Once the VM has been created, select the VM and click **Settings**
- Go to **General** > **Advanced** > **Shared Clipboard** > **Bidirectional**
- Go to **General** > **Advanced** > **Drag'n'Drop** > **Bidirectional**
- These options will enable copying and pasting, as well as dragging and dropping between host and VM
- Go to **Network** > **Adapter 1** > **Attatched to:** NAT
- Go to **Network** > **Adapter 2** > Check Enable Network Adapter Box > **Attatched to:** Internal Network
- Click **Ok**

![](images/dc_properties.png)

## Step 5: Install Windows Server 2019 and Change PC Name
- Double click the **Domain Controller** VM
- It will fail to boot because it is missing the OS
- Select the DVD dropdown menu and find the Windows Server 2019 ISO file
- Click **Mount and Retry Boot**

![](images/mount_iso.png)

### Collecting information
- Leave **Language** **Time** and **Keyboard** as defaults and click **Next**
- Click **Install Now**
- Select Windows Server 2019 Standard Evaluation (Desktop Experience)
- Click **Next** 
- Read and accept license terms, then click **Next**
- Select **Custom: Insall Windows only (advanced) and click **Next**
- Allow time to install Windows and wait until it boots up into Windows
- Create a password for the Administrator account. Use `Password1` for lab ONLY and click **Finish**
- When prompted to press **CTRL+ALT+DEL** go to the menu bar above the VM and select **Input** > **Keyboard** > **Insert CTRL+ALT+DEL**

![](images/unlock.png)

- Login with the **Administrator** account and the password: **Password1**
- Close out of notifications and pop ups
- On the upper menu of the VM select **Devices** > **Insert Guest Additions CD Image**

![](images/guest_additions.png)

- Open **File Explorer** > **This PC** > **CD Drive D: VirtualBox Guest Additions** > **VBoxWindowsAdditions-amd64**
- Follow installer prompts with default options
- Select **I want to manually reboot later** and click **Finish**
- Right-click the **Windows icon** on the bottom left and select **System**
- Press **Rename this PC** and name it "DC"
- Select the **Restart later** button 
- Manually shutdown the VM and restart


## Step 6: Configure Internal and External NICs
- Select the network icon on the bottom right of the screen and click **Network**
- Press **Change adapter options**
- Right-click the **Unidentified Network** > **Status** and click **Details**
- It will not have internet access because the adapter is looking for a DHCP server that has not been configured yet

![](images/no_dhcp.png)

- Rename adapters accordingly:
  - Ethernet 1: INTERNETexternal
  - Ethernet 2: INTRANETinternal
  
![](images/nic_names.png)

- Right-click the internal network adapter and select **properties**
- Select **Internet Protocol Version 4 (TCP/IPv4)
- Select ** Use the follwoing IP address in order to enter it manually
  - **IP:** 172.16.0.1
  - **Subnet Mask:** 255.255.255.0
  - **Default Gateway:** <Empty> 
  > NOTE: The Domain Controller will serve as the Default Gateway
- Installing Active Directory automatically installs DNS therefore the server will use itself as the DNS server
- For **Use the following DNS server addresses**
  - **Preferred DNS Server:** 127.0.0.1 (Loopback address)
- Click **Ok** and **Ok** again

![](images/ip_configs.png)

## Step 7: Install Active Directory Domain Services (AD DS) and Create a Domain
- If not already open, search for **Server Manager** within the bottom searchbar
- Select **Add roles and features**
- Continue to click **Next** until it prompts to select a server to install it to which should be the default and only selection available
- Check only the **Active Directory Domain Services** box, click **Add features** and click **Next**
- Continue to click **Next** with the default options and finally **Install** at the end
- Hit **Close** once it has finished
- Within the Server Manager Dashboard click the **Notifications Flag Icon** that can be found on the upper right corner
- Click **Promote this server to a domain controller**

### Deployment Configuration
- **Select the deployment operation:** Add a new forest
- Under **Specify the domain information for this operation** give the domain a name
- **Root domain name:** "mydomain.com"
- Press **Next**
### Domain Controller Options
- Under **type the Directory Services Restore Mode (DSRM) password** you would put the restoration password
  - **Password:** Password1
- Continue to click **Next** with all the default fields until **Install**
- Wait for it to restart and log back in
- Click the Windows start icon and select **Windows Administrative Tools** > **Active Directory Users and Computers** 
- Right-click **mydomain.com** > **New** > **Organizational Unit**


