<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. (Video demonstration not yet published)<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setting Up Resources
- Deploying Active Directory
- Configuring Active Directory
- Joining Our New Domain
- Creating Users With PowerShell

<h2>Setting Up Resources</h2>

First, we must set up and deploy resources. 
The diagram below will display how we may configure said resources.

Navigate to Azure services at the dashboard and select virtual machines to get started.

<img src="https://github.com/user-attachments/assets/06a737b4-1afc-471c-bf59-7aa678921130">

For this project we will need:

- 1 Windows 2022 Server (this will act as our Domain Controller: LabDC)
- 1 Windows 10 Client (Workstation: Client-1)
- 1 Virtual Network (created automatically upon set up)

All within an Azure Resource Group (created upon set up).
Make sure all resources are in the same resource group, virtual network, and region. For this lab, I used US EAST 1.

(If you need help creating these resources you can reference my Network Security Groups and Inspecting Network Protocols in Azure project for more specific instructions.)

The Windows server cannot have a dynamic IP address because our client computer will need to locate it consistently.
For this reason, we will assign a static IP. 

![image](https://github.com/user-attachments/assets/c211a6ab-d4ef-4905-a67b-83b725cb63b6)

Setting up our resources in this way will allow the client computer to utilize the soon-to-be Domain Controller for Domain Name Services (DNS) and join our domain successfully later on in this process.
 
To accomplish a static IP configuration for the Server, navigate to the Network interface settings.

![image](https://github.com/user-attachments/assets/ffd0cd25-e86d-42c6-9353-2652e3f819a9)
![image](https://github.com/user-attachments/assets/b98a8d18-baaf-4f92-977b-2d3e475d6da4)

Set the static IP as shown above and we are good to go! Here is what the Resource Group looks like. (Note: Both the Client and DC utilize the same Virtual Network)

![image](https://github.com/user-attachments/assets/900c0345-e379-42b8-bd43-0709a497f991)

Note: It is important to verify both virtual machines are a part of the same virtual network before continuing any further.

Next, we will ensure connectivity between the Server and the Windows 10 Client computer by sending a ping from the Client. 

To do this, remote into both machines via the Remote Desktop Protocol port 3389 (left open for testing purposes by default in Azure unless otherwise disabled.)

We can use the help of the connect feature provided by the Azure GUI or manually remote in via our host computer by searching for Remote Desktop Connection. Enter the public ip address of the VM (found in the settings page of the VM) and the credentials we created when deploying the virtual machines at the prompts. (this guide assumes you are using Windows.)

![image](https://github.com/user-attachments/assets/d6f3ab81-528d-47e6-b847-3f7a644c0c70)


![image](https://github.com/user-attachments/assets/75041719-f264-442f-a42b-b47ba799b04f)

Next, we will configure the local firewall (search for wf.msc on the taskbar) on the Server to allow ICMPv4 echo requests because it is the protocol ping uses to send packets. Without making this configuration the server will block the request.

![image](https://github.com/user-attachments/assets/66558849-e24a-4d4f-8bca-f3bacf496078)

![image](https://github.com/user-attachments/assets/3910db5e-898a-4450-93ba-ff51a0f8efa4)

With the ICMPv4 inbound rules enabled to allow echo requests, we successfully pinged the server's statically assigned private IP to verify connectivity (all 4 replies from DC shown above).

Everything is working as it should! Let's set up Active Directory.



<h2>Deploying Active Directory</h2>

Next, we will install Active Directory Domain Services and promote our Server to a Domain Controller.

To do so we will navigate to Server Manager>Manage>Add Roles and Features (alternatively we could use the quick start menu).

![image](https://github.com/user-attachments/assets/d75b2b2f-b1b9-49c3-8004-1ae9e3df5505)

Hit next a few times in the wizard selecting the default options and our Server, then select Active Directory Domain Services. Add the default features when prompted
(this will include DNS.)

![image](https://github.com/user-attachments/assets/a3420453-517d-44ee-bd52-30fe1b6a538c)

Allow the installation to complete.

![image](https://github.com/user-attachments/assets/fbae97ca-c393-42a6-a9f2-99ed160a9ab9)

Then navigate to the flag/notification center icon and select Promote this server to a Domain Controller.

![image](https://github.com/user-attachments/assets/bca3293e-e2c8-4747-a133-e1cfbef795b4)

Add a new forest and name the Domain. Hit next, set the domain admin password, and continue through the prompts to complete the installation.

The Server will automatically restart after completion, kicking you out of the remote session.

<h2>Configuring Active Directory</h2>

Now that we have promoted our Server to a Domain Controller, we can start configuring Active Directory.

Reconnect to the newly promoted domain controller with the same local credentials OR with domain name/username will work.

First things first, we will create two organizational units (OUs) and one respective user per OU.

OUs contain resources such as user accounts, security groups, and computers/devices connected to the domain. (or additional OUs depending on structure)
They are often organized by departments or regions depending on the size and scale of the company.

to get started, open Server Manager and navigate to Tools>Active Directory Users and Computers on the taskbar. (you can also search for it, or open the Windows Administrative Tools folder to find it there.)

![image](https://github.com/user-attachments/assets/ba962650-7db3-43e8-872f-bd5b08221240)

Click the drop-down on the domain to view all of your default organizational units. It is important to be familiar with this tool so take a look around.

![image](https://github.com/user-attachments/assets/9591d07c-2cd6-48d0-a326-58a9dc709cf3)

In this demonstration, I will create my resources as follows:

OUs:
- _Admins
- _Employees

Users: 
- a-rbradford (a- will indicate Administrative user account)
- msmith (this account will be used for non-privileged access to the domain. a standard user with only enough access to do their respective job.)

Replace names as you please if you are following along on your own.

With the domain container selected hit the highlighted button to create a new OU.
(You can also right-click the desired container and create new users, groups, or OUs that way.)

![image](https://github.com/user-attachments/assets/2975d62c-4189-4d92-a59b-29f6473f9e58)

you will see the object prompt, enter a name, and hit ok. For demonstration and practice purposes feel free to uncheck the "protect container from accidental deletion" option

when creating OUs to easily delete the OU if needed. In a production environment, it is not recommended but this is a lab environment, break whatever you'd like!

![image](https://github.com/user-attachments/assets/3d4f7a69-50d4-4b2a-9141-84a120cd6fd0)

To create a user in your new OU make sure to highlight the container you created and hit the highlighted button.

![image](https://github.com/user-attachments/assets/0d72b3a8-b890-48dd-8bc1-5def890aa7e2)

Enter first and last name. for user logon name a common naming structure is first initial last name, but you can use whatever structure you'd like.
hit next to configure password and settings.

![image](https://github.com/user-attachments/assets/45f42311-7945-494d-8943-d735f4c33a1f)

Usually, in a production environment, you would assign a temporary password and prompt the user to create a new password at first log on.
In the lab environment, we set the password to never expire and use the same password for everything to streamline processes.

![image](https://github.com/user-attachments/assets/67c4229b-424f-45a0-86c9-54b6373c177c)

The account we created will need administrative privileges to make system changes. so let's grant this user root access. (obviously not recommended in production).

To do so, add the user to the built-in Domain Admins group. Right-click the user and select Add to Group.

![image](https://github.com/user-attachments/assets/a6e4d1b9-b61d-4dcd-a600-3af625435a93)

Type "domain admins" in the box and select check names. It will auto-complete if the group exists. if not you will be prompted that it does not exist. 

Hit ok, a pop-up will let you know that you have successfully added the user to the group. Let's verify that by viewing the properties of our _Admins user. 

![image](https://github.com/user-attachments/assets/a311c2be-5082-41b9-a8b7-df8152f9e96b)

In the properties menu, select the "member of" tab to see what groups the user is a part of. In production, this is one way to show what level of access a user has.

(Permissions are typically assigned at the group level to streamline privilege and access management for administrators.)

![image](https://github.com/user-attachments/assets/19d2ebff-0521-46f9-a859-285c4d5fbd55)

We have verified the admin user is a part of the Domain Admins group, granting this account the highest level of access.

From this point forward we can now use this account as an admin. Let's log out of the local user and reconnect to the server with RDP.

![image](https://github.com/user-attachments/assets/9749cb06-a677-4647-ae4d-067c08fc444d)

Connect to the server's public IP again, when prompted to log in select more choices>use a different account.
enter domain name\ admin username and password to log in as the admin account we created.

![image](https://github.com/user-attachments/assets/eb0cb96f-fb21-4d80-848f-a320a33d48ef)

Once logged in, we can open the command prompt (cmd.exe) and type in the whoami command to verify.


<h2>Joining Our New Domain</h2>

Let's join the domain with Client-1, the Windows 10 machine we created.

For this to work the Windows 10 Client needs to be using the Server we created for DNS services. 
By default, the virtual network is using a public DNS server provided by Azure. If we try to join our domain using the default public DNS server it will not find our domain because it is not out on the public internet, it is contained within our private virtual network. 

To change our DNS server lets navigate to the Network Interface settings within Azure for Client-1 like we did earlier for the Domain controller.

Go to the resource group, click on Client-1 virtual machine. Then select Network Settings on the left, and select the network interface.

![image](https://github.com/user-attachments/assets/a1aa0dfc-b6bb-40bf-96ec-b21652f8bbcc)

Now select DNS servers in settings, change the setting from inherit from virtual network to custom. Enter the private IP for our AD Server/ Domain controller that we statically assigned earlier, and hit save.

Once saved you will be prompted after the change is successful.

![image](https://github.com/user-attachments/assets/af99f897-5376-4e24-bf07-ca7d7d01dcbe)

Next, we need to restart Client-1 to make sure the network setting changes are successful. Go back to the virtual machine settings for Client-1 and hit restart.

![image](https://github.com/user-attachments/assets/11cac910-d41e-476c-99f7-93dee10dbdad)

Connect to Client-1 via RDP as we did earlier using the public IP. Use the same local credentials as we did originally because Client-1 is not yet a part of the domain. Once logged in, let's join the domain. 

Right-click the Windows start menu and select System.

![image](https://github.com/user-attachments/assets/2944b76c-acec-414b-9274-0d1ca4b84509)

From the system settings, find the rename this pc (advanced) hyperlink on the right-hand side and select it.

![image](https://github.com/user-attachments/assets/a776b5ee-cf48-4d34-9afc-68e0184ddda8)

From the system properties window, rename this computer or change its domain or workgroup. click Change. Select the Change... button.

![image](https://github.com/user-attachments/assets/6b7f9b81-fd1b-4b93-9020-6542284d10f6)

Select Domain: under member of and enter the domain you used for the Active Directory forest. Hit ok.

When prompted for user credentials, use the domain admin account we created earlier to log in.

![image](https://github.com/user-attachments/assets/f1f0d8f3-00ad-44a4-8118-e7ccdc5f8bc4)

You should see this prompt. Hit ok and then hit ok again when prompted to restart.

![image](https://github.com/user-attachments/assets/d1f7b9ac-dc99-4e92-b221-de4f3c3a2077)

Once restarted, You can log in the same way we did with the server by using our domain admin user credentials. If we choose.
(Note to log in to the Client-1 with our Admin Account we would have to be logged out of the Domain Controller.)

Client-1 has now joined the domain! 

Let's go ahead and verify that by navigating to users and computers back on the Domain Controller. Client-1 will show up in the built-in Computers OU.

Navigate to Server Manager>Active Directory Users and Computers. Select the drop-down for the Domain and then select the Computers OU.

![image](https://github.com/user-attachments/assets/235065dc-cbd3-4fa7-909e-360f506311f3)

There it is! Client-1 has joined the domain and we can verify it is there. 

As a side note, if you delete the Client-1 Computer and attempt to log in you will receive a message that the trust relationship to the domain has been broken.
This means the computer has fallen off of the domain. To rejoin the domain, log into the Windows 10 system with local credentials and rejoin the domain the same as before.

Now, you may wonder why we might have chosen to use the Admin account when joining the domain instead of a standard user account.
Standard users do not have access to Remote into any systems on the domain. To log in as a standard user this will have to change.

Log into Client-1 with the Admin Domain user. Navigate to system settings the same way we did when joining the domain. 

However, this time select Remote Desktop and then the hyperlink for Select users that can remotely access this PC.

![image](https://github.com/user-attachments/assets/70be7041-ba67-4784-8828-760e62f23655)

Then select Add.

![image](https://github.com/user-attachments/assets/903cf44f-f4ce-4905-ba0e-6d4071546901)

Enter "Domain Users" into the box, select check names, and then ok. 

![image](https://github.com/user-attachments/assets/53f77c8d-441d-4e2a-84d7-e037ee4da744)

Now you can see all users in the Domain Users group can remotely access this PC.
Hit ok.

![image](https://github.com/user-attachments/assets/88711663-671c-4a7e-9b90-bc8299959ef0)

To verify this, let's log in as the standard user we created earlier (Mike Smith). He is a part of the Domain Users group in our domain so we should now have no problems logging in.

![image](https://github.com/user-attachments/assets/e70f7704-99b7-494b-bed0-e82ad34d6276)

![image](https://github.com/user-attachments/assets/b967fd7d-e07c-49e1-b57a-9e7951df791a)

Keep in mind. For this lab demonstration, we modified the remote desktop settings at the LOCAL desktop level. 
In a real environment, this is typically done within the Group Policy settings for the domain. (This is beyond the scope of this lab).

We have created an Active Directory Domain Controller and successfully joined the domain with a standard user account. 

From here you can experiment with Users, OUs, Groups, Group Policy, Network Shares, etc as you please. You can treat this as a learning environment with no consequences.


<h2>Creating Users With Powershell</h2>

This next part of the lab is completely optional if you are following along on your own.

Next, we will use a PowerShell script to create 1000 randomly generated users within our employees' OU to simulate a real environment.

The additional users can be organized into OUs, and assigned different levels of permissions within Security Groups, and you can login to any of them as you please.
You can even add additional client computers and join them to the domain as well, the possibilities are endless. 

To get started, we will log in to the DC with our Domain Admin account. 

Type "Powershell ISE" into the search bar and right-click Run as administrator. 
ISE stands for Integrated Scripting Environment.

![image](https://github.com/user-attachments/assets/50b94f35-5f0e-4864-b3b7-fcdab5382858)

This is a very powerful tool used to automate different administrative tasks via a .PS1 file extension scripts.
Once again this is beyond the scope of our lab, if you are interested in learning more about Powershell there is a lot you can accomplish with it.

To use the script created by Josh Madakor navigate to https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1 and select raw contents.
Select all and hit copy, we will paste this script into a new file in our Powershell ISE window. 

Once pasted, make sure to change the OU name within the script to your desired OU where the generated users will reside. It is case-sensitive.

Also, you can change the number of accounts to create at the third line near the top. By default the script is set to create 10000 users, I will create 1000.

![image](https://github.com/user-attachments/assets/6c43d30f-8bb6-4ac9-8477-02815d5d7fd8)

If you are interested in how this script works take some time to review it. By looking at it you might have an idea of what is happening when it is executed. The syntax in Powershell is relatively descriptive.
If you do not understand any programming concepts it may be confusing but it just might pique your interest. I will not be explaining each line of this script but I will provide a summary.

Within a function: It will generate two names with alternating consonants and vowels at a minimum of 3 letters and maximum of 7 set.
It will use each name and convert it to a string of text to store it within the respective variables ($firstName $lastName). 
It will then create a username with the stored strings, and pull a set variable for the password ($PASSWORD_FOR_USERS = Password1 at the top).
then with the generated username and preset password it will create an AD user with the name and place it witin the selected path, which is OU=Employees.
Each time this loop occurs it will add 1 to a counter.
This process will loop until the number in the counter reaches the number set at the top ($NUMBER_OF_ACCOUNTS_TO_CREATE = 1000). 

Once you have changed the desired number of accounts to create and which OU to place them in, run the script.

![image](https://github.com/user-attachments/assets/731c5581-abbb-4aef-a2ba-9295e58bd967)

Now you can see the script is creating the users at the bottom window.

![image](https://github.com/user-attachments/assets/71a5c775-4bb0-4eeb-8499-d2679749bb1f)

This may take some time depending on how many users you created. However you can verify the users are there in AD Users and Computers.

![image](https://github.com/user-attachments/assets/6e81efd4-a9be-490e-a927-4f7b0b9f686a)

You can log in to any of these users the same way we did with Mike Smith earlier in this Lab. Keep in mind it will use the password set by the script (Password1).

For additional practice:
You can reset passwords on these accounts, lock out accounts with too many password attempts and unlock them, map share drives, add to security groups for more or less permissions, set dates to disable, experiment with group policy for passwords and applications, etc.

![image](https://github.com/user-attachments/assets/b43db937-4c74-48a4-b7e4-699a75792e2f)

Thank you for reading, I hope you gained some value from this documentation. 

Check out my other projects or connect with me on LinkedIn with any questions and I would be happy to answer!



