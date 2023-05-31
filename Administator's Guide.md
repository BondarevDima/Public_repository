# 1.	Components of the Visitor Registration System

## 1.1.	Management WebUI
This component is a web site. You can find a link to it on the main page of corporate Intranet website.
This component has the following functions:
- It provides an interface that allow any employee apply for a visitor to Company’s office.
- Provide an interface for employees who provides system management.
**Management WebUI** is implemented in C# and presented as an ASP.NET appendix.


## 1.2.	Processing Service
This component provide automatic processing of confirmed applications for the visits and sent applications to external systems. This component also generates, and sends notications about events occurring in the visitor registration systems to interested parties.
**Processing Service** is an executable module that runs periodically through the Windows Task Scheduler.
**Processing Service** is implemented in C #. The component requires the .NET Framework.

   **Hardware requirements**
|**Component**|	**Description**|
|---------|------------|
|Processor|	x86 or x64 processor with a minimum frequency of 1.4 GHz|
|RAM|	Minimum 512 MB|
|HDD|	1 GB of free space for log files (when logging is enabled)|
|Network adapter|	100 mb/s Nominal Bit Rate|
# 2.	Installation of the visitor registration system
## 2.1.	Installation of Management WebUI
You should to the following to install **Management WebUI** on the target server: 
1.	Creating an account.
2.	Creating and configuring the pool server 
3.	Preparing the Web-files
4.	Creating and configuring the website .

## 2.1.1.	Create an account
You should do the following to create an account on whose behalf the website will operate:
1.	Add a new user or use an existing one. This user should registered in Active Directory database.
2.	Add the user to the USERS group.

### 2.1.2.	Creating and configuring the pool server
You must to do the following to create a new pool in IIS:
1.	Open **Computer Management** application.
2.	In a drop-down list open **Services and Applications** → **Internet Information Services (IIS) Manager** → **Application Pools**.
3.	Open the **Application Pools** context menu by right-clicking and select **New → Application Pool**.
4.	Select the **Use default settings for new application pool** check box, to use the default settings.
5.	Press **OK**.
In the properties of the created pool, specify the following required parameters:
1.	Open the context menu of the created pool by right click and choosing **Properties**.
2.	On the **Identity** tab, check the **Configurable** box and specify the username and password of the account created in the “Create the account“ section. 
3.	For the remaining parameters, leave the default values.
### 2.1.3.	Preparation the Webfiles.
You should do the following on the server before website deployment: 
1.	Created a folder called **Visitor Registration System** anywhere on the server.
2.	Copy the contents of the ```\\ builds \ system \ Release_ <date.version> \ Web ``` folder into it (where date.version is the date and version of the release, for example ```Release_20120218.7```).
### 2.1.4.	Creating and configuring the website 
Create a website in IIS using the Wizard and specify the following parameters:
1.	In the **TCP-port field of this Web site should use**, specify a unique TCP port value within the subnet.
2.	 In the **Path** field specifys the path to the System home folder, where the website is deploid.
3.	Uncheck **Allow anonymous access to this Web site**.
4.	Leave the default values for the remaining parameters.

## 2.2.	Installation Active Directory Federation Service
You should to the following to install **Active Directory Federation Service 2.0 (AD FS 2.0)** on the target server: 

*For Windows Server 2008:*
1.	Run the AD FS 2.0 setup file. Click **Next**
2.	Accept the License Agreement and at the step **"Server Role"** select the role **"Federation Server"**. Click **Next**
3.	After the wizard completes, check the "When the wizard closes, launch the AD FS 2.0 Management snap-in" option to further configure the service.

*For Windows Server 2012:*
1.	Start Windows Server Manager.
2.	From the Dashboard, click **Add roles and features**.
3.	From the Before You Begin page, click **Next**.
4.	For Installation Type, select **Role-based or feature-based installation** and then click **Next**.
5.	Select the **Select a server from the server pool** option, and then choose your local server from the Server Pool list. Click **Next**.
6.	In the Server Roles list, select **Active Directory Federation Service**. Click **Next**.
7.	From the Select Features page, accept the defaults and click **Next**.
8.	On the confirmation page, click **Install**.
9.	Click **Close** when the installation is complete and then restart the server.
10.	From Server Manager, click **AD FS** in the dashboard.
11.	Click the **Configuration required for Active Directory Federation Service** warning indicator.
12.	From the All Servers Task Details and Notification page, click the **Promote this server to a domain controller** action.
13.	Complete steps in the **Active Directory Federation Service** Configuration wizard. Review the summary, click **Next**, and then click **Install**.
14.	After the server is restarted, click **Add roles and features** from the Server Manager. Select **Active Directory Federation Services (AD FS) and Web Server (IIS).**

## 2.3.	AD FS Configuration
### 2.3.1.	Add users to group AD FS (LDAP)
You should to the following to add user on the target server: 
1.	From the Server Manager menu bar, click **Tools → Active Directory Users and Computers**.
The Active Directory Users and Computers MMC opens.
2.	Click the **Create a new user in the current container** icon.
3.	Add user. These user are added to the appropriate groups created in the next step.

### 2.3.2.	Create new user and group
To create a user, follow these steps:
1.	On the **New Object** - User page, type the user's first name, full name. Then click **Next**.
2.	Enter a password and confirm. Select password options for this user and then click **Next**.
3.	Review the configuration and click **Finish**.
To create an group, follow these steps:
1.	On the **New Object** - Group page, type the group name. In Group scope, select one of the following options: Domain local, Global, or Universal. 
2.	Click **OK**.
To add a user to a specific group, follow these steps:
1.	From the Active Directory Users and Computers console, right-click the group in which you want to add the user.
2.	Click **Properties** to open the Properties dialog box.
3.	Click the **Members** tab and then click **Add**.
4.	In **Enter the object names to select**, type the name of the user and group that you want to add, and then click OK.
5.	Repeat these steps to assign group membership for each user that you added.
## 2.4.	Create and add SSL ceftificate 
You should to the following to create or add SSL certificate on the target server: 
1.	From the Server Manager menu, select **Tools → Internet Information Services (IIS) Manager**. 
2.	Click **Server Certificates** and select to **Create Self-Signed Certificate**. 
3.	In the opened window specify a name for certificate and click OK.
4.	In Server Manager menu select **Bindings → Edit → Select new SSL Self-Signed Certificate** and click **OK**.
5.	You need to restart the server. To do this, you must open the **cmd**  and run the command: ```iisreset /restart```
If you have your SSL certificate in format .pfx, you can will be used it. For this necessary:
1.	From the Server Manager menu, select **Tools → Internet Information Services (IIS) Manager**. 
2.	Click **Server Certificate** and select to **Import**.
3.	Select the certificate and enter the password that was entered during creation. In a drop-down list open Select Certificate Store select Personal and check **Allow this certificate to be exported.**

# 3. Some HTTP Status Codes and Possible Solutions
|Code|Description|Possible Solutions|
|----|-----------|------------------|
|400|	Invalid request|	Increase the settings for the MaxFieldLength to 65534 and MaxRequestBytes to 16777216. **WARNING: Increasing these values allow larger HTTP packets to be sent to IIS. This, in turn, may cause Http.sys to use more memory**.|
|401|	Access denied|	Run ```regedit.exe``` and following branch in the registry: ```HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0``` Create a new key: **Type** –  Multi-String Value; **Name** – BackConnectionHostNames; Specify the desired address in the key values.|
|403|	Forbidden|	**Enable the Directory Browsing feature in IIS** <ul><li> Start IIS Manager. To do it, select Start, select Run, type inetmgr.exe, and then select OK.</li><li>In IIS Manager, expand server name, expand Web sites, and then select the website that you want to change.</li><li>In the Features view, double-click Directory Browsing.<li>In the Actions pane, select Enable.|</ul>
|404|	Not found|	<ul><li>Go the Control Panel.</li><li>Click Network and Internet, then Network and Sharing Center, and click Change adapter settings.</li><li>Select the connection For example: To change the settings for an Ethernet connection, right-click Local Area Connection, and click Properties.</li><li>Select the Networking tab. Under This connection uses the following items, select Internet Protocol Version 6 (TCP/IPv6) and then click Properties.</li><li>Click Advanced and select the DNS tab.|</ul>
|500|	Internal server error| **Add the ISAPIModule module to the modules list for the Web site.** <ul><li>Click Start, click Run, type inetmgr.exe, and then click OK.</li><li> In IIS Manager, expand <server name>, expand Web sites, and then click the Web site that you want to modify (where server name name your server)</li><li>In Features view, double-click Module.</li><li>In the Actions pane, click Add Native Module.</li><li>In the Add Native Module dialog box, click to select the IsapiModule check box, and then click OK.|</ul>

