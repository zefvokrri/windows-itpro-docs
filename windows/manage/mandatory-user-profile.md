---
title: Create mandatory user profiles (Windows 10)
description: A mandatory user profile is a special type of pre-configured roaming user profile that administrators can use to specify settings for users.
keywords: [".man","ntuser"]
ms.prod: w10
ms.mktglfcycl: manage
ms.sitesec: library
author: jdeckerMS
---

# Create mandatory user profiles


**Applies to**

-   Windows 10

> [!NOTE]
> When a mandatory profile is applied to a PC running Windows 10, version 1511, some features such as Universal Windows Platform (UWP) apps, the Start menu, Cortana, and Search, will not work correctly. This will be fixed in a future update. 

A mandatory user profile is a roaming user profile that has been pre-configured by an administrator to specify settings for users. Settings commonly defined in a mandatory profile include (but are not limited to): icons that appear on the desktop, desktop backgrounds, user preferences in Control Panel, printer selections, and more. Configuration changes made during a user's session that are normally saved to a roaming user profile are not saved when a mandatory user profile is assigned. 

Mandatory user profiles are useful when standardization is important, such as on a kiosk device or in educational settings. Only system administrators can make changes to mandatory user profiles. 

When the server that stores the mandatory profile is unavailable, such as when the user is not connected to the corporate network, users with mandatory profiles can sign in with the locally cached copy of the mandatory profile, if one exists. Otherwise, the user will be signed in with a temporary profile. 

User profiles become mandatory profiles when the administrator renames the NTuser.dat file (the registry hive) of each user's profile in the file system of the profile server from `NTuser.dat` to `NTuser.man`. The `.man` extension causes the user profile to be a read-only profile.

<span id="extension"/>
## Profile extension for each Windows version

The name of the folder in which you store the mandatory profile must use the correct extension for the operating system it will be applied to. The following table lists the correct extension for each operating system version.

| Client operating system version | Server operating system version | Profile extension |
| --- | --- | --- |
| Windows XP | Windows Server 2003 </br>Windows Server 2003 R2 | none |
| Windows Vista</br>Windows 7 | Windows Server 2008</br>Windows Server 2008 R2 | v2 |
| Windows 8 | Windows Server 2012 | v3 |
| Windows 8.1 | Windows Server 2012 R2 | v4 |
| Windows 10, versions 1507 and 1511 | N/A | v5 |
| Windows 10, version 1607 (also known as the Anniversary Update) |  Windows Server 2016 | v6 |

For more information, see [Deploy Roaming User Profiles, Appendix B](https://technet.microsoft.com/library/jj649079.aspx) and [Roaming user profiles versioning in Windows 10 and Windows Server Technical Preview](https://support.microsoft.com/kb/3056198).

## How to create a mandatory user profile

First, you create a default user profile with the customizations that you want, run Sysprep with CopyProfile set to **True** in the answer file, copy the customized default user profile to a network share, and then you rename the profile to make it mandatory.

**To create a default user profile**

1. Sign in to a computer running Windows 10 as a member of the local Administrator group. Do not use a domain account.

   > [!NOTE] 
   > Use a lab or extra computer running a clean installation of Windows 10 to create a default user profile. Do not use a computer that is required for business (that is, a production computer). This process removes all domain accounts from the computer, including user profile folders. 
 
2. Configure the computer settings that you want to include in the user profile. For example, you can configure settings for the desktop background, uninstall default apps, install line-of-business apps, and so on. 

   >[!NOTE]
   >Unlike previous versions of Windows, you cannot apply a Start and taskbar layout using a mandatory profile. For alternative methods for customizing the Start menu and taskbar, see [Related topics](#related-topics).

3. [Create an answer file (Unattend.xml)](https://msdn.microsoft.com/library/windows/hardware/dn915085.aspx) that sets the [CopyProfile](https://msdn.microsoft.com/library/windows/hardware/dn922656.aspx) parameter to **True**. The CopyProfile parameter causes Sysprep to copy the currently signed-on user’s profile folder to the default user profile. You can use [Windows System Image Manager](https://msdn.microsoft.com/library/windows/hardware/dn922445.aspx), which is part of the Windows Assessment and Deployment Kit (ADK) to create the Unattend.xml file. 
 
3. Use the [Remove-AppxProvisionedPackage](https://technet.microsoft.com/library/dn376476%28v=wps.620%29.aspx) cmdlet in Windows PowerShell to uninstall the following applications: 
 
     - Microsoft.windowscommunicationsapps_8wekyb3d8bbwe
     - Microsoft.BingWeather_8wekyb3d8bbwe
     - Microsoft.DesktopAppInstaller_8wekyb3d8bbwe
     - Microsoft.Getstarted_8wekyb3d8bbwe
     - Microsoft.Windows.Photos_8wekyb3d8bbwe
     - Microsoft.WindowsCamera_8wekyb3d8bbwe
     - Microsoft.WindowsFeedbackHub_8wekyb3d8bbwe
     - Microsoft.WindowsStore_8wekyb3d8bbwe
     - Microsoft.XboxApp_8wekyb3d8bbwe
     - Microsoft.XboxIdentityProvider_8wekyb3d8bbwe
     - Microsoft.ZuneMusic_8wekyb3d8bbwe 
     
     >[!NOTE]
     >Uninstalling these apps will decrease sign-in time. If your deployment needs any of these apps, you can leave them installed.

3. At a command prompt, type the following command and press **ENTER**.

    `sysprep /oobe /reboot /generalize /unattend:unattend.xml` 

    (Sysprep.exe is located at: C:\Windows\System32\sysprep. By default, Sysprep looks for unattend.xml in this same folder.)
    
    >[!TIP]
    >If you receive an error message that says "Sysprep was not able to validate your Windows installation", open %WINDIR%\System32\Sysprep\Panther\setupact.log and look for an entry like the following:
    
    >![Microsoft Bing Translator package](images/sysprep-error.png)
    
    >Use the [Remove-AppxProvisionedPackage](https://technet.microsoft.com/library/dn376476%28v=wps.620%29.aspx) cmdlet in Windows PowerShell to uninstall the app that is listed in the log.
  
5. The sysprep process reboots the PC and starts at the first-run experience screen. Complete the set up, and then sign in to the computer using an account that has local administrator privileges.

6. Right-click Start, go to **Control Panel** (view by large or small icons) > **System** > **Advanced system settings**, and click **Settings** in the **User Profiles** section.
    
7. In **User Profiles**, click **Default Profile**, and then click **Copy To**.

   ![Example of UI](images/copy-to.png)

8. In **Copy To**, under **Permitted to use**, click **Change**.

   ![Example of UI](images/copy-to-change.png)
   
9. In **Select User or Group**, in the **Enter the object name to select** field, type `everyone`, click **Check Names**, and then click **OK**.

10. In **Copy To**, in the **Copy profile to** field, enter the path and folder name where you want to store the mandatory profile. The folder name must use the correct [extension](#extension) for the operating system version. For example, the folder name must end with “.v6” to identify it as a user profile folder for Windows 10, version 1607.

   - If the device is joined to the domain and you are signed in with an account that has permissions to write to a shared folder on the network, you can enter the shared folder path.
   - If the device is not joined to the domain, you can save the profile locally and then copy it to the shared folder location.  
   
   ![Example of UI](images/copy-to-path.png) 

9. Click **OK** to copy the default user profile.


**To make the user profile mandatory**

 
3. In File Explorer, open the folder where you stored the copy of the profile.

    >[!NOTE]
    >If the folder is not displayed, click **View** > **Options** > **Change folder and search options**. On the **View** tab, select **Show hidden files and folders**, clear **Hide protected operating system files**, click **Yes** to confirm that you want to show operating system files, and then click **OK** to save your changes.

1. Rename `Ntuser.dat` to `Ntuser.man`. 

## How to apply a mandatory user profile to users

In a domain, you modify properties for the user account to point to the mandatory profile in a shared folder residing on the server.

**To apply a mandatory user profile to users**

1. Open **Active Directory Users and Computers** (dsa.msc).

2. Navigate to the user account that you will assign the mandatory profile to.

3. Right-click the user name and open **Properties**.

4. On the **Profile** tab, in the **Profile path** field, enter the path to the shared folder without the extension. For example, if the folder name is \\\\*server*\profile.v6, you would enter \\\\*server*\profile.

5. Click **OK**.

It may take some time for this change to replicate to all domain controllers.



## Apply policies to improve sign-in time

When a user is configured with a mandatory profile, Windows 10 starts as though it was the first sign-in each time the user signs in. To improve sign-in performance for users with mandatory user profiles, apply the following Group Policy settings.

- Computer Configuration > Administrative Templates > System > Logon > **Show first sign-in animation** = Disabled
- Computer Configuration > Administrative Templates > Windows Components > Search > **Allow Cortana** = Disabled
- Computer Configuration > Administrative Templates > Windows Components > Cloud Content > **Turn off Microsoft consumer experience** = Enabled









## Related topics

- [Manage Windows 10 Start layout and taskbar options](windows-10-start-layout-options-and-policies.md)
- [Lock down Windows 10 to specific apps](lock-down-windows-10-to-specific-apps.md)
- [Windows Spotlight on the lock screen](windows-spotlight.md)
- [Configure devices without MDM](configure-devices-without-mdm.md)


